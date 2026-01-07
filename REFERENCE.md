# REFERENCE.md

## Canonical PRNG Derivation (Executable-Style Pseudocode) (LOCKED)

```text
# derive_child_rng produces a new PRNG for an entity.
# It is the ONLY allowed method to create entity-local PRNGs.
# parent_rng MUST provide Uint64() -> uint64

function derive_child_rng(parent_rng):
    # This function MUST consume exactly two Uint64 values from parent_rng, in this order.
    seed := parent_rng.Uint64()
    seq  := parent_rng.Uint64()

    # seq MUST be odd for PCG stream selection.
    # Force odd deterministically.
    seq := seq BITOR 1

    return NewPCG(seed, seq)
```

```text
# Example: entity creation
function create_entity(parent_rng, ...):
    entity_rng := derive_child_rng(parent_rng)
    entity := new Entity()
    entity.rng = entity_rng
    ...
    return entity
```

## Location-Based Identifier Construction & Parsing (Candidate)

---

### Constants

```text
CONST MIN_COORD = 1
CONST MAX_COORD = 31

CONST SYSTEM_COUNT = 100

CONST STAR_MIN = 1
CONST STAR_MAX = 4

CONST ORBIT_COUNT = 10
CONST ORBIT_LETTERS = ["a","b","c","d","e","f","g","h","i","j"]
CONST PLANET_LETTERS = ["A","B","C","D","E","F","G","H","I","J"]

CONST RING_MIN = 0
CONST RING_MAX = 100
```

---

### Coordinate Validation

```text
function validate_coordinates(x, y, z):
    assert MIN_COORD <= x <= MAX_COORD
    assert MIN_COORD <= y <= MAX_COORD
    assert MIN_COORD <= z <= MAX_COORD
```

---

### System Identifier

```text
function format_system_id(x, y, z):
    validate_coordinates(x, y, z)

    return zero_pad(x, 2) + "-" +
           zero_pad(y, 2) + "-" +
           zero_pad(z, 2)
```

```text
function parse_system_id(system_id):
    parts := split(system_id, "-")
    assert length(parts) == 3

    x := parse_int(parts[0])
    y := parse_int(parts[1])
    z := parse_int(parts[2])

    validate_coordinates(x, y, z)

    return (x, y, z)
```

---

### Jump Point Identifier

```text
function format_jump_point_id(system_id):
    return system_id + "/0"
```

```text
function is_jump_point_id(id):
    return ends_with(id, "/0")
```

---

### Star Identifier

```text
function format_star_id(system_id, star_index):
    assert STAR_MIN <= star_index <= STAR_MAX

    return system_id + "/" + to_string(star_index)
```

```text
function parse_star_id(star_id):
    parts := split(star_id, "/")
    assert length(parts) == 2

    system_id := parts[0]
    star_index := parse_int(parts[1])

    assert STAR_MIN <= star_index <= STAR_MAX

    return (system_id, star_index)
```

---

### Orbit Identifier

```text
function format_orbit_id(star_id, orbit_index):
    assert 1 <= orbit_index <= ORBIT_COUNT

    letter := ORBIT_LETTERS[orbit_index - 1]

    return star_id + letter
```

```text
function parse_orbit_id(orbit_id):
    star_id := orbit_id[0 : length(orbit_id) - 1]
    letter  := orbit_id[length(orbit_id) - 1]

    index := index_of(ORBIT_LETTERS, letter)
    assert index != -1

    orbit_index := index + 1

    return (star_id, orbit_index)
```

---

### Planet Identifier

```text
function format_planet_id(star_id, orbit_index):
    assert 1 <= orbit_index <= ORBIT_COUNT

    letter := PLANET_LETTERS[orbit_index - 1]

    return star_id + letter
```

```text
function parse_planet_id(planet_id):
    star_id := planet_id[0 : length(planet_id) - 1]
    letter  := planet_id[length(planet_id) - 1]

    index := index_of(PLANET_LETTERS, letter)
    assert index != -1

    orbit_index := index + 1

    return (star_id, orbit_index)
```

---

### Orbital Ring Identifier

```text
function format_ring_id(planet_id, ring_index):
    assert RING_MIN <= ring_index <= RING_MAX

    return planet_id + "+" + to_string(ring_index)
```

```text
function parse_ring_id(ring_id):
    parts := split(ring_id, "+")
    assert length(parts) == 2

    planet_id := parts[0]
    ring_index := parse_int(parts[1])

    assert RING_MIN <= ring_index <= RING_MAX

    return (planet_id, ring_index)
```

---

### Identifier Classification

```text
function classify_identifier(id):
    if contains(id, "+"):
        return "ring"
    if ends_with(id, "/0"):
        return "jump_point"
    if contains(id, "/") and ends_with_digit(id):
        return "star"
    if contains(id, "/") and ends_with_uppercase_letter(id):
        return "planet"
    if contains(id, "/") and ends_with_lowercase_letter(id):
        return "orbit"
    if contains(id, "-"):
        return "system"

    error "invalid identifier"
```

---

### Invariants

```text
# All identifier construction MUST use these functions.
# Parsing functions MUST reject malformed identifiers.
# Identifier case is significant and MUST be preserved.
# Identifiers encode location only and imply no existence.
```

---

```text
## Cluster Generation (Executable-Style Pseudocode) (Draft)

# NOTE: All generation errors are hard failures.
# Implementations MUST panic() on any invalid input or unsatisfiable constraints.

# Generation order is always: systems → stars → orbits → planets.


############################
# Types
############################

type Vec3i:
    x int
    y int
    z int

type System:
    index       int         # generation index i
    coord       Vec3i       # integer coordinate (x,y,z)
    starCount   int         # 1..4
    # occupied[starSeq][orbitIndex] => bool
    # starSeq is 1-based; orbitIndex is 1..10
    occupied    bool[1..4][1..10]


type Preset int
const (
    PresetCoreForward = 0
    PresetBalanced    = 1
    PresetFlatter     = 2
)


############################
# Constants (from SEMANTICS / REFERENCE)
############################

CONST CENTER = (16,16,16)

CONST MIN_COORD = 1
CONST MAX_COORD = 31

CONST STAR_MIN = 1
CONST STAR_MAX = 4

CONST ORBIT_COUNT = 10
CONST ORBIT_LETTERS  = ["a","b","c","d","e","f","g","h","i","j"]
CONST PLANET_LETTERS = ["A","B","C","D","E","F","G","H","I","J"]

# Child PRNG derivation is normative (REFERENCE v0.1)
# derive_child_rng(parent_rng) -> child_rng


############################
# Helpers: math
############################

function sqr(f): return f*f

function radius3(x, y, z):
    return sqrt(sqr(x) + sqr(y) + sqr(z))

function dist3i(a Vec3i, b Vec3i):
    dx := float64(a.x - b.x)
    dy := float64(a.y - b.y)
    dz := float64(a.z - b.z)
    return sqrt(dx*dx + dy*dy + dz*dz)


############################
# Helpers: validation
############################

function assert_coord_in_bounds(v Vec3i):
    assert MIN_COORD <= v.x <= MAX_COORD
    assert MIN_COORD <= v.y <= MAX_COORD
    assert MIN_COORD <= v.z <= MAX_COORD


############################
# Helpers: float → int quantization
############################

# Requirement: Use int(math.Trunc(f)) when converting floats to int.
# This truncates towards the origin (zero).

function trunc_to_int(f):
    return int(math.Trunc(f))

# Convert a float position (relative to origin) to integer grid coords centered at (16,16,16).
# Floats MUST round towards the origin before adding the center offset.
# Example:
#   xf = -3.8  → trunc_to_int(xf) = -3
#   xi = 16 + (-3) = 13
function quantize_centered(xf, yf, zf):
    xi := CENTER.x + trunc_to_int(xf)
    yi := CENTER.y + trunc_to_int(yf)
    zi := CENTER.z + trunc_to_int(zf)

    v := Vec3i{xi, yi, zi}
    assert_coord_in_bounds(v)
    return v


############################
# System star-count mapping (fixed)
############################

function star_count_for_system_index(i, N):
    assert 0 <= i < N

    if i == 0:
        return 4
    if 1 <= i and i <= 8:
        return 3
    if 9 <= i and i <= 24:
        return 2
    return 1


############################
# Truncated Plummer sphere sampling
############################

# Returns a float point (xf,yf,zf) relative to the cluster center origin (0,0,0).
# Candidate is later quantized and bounds-checked.
#
# Preset affects Plummer scale parameter 'a' deterministically.
# This definition is normative for sampling candidates.
#
# Sampling method:
# - Generate Plummer radius r with scale a:
#     r = a / sqrt(u^(-2/3) - 1), where u ~ Uniform(0,1)
# - Generate isotropic direction via cos(theta) in [-1,1], phi in [0, 2π)
# - Convert to Cartesian
#
# Truncation rule:
# - Reject candidates with radius(r) > maxRadius

function plummer_scale_a(maxRadius, preset):
    assert maxRadius > 0

    # Preset controls concentration:
    # - smaller 'a' => denser center
    # - larger 'a'  => flatter distribution within truncation radius
    if preset == PresetCoreForward:
        return maxRadius / 4.0
    if preset == PresetBalanced:
        return maxRadius / 2.0
    if preset == PresetFlatter:
        return maxRadius * 1.0

    panic("invalid preset")

function sample_truncated_plummer_candidate(rng, maxRadius, preset):
    a := plummer_scale_a(maxRadius, preset)

    loop forever:
        # u must be in (0,1) to avoid division by zero; clamp away from endpoints.
        u := rng.Float64()
        if u <= 0.0: continue
        if u >= 1.0: continue

        # r = a / sqrt(u^(-2/3) - 1)
        t := pow(u, -2.0/3.0) - 1.0
        if t <= 0.0: continue
        r := a / sqrt(t)

        if r > maxRadius:
            continue

        # isotropic direction
        cosTheta := (2.0 * rng.Float64()) - 1.0   # [-1,1]
        sinTheta := sqrt(max(0.0, 1.0 - cosTheta*cosTheta))
        phi := 2.0 * PI * rng.Float64()

        xf := r * sinTheta * cos(phi)
        yf := r * sinTheta * sin(phi)
        zf := r * cosTheta

        return (xf, yf, zf)


############################
# Generate cluster (systems only)
############################

# Go note (normative intent):
# - For deterministic shuffling, a *rand.Rand must be created using the PCG source:
#     r := rand.New(rng)
#     r.Shuffle(...)
# In this reference pseudocode we express this behavior via the system-local PRNG and Shuffle.

function GenerateClusterPlummerInt(
    rng,                       # *rand.Rand from math/rand/v2, backed by PCG
    numberOfSystemsToGenerate,  # N
    maxRadius,                 # float64
    minSep,                    # float64 (in integer-grid units after quantization)
    preset                     # Preset
) -> systems[]:

    # Hard-fail validation
    if rng == nil: panic("rng is nil")
    if numberOfSystemsToGenerate <= 0: panic("invalid N")
    if maxRadius <= 0: panic("invalid maxRadius")
    if minSep < 0: panic("invalid minSep")

    N := numberOfSystemsToGenerate

    systems := []
    usedCoords := map<Vec3i,bool>{}

    for i = 0 .. N-1:
        sys_rng := derive_child_rng(rng)

        starCount := star_count_for_system_index(i, N)

        # Sample until we find a valid, unique coordinate satisfying minSep.
        loop forever:
            (xf, yf, zf) := sample_truncated_plummer_candidate(sys_rng, maxRadius, preset)

            coord := quantize_centered(xf, yf, zf)   # may panic if out of bounds

            # Unique coordinate constraint (implicit from "identified by location")
            if usedCoords[coord] == true:
                continue

            # minSep constraint is checked against already-accepted integer coordinates.
            ok := true
            for each prior in systems:
                if dist3i(coord, prior.coord) < minSep:
                    ok = false
                    break
            if ok == false:
                continue

            # accept
            usedCoords[coord] = true

            sys := System{
                index: i,
                coord: coord,
                starCount: starCount,
                occupied: all_false
            }

            systems.append(sys)
            break

    # After systems exist, generate planets deterministically per system (below).
    for each sys in systems:
        generate_planets_for_system(sys, derive_child_rng(rng))

    return systems


############################
# Dice
############################

# 3d4 - 2
function roll_3d4_minus_2(rng):
    # d4 returns {1,2,3,4}
    a := 1 + rng.IntN(4)
    b := 1 + rng.IntN(4)
    c := 1 + rng.IntN(4)
    return (a + b + c) - 2   # range 1..10


############################
# Planet distribution per system
############################

# Planets are generated at the system level, then assigned to stars/orbits.
# Only occupied orbits become planets; empty orbits are implicit.

function generate_planets_for_system(sys System, sys_rng):
    if sys_rng == nil: panic("sys_rng is nil")
    if sys.starCount < STAR_MIN or sys.starCount > STAR_MAX: panic("invalid starCount")

    starCount := sys.starCount

    # 1) total planet count
    planetCount := 0
    for s = 1 .. starCount:
        planetCount += roll_3d4_minus_2(sys_rng)

    # Safety bound: cannot exceed available orbits
    if planetCount > (starCount * ORBIT_COUNT):
        panic("planetCount exceeds available orbits")

    # 2) candidate orbit list
    candidate := []  # list of (starSeq, orbitIndex)
    for starSeq = 1 .. starCount:
        for orbitIndex = 1 .. ORBIT_COUNT:
            candidate.append((starSeq, orbitIndex))

    # 3) shuffle deterministically using sys_rng
    # Go mapping: sys_rng.Shuffle(len(candidate), swap)
    shuffle_in_place(candidate, sys_rng)

    # 4) occupy first planetCount
    for k = 0 .. planetCount-1:
        (starSeq, orbitIndex) := candidate[k]
        sys.occupied[starSeq][orbitIndex] = true

    # 5) coverage correction: ensure at least 1 planet per star
    for starSeq = 1 .. starCount:
        hasPlanet := false
        for orbitIndex = 1 .. ORBIT_COUNT:
            if sys.occupied[starSeq][orbitIndex] == true:
                hasPlanet = true
                break

        if hasPlanet == false:
            # select one orbit uniformly at random from 1..10
            orbitIndex := 1 + sys_rng.IntN(ORBIT_COUNT)
            sys.occupied[starSeq][orbitIndex] = true


############################
# Deterministic shuffle (Fisher–Yates)
############################

# Equivalent to Go's rand.Shuffle using sys_rng.
function shuffle_in_place(list, sys_rng):
    n := length(list)
    for i = n-1 down to 1:
        j := sys_rng.IntN(i+1)
        swap(list[i], list[j])


############################
# Canonical orbit letter mapping and occupancy casing
############################

function orbit_letter_for_index(orbitIndex):
    assert 1 <= orbitIndex <= ORBIT_COUNT
    return ORBIT_LETTERS[orbitIndex - 1]

function planet_letter_for_index(orbitIndex):
    assert 1 <= orbitIndex <= ORBIT_COUNT
    return PLANET_LETTERS[orbitIndex - 1]

function orbit_letter_with_occupancy(orbitIndex, occupied):
    if occupied:
        return planet_letter_for_index(orbitIndex)  # uppercase
    return orbit_letter_for_index(orbitIndex)       # lowercase

function IsOccupiedOrbit(orbitLetter):
    assert orbitLetter is a single ASCII letter
    return orbitLetter is uppercase


############################
# Hard-fail invalid inputs (orbit indices / letters)
############################

function validate_orbit_index(orbitIndex):
    if orbitIndex < 1 or orbitIndex > ORBIT_COUNT:
        panic("invalid orbit index")

function validate_orbit_letter(letter):
    if index_of(ORBIT_LETTERS, letter) != -1:
        return
    if index_of(PLANET_LETTERS, letter) != -1:
        return
    panic("invalid orbit letter")
```



## Notation (Obsolete?)

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
