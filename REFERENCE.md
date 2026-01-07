# REFERENCE.md

## Notation

```pseudocode
// All "assert" statements are normative.
// Violations MUST produce an error (or equivalent hard failure).

type Int
type Float
type Bool
type String
type Error
````

---

## Constants

```pseudocode
const CLUSTER_DEFAULT_SYSTEMS = 100
const STAR_ORBITS_PER_STAR    = 10   // orbit indices 0..9
const SYSTEM_ID_WIDTH         = 2    // "xx-yy-zz"
```

---

## Core Types

```pseudocode
type Vec3i { x:Int, y:Int, z:Int }    // integer coordinates
type Vec3f { x:Float, y:Float, z:Float }

type Preset Int
const PresetCoreForward = 0
const PresetBalanced    = 1
const PresetFlatter     = 2

type OrbitSlot { starSeq:Int, orbitIndex:Int } // orbitIndex 0..9, starSeq 1..starCount
```

---

## PRNG Requirements

```pseudocode
// The generator MUST accept an external PRNG.
// Any entity requiring its own PRNG MUST be seeded deterministically from the parent PRNG
// in creation order.

type RNG // implementation-defined; must provide NextInt/NextFloat and Shuffle primitives

function DeriveSeed(parent:RNG) -> UInt64:
    // Draw exactly one 64-bit value from parent
    return parent.NextU64()

function NewRNGFromSeed(seed:UInt64) -> RNG:
    // implementation-defined; must be deterministic
    return RNG(seed)
```

---

## Dice

```pseudocode
function Roll1d4(rng:RNG) -> Int:
    // returns 1..4 inclusive
    return 1 + (rng.NextInt() mod 4)

function Roll3d4Minus2(rng:RNG) -> Int:
    // returns 1..10 inclusive
    return (Roll1d4(rng) + Roll1d4(rng) + Roll1d4(rng)) - 2
```

---

## Coordinate Formatting and Parsing

```pseudocode
function Format2(n:Int) -> String:
    assert 0 <= n <= 99
    // returns exactly two digits, zero-padded
    return zeroPadDecimal(n, 2)

function SystemIDFromXYZ(x:Int, y:Int, z:Int) -> String:
    assert 1 <= x <= 31
    assert 1 <= y <= 31
    assert 1 <= z <= 31
    return Format2(x) + "-" + Format2(y) + "-" + Format2(z)

function ParseSystemID(systemID:String) -> Vec3i:
    // expects "xx-yy-zz" where xx/yy/zz are two digits each
    parts = split(systemID, "-")
    assert length(parts) == 3
    x = parseInt(parts[0]); y = parseInt(parts[1]); z = parseInt(parts[2])
    assert 1 <= x <= 31
    assert 1 <= y <= 31
    assert 1 <= z <= 31
    return Vec3i{x,y,z}
```

---

## Star, Jump Point, and Orbit Identifiers

```pseudocode
function JumpPointID(systemID:String) -> String:
    // "{systemId}/0"
    _ = ParseSystemID(systemID) // validates
    return systemID + "/0"

function StarID(systemID:String, starSeq:Int) -> String:
    _ = ParseSystemID(systemID)
    assert starSeq >= 1
    return systemID + "/" + toDecimal(starSeq)

function OrbitID(systemID:String, starSeq:Int, orbitLetter:Char) -> String:
    _ = ParseSystemID(systemID)
    assert starSeq >= 1
    assert orbitLetter in ['a'..'j'] or orbitLetter in ['A'..'J']
    return systemID + "/" + toDecimal(starSeq) + string(orbitLetter)
```

---

## Orbit Index ↔ Letter Mapping (Canonical)

```pseudocode
function OrbitIndexToLetter(orbitIndex:Int, occupied:Bool) -> Char:
    assert 0 <= orbitIndex <= 9
    base = char('a' + orbitIndex)
    if occupied:
        return uppercase(base)
    return base

function OrbitLetterToIndex(orbitLetter:Char) -> Int:
    letter = lowercase(orbitLetter)
    assert 'a' <= letter <= 'j'
    return ord(letter) - ord('a')

function IsOccupiedOrbit(orbitLetter:Char) -> Bool:
    // occupied iff uppercase A..J
    return orbitLetter is uppercase
```

---

## Star Count by System Generation Index

```pseudocode
function StarCountForSystemIndex(i:Int) -> Int:
    assert i >= 0

    if i == 0:
        return 4
    if 1 <= i and i <= 8:
        return 3  // 8 systems
    if 9 <= i and i <= 24:
        return 2  // 16 systems
    return 1      // remaining systems
```

---

## Plummer Sampling (Truncated) + Separation

```pseudocode
function Radius(v:Vec3f) -> Float:
    return sqrt(v.x*v.x + v.y*v.y + v.z*v.z)

function Distance(a:Vec3f, b:Vec3f) -> Float:
    dx=a.x-b.x; dy=a.y-b.y; dz=a.z-b.z
    return sqrt(dx*dx + dy*dy + dz*dz)

// Implementation-defined; must sample from a Plummer-like distribution depending on preset.
// NOTE: This is intentionally abstract. The accept/reject + determinism rules are normative.
function SamplePlummer(rng:RNG, preset:Preset) -> Vec3f:
    // returns a 3D point in continuous space
    return implementationDefinedPlummerSample(rng, preset)

function QuantizeToGrid(p:Vec3f) -> Vec3i:
    // Canonical quantization rule:
    // - Round to nearest integer
    // - Ties round away from zero (or consistently defined by language)
    return Vec3i{
        x = round(p.x),
        y = round(p.y),
        z = round(p.z),
    }

function InBoundsGrid(c:Vec3i) -> Bool:
    return (1 <= c.x <= 31) and (1 <= c.y <= 31) and (1 <= c.z <= 31)

function AcceptCandidate(candidate:Vec3f, accepted:List<Vec3f>, maxRadius:Float, minSep:Float) -> Bool:
    if Radius(candidate) > maxRadius:
        return false
    for each a in accepted:
        if Distance(candidate, a) < minSep:
            return false
    return true

function SampleSystemLocation(
    rng:RNG,
    preset:Preset,
    maxRadius:Float,
    minSep:Float,
    accepted:List<Vec3f>,
    maxAttempts:Int
) -> Vec3i or Error:
    attempts = 0
    while attempts < maxAttempts:
        attempts += 1
        p = SamplePlummer(rng, preset)
        if not AcceptCandidate(p, accepted, maxRadius, minSep):
            continue
        c = QuantizeToGrid(p)
        if not InBoundsGrid(c):
            continue
        // IMPORTANT: record accepted position in continuous space for separation testing
        append(accepted, p)
        return c
    return Error("unable to place system within constraints")
```

---

## Planet Distribution Across Stars and Orbits (Shuffled Slots)

```pseudocode
function BuildCandidateOrbits(starCount:Int) -> List<OrbitSlot>:
    assert starCount >= 1
    slots = emptyList<OrbitSlot>()
    for starSeq from 1 to starCount:
        for orbitIndex from 0 to 9:
            append(slots, OrbitSlot{starSeq, orbitIndex})
    return slots

function ShuffleInPlace(rng:RNG, slots:List<OrbitSlot>):
    // MUST be deterministic given rng state and slot order
    rng.Shuffle(slots)

function PlanetCountForSystem(rng:RNG, starCount:Int) -> Int:
    assert starCount >= 1
    perStar = Roll3d4Minus2(rng)   // 1..10
    return starCount * perStar     // starCount..(starCount*10)

function AssignPlanetsToOrbits(rng:RNG, starCount:Int) -> List<OrbitSlot>:
    // Returns occupied orbits as a list of OrbitSlot; order is placement order (after shuffle).
    planetCount = PlanetCountForSystem(rng, starCount)
    slots = BuildCandidateOrbits(starCount)
    ShuffleInPlace(rng, slots)

    assert planetCount <= length(slots) // always true given formula, but still normative

    occupied = emptyList<OrbitSlot>()
    for i from 0 to planetCount-1:
        append(occupied, slots[i])
    return occupied
```

---

## Cluster Generation (Systems → Stars → Orbits → Planets)

```pseudocode
type GeneratedSystem {
    index:Int
    coord:Vec3i
    systemID:String
    starCount:Int
    systemRNG:RNG
    // occupiedOrbits is a list of slots that become planets
    occupiedOrbits:List<OrbitSlot>
}

function GenerateClusterPlummerInt(
    rng:RNG,
    numberOfSystemsToGenerate:Int,
    maxRadius:Float,
    minSep:Float,
    preset:Preset
) -> List<GeneratedSystem> or Error:

    assert rng != nil
    assert numberOfSystemsToGenerate > 0
    assert maxRadius > 0
    assert minSep >= 0

    systems = emptyList<GeneratedSystem>()
    acceptedContinuous = emptyList<Vec3f>()
    maxAttemptsPerSystem = 1_000_000

    // 1) systems
    for i from 0 to numberOfSystemsToGenerate-1:
        // 1a) per-system RNG (derived deterministically)
        sysSeed = DeriveSeed(rng)
        sysRNG  = NewRNGFromSeed(sysSeed)

        // 1b) location (uses sysRNG)
        coordOrErr = SampleSystemLocation(sysRNG, preset, maxRadius, minSep, acceptedContinuous, maxAttemptsPerSystem)
        if coordOrErr is Error:
            return coordOrErr
        coord = coordOrErr

        // 1c) identifiers
        systemID = SystemIDFromXYZ(coord.x, coord.y, coord.z)

        // 2) stars (count by generation index)
        starCount = StarCountForSystemIndex(i)

        // 3) orbits (implicit: 10 per star) + 4) planets (occupy subset)
        // Use sysRNG for planet counts + shuffling to keep all system-local randomness isolated.
        occupiedSlots = AssignPlanetsToOrbits(sysRNG, starCount)

        append(systems, GeneratedSystem{
            index=i,
            coord=coord,
            systemID=systemID,
            starCount=starCount,
            systemRNG=sysRNG,
            occupiedOrbits=occupiedSlots,
        })

    return systems
```

---

## Derived Entity IDs for Occupied Orbits (Planets)

```pseudocode
function PlanetID(systemID:String, slot:OrbitSlot) -> String:
    assert 1 <= slot.starSeq
    assert 0 <= slot.orbitIndex <= 9
    letter = OrbitIndexToLetter(slot.orbitIndex, occupied=true) // uppercase
    return OrbitID(systemID, slot.starSeq, letter)
```

---
