# Semantics for Coding Agent

## PRNG & Deterministic Generation (LOCKED)

### PRNG Requirement

1. Every generator function **MUST** accept an explicit PRNG input named `rng`.
2. No generator **MAY** read randomness from any implicit or global source (including global RNGs, time, OS entropy, map iteration order, iteration over sets, unstable sort tie-breaking, or concurrency scheduling).

### Determinism

3. A generator **MUST** be deterministic: given identical:

    * input PRNG state,
    * generator parameters,
    * and identical call order,
      it **MUST** produce identical outputs.

4. All “first/next” rules refer to generation index order (creation order), not any ordering derived from map iteration, sets, unstable sort tie-breaking, concurrency scheduling, coordinates, distance, sorting, or IDs.

“Input PRNG state” means the full internal state of the PRNG at the moment the generator function is invoked (not merely its initial seed).

### Child PRNGs (Entity-Local RNGs)

5. If an entity requires its own PRNG (`entity.rng`), the entity PRNG **MUST** be derived deterministically from the parent PRNG `rng` **at the moment the entity is created**, and in **creation order**.

6. Child PRNG derivation **MUST** consume randomness from the parent PRNG. (This makes child seeding part of the parent’s deterministic stream and prevents “seed skipping” bugs.)

7. Child PRNG derivation is **normative** and is defined in `REFERENCE.md` as `derive_child_rng(parent_rng) -> child_rng`.

### Go PRNG Family

8. All PRNGs are from Go’s `math/rand/v2`.

9. To avoid ambiguity across possible `math/rand/v2` engines, all PRNG instances used by Seiicho generators **MUST** be created using the PCG constructor (NewPCG(seed, seq)), and child PRNGs MUST also be created using NewPCG.

10. PRNGs MUST NOT be re-seeded after creation.

---

## Cluster Generation (Draft)

Generation order is always: **systems → stars → orbits → planets**.

### Rules

1. Generate `N` systems (default `N=100`) in **generation index order** `i = 0..N-1`.
2. Star count by generation index:
    - `i == 0` → 4 stars
    - `1 <= i <= 8` → 3 stars (8 systems)
    - `9 <= i <= 24` → 2 stars (16 systems)
    - `25 <= i` → 1 star (remaining systems)
3. Assign each system a 3D location by sampling a **truncated Plummer sphere**:
    - Sample candidate points until accepted.
    - Reject a candidate if `radius(candidate) > maxRadius`.
    - Reject a candidate if it violates `minSep` from any accepted system location.
    - Quantize the accepted location to integer coordinates (define rounding rule elsewhere, or here).
4. Planets:
    - Number and orbit of planets per system in following section.

```pseudocode
// Preset controls how concentrated the cluster is.
type Preset int

const (
	PresetCoreForward Preset = iota // denser center, sparser outskirts
	PresetBalanced                  // good default
	PresetFlatter                   // more even exploration
)

func GenerateClusterPlummerInt(
	rng *rand.Rand, // from math/rand/v2
	numberOfSystemsToGenerate int,
	maxRadius float64,
	minSep float64,
	preset Preset,
) ([]Vec3i, error) {
	// ...
}
```

### Planet Distribution Across Stars and Orbits

Planets are generated at the **system level** and then assigned to specific stars and orbits in a deterministic manner.

#### Definitions

- Each **star** has exactly **10 orbits**, indexed `0..9`, corresponding to letters `a..j`.
- Only **non-empty orbits** (planets) are instantiated as entities.
- Empty orbits are implicit and not stored.

#### Generation Rules

1. Determine the total number of planets for the system:
    - Let `starCount` be the number of stars in the system.
    - For `i = 0..starCount-1`:
      - `planetCount += 3d4 - 2`.
    - Valid range per star is `1..10`; total planet count per system is `starCount..(starCount * 10)`.

2. Construct the list of candidate orbits:
    - For each star with sequence number `1..starCount`:
        - Add all 10 orbit slots (`a..j`, indices `0..9`) to a list of candidate orbits.
    - Each candidate orbit is uniquely identified by `(starSeq, orbitIndex)`.

3. Shuffle candidate orbits:
    - Shuffle the candidate-orbit list using the system’s PRNG.
    - The shuffle MUST be deterministic given the same PRNG state.

4. Assign planets:
    - Assign planets to the first `planetCount` entries in the shuffled candidate-orbit list.
    - Each selected orbit is marked as occupied.
    - All remaining orbits remain empty.

5. Completion:
    - Stop once `planetCount` planets have been placed.

#### Orbit Identification

- Orbit letters:
    - Lowercase (`a..j`) indicate empty orbits (implicit, not stored).
    - Uppercase (`A..J`) indicate occupied orbits (planets).
- A planet’s identifier is constructed as: `{systemId}/{starSeq}{orbitLetter}`

Example:
- `28-02-18/1C` → third orbit of star 1 in system 28-02-18.

#### Constraints

- A star cannot have more than 10 planets.
- If planet distribution would exceed available orbits across all stars:
- This is a generation error and MUST cause cluster generation to fail.

#### Determinism

- Planet assignment order MUST NOT depend on:
  - system coordinates,
  - orbit randomization,
  - star properties.
- Only the following affect planet placement:
  - input PRNG state,
  - number of stars in the system,
  - `planetCount`.

This guarantees identical planet layouts for identical inputs and replayable turn resolution.

### Orbit Index ↔ Letter Mapping (Canonical)

This section defines the authoritative mapping between numeric orbit indices and orbit letters.
All implementations MUST use this mapping.

#### Definitions

- Orbit indices are zero-based integers in the range `0..9`.
- Orbit letters are single ASCII characters in the range:
  - `'a'..'j'` for empty orbits
  - `'A'..'J'` for occupied orbits

#### Index → Letter

```pseudocode
function OrbitIndexToLetter(orbitIndex, occupied):
    assert 0 <= orbitIndex <= 9

    baseLetter = char('a' + orbitIndex)

    if occupied == true:
        return uppercase(baseLetter)

    return baseLetter
````

#### Letter → Index

```pseudocode
function OrbitLetterToIndex(orbitLetter):
    assert orbitLetter is a single ASCII letter

    letter = lowercase(orbitLetter)

    assert 'a' <= letter <= 'j'

    return ord(letter) - ord('a')
```

#### Occupancy Test

```pseudocode
function IsOccupiedOrbit(orbitLetter):
    assert orbitLetter is a single ASCII letter
    return orbitLetter is uppercase
```

#### Canonical Properties

* Orbit index `0` always maps to letter `'a'` (empty) or `'A'` (occupied).
* Orbit index `9` always maps to letter `'j'` (empty) or `'J'` (occupied).
* Orbit occupancy is encoded **solely** by letter casing.
* Orbit index and letter mapping MUST NOT depend on:

    * star type
    * system location
    * planet type
    * PRNG state

#### Invalid Inputs

* Any orbit index outside `0..9` is invalid and MUST cause an error.
* Any orbit letter outside `'a'..'j'` or `'A'..'J'` is invalid and MUST cause an error.

