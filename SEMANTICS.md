# Semantics for Coding Agent

## PRNG & Deterministic Generation (Candidate)

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

8. All PRNGs are instances of `*prng.Rand` from `github.com/mdhender/prng` (version **MUST** be >= v1.1.0) and **MUST** provide the same method behavior as Go’s `math/rand/v2` `*rand.Rand` for all methods used by this specification.

9. All PRNG instances used by Seiicho generators **MUST** be backed by a PCG source created with `math/rand/v2`:
>   src := rand.NewPCG(seed, seq)

`src` **MUST** be a `*rand.PCG` instance (not a `*rand.Rand` and not any other engine).

The canonical way to construct a Seiicho PRNG is:
>   rng := prng.New(src)

10. Child PRNGs **MUST** be created **ONLY** via:
>   child_rng := rng.Child()

rng.Child() is normative and **MUST** be equivalent to `derive_child_rng(parent_rng)` as defined in `REFERENCE.md`.

11. PRNGs **MUST NOT** be re-seeded after creation.

---

## Location-Based Identifiers (Candidate)

### Overview

All **systems, stars, orbits, planets, and orbital rings** are identified **solely by location-based identifiers**.
Identifiers are **canonical, deterministic, and stable** once assigned.

No identifier may depend on discovery order, sorting, or runtime state beyond the rules defined here.

---

### Coordinate Space

1. The cluster coordinate space is a three-dimensional integer grid.

2. Coordinates are represented as integer triples `(x, y, z)` where:

    * `x`, `y`, and `z` each range from **1 to 31**, inclusive.

3. The center of the cluster is fixed at:

   ```
   (16, 16, 16)
   ```

---

### System Identifiers

4. Every cluster contains **exactly 100 systems**.

5. Each system is uniquely identified by its coordinates, formatted as:

   ```
   "xx-yy-zz"
   ```

   where:

    * `xx`, `yy`, `zz` are zero-padded, two-digit decimal representations of `x`, `y`, and `z`.

6. Example:

    * Coordinates `(14, 8, 29)` → system identifier:

      ```
      "14-08-29"
      ```

---

### System Jump Point

7. Every system has exactly one **entry/exit point**, called the **jump point**.

8. The jump point identifier is:

   ```
   "{systemId}/0"
   ```

9. Example:

    * Jump point for system `"14-08-29"`:

      ```
      "14-08-29/0"
      ```

---

### Star Identifiers

10. Each system contains **between 1 and 4 stars**, inclusive.

11. Stars within a system are assigned a **1-based sequence index** in **generation order**.

12. A star identifier is formatted as:

```
"{systemId}/{n}"
```

where `n ∈ {1, 2, 3, 4}`.

13. Examples:

* First star in system `"28-02-18"`:

  ```
  "28-02-18/1"
  ```
* Second star:

  ```
  "28-02-18/2"
  ```

---

### Orbit Identifiers

14. Every star contains **exactly 10 orbits**.

15. Orbits are identified by a **lowercase letter** from `a` through `j`, inclusive.

16. Orbit letters correspond to **orbit index order**:

```
a → 1st orbit
b → 2nd orbit
…
j → 10th orbit
```

17. An orbit identifier is formatted as:

```
"{starId}{orbitLetter}"
```

18. Examples:

* First orbit of star `"28-02-18/1"`:

  ```
  "28-02-18/1a"
  ```
* Ninth orbit:

  ```
  "28-02-18/1i"
  ```

---

### Planet Identifiers

19. An orbit **may contain at most one planet**.

20. Planets are identified by an **uppercase letter** from `A` through `J`, inclusive.

21. Planet letters correspond positionally to orbit letters:

```
a ↔ A
b ↔ B
…
j ↔ J
```

22. A planet identifier is formatted as:

```
"{starId}{planetLetter}"
```

23. Examples:

* Planet in the first orbit of `"28-02-18/1"`:

  ```
  "28-02-18/1A"
  ```
* Planet in the ninth orbit:

  ```
  "28-02-18/1I"
  ```

---

### Orbital Rings

24. Orbital rings represent relative distance from a planet’s surface.

25. Ring indices range from **0 to 100**, inclusive:

* `0` represents the planet surface.

26. An orbital ring identifier is formed by appending:

```
"+{ring}"
```

to a planet identifier.

27. Examples:

* Surface of planet `"28-02-18/1A"`:

  ```
  "28-02-18/1A+0"
  ```
* Ring 42:

  ```
  "28-02-18/1A+42"
  ```

---

### Identifier Constraints

28. Identifier components are **case-sensitive**:

* orbit letters are lowercase
* planet letters are uppercase

29. Identifiers are **purely structural**:

* they encode location only,
* they do not imply physical presence, habitability, or usability.

30. Any identifier not conforming to the formats defined in this section is **invalid**.

---

## Cluster Generation (Candidate)

### Overview

Cluster generation proceeds in a fixed hierarchical order:

```
systems → stars → orbits → planets
```

All generation steps are **deterministic** and **PRNG-driven**, subject to the rules defined below.

---

## System Generation (Candidate)

### Generation Order

1. Generate `N` systems, where `N = numberOfSystemsToGenerate` (default `N = 100`).
2. Systems are generated in **generation index order**:

   ```
   i = 0 .. N−1
   ```

---

### Star Count per System

3. The number of stars in a system is determined solely by its generation index `i`:

| Generation Index `i` | Star Count |
| -------------------- | ---------- |
| `i == 0`             | 4          |
| `1 ≤ i ≤ 8`          | 3          |
| `9 ≤ i ≤ 24`         | 2          |
| `i ≥ 25`             | 1          |

This mapping is **fixed and invariant**.

---

### System Coordinate Assignment

4. Each system is assigned a unique 3D location by sampling a **truncated Plummer sphere**.

5. Candidate locations are sampled repeatedly until accepted.

6. A candidate location is **rejected** if any of the following hold:

    * `radius(candidate) > maxRadius`
    * distance from candidate to any previously accepted system location `< minSep`

7. Once accepted, the location is **quantized to integer coordinates** `(x, y, z)`.

8. The coordinate quantization rule **MUST be deterministic** and is defined elsewhere.

---

### System Generation Signature

```pseudocode
// Preset controls cluster concentration.
type Preset int

const (
    PresetCoreForward Preset = iota  // denser center, sparser outskirts
    PresetBalanced                   // default
    PresetFlatter                    // more even distribution
)

function GenerateClusterPlummerInt(
    rng,
    numberOfSystemsToGenerate,
    maxRadius,
    minSep,
    preset
) -> (systems[], error)
```

---

## Planet Distribution Across Stars and Orbits

### Overview

Planets are generated **at the system level** and then assigned to specific stars and orbits in a deterministic manner.

Only **occupied orbits** are instantiated as planet entities.
Empty orbits are implicit and not stored.

---

### Definitions

1. Each star has **exactly 10 orbits**, indexed `1..10`.
2. Orbit indices map canonically to letters:

```
1 → a / A
2 → b / B
…
10 → j / J
```

Lowercase letters denote **empty orbits**.
Uppercase letters denote **occupied orbits (planets)**.

---

### Planet Count Determination

3. Let `starCount` be the number of stars in the system.

4. The total number of planets for the system is determined as follows:

```pseudocode
planetCount := 0
for s = 1 .. starCount:
    planetCount += (3d4 - 2)
```

5. Per-star planet contribution range:

    * minimum: `1`
    * maximum: `10`

6. Total planet count range per system:

   ```
   starCount .. (starCount × 10)
   ```

---

### Candidate Orbit Construction

7. Construct a list of candidate orbits:

```pseudocode
candidateOrbits := []
for starSeq = 1 .. starCount:
    for orbitIndex = 1 .. 10:
        candidateOrbits.append((starSeq, orbitIndex))
```

Each candidate orbit is uniquely identified by `(starSeq, orbitIndex)`.

---

### Orbit Shuffling

8. Shuffle `candidateOrbits` using the **system’s PRNG**.

9. The shuffle **MUST be deterministic** given the same PRNG state.

---

### Planet Assignment

10. Assign planets to the **first `planetCount` entries** in the shuffled candidate-orbit list.

11. Each assigned orbit is marked as **occupied**.

12. All remaining orbits remain empty.

---

### Star Coverage Correction

13. After assignment, verify that **each star has at least one occupied orbit**.

14. For any star with zero planets:

* Select **one orbit at random** using the system PRNG.
* Mark it as occupied.
* All other orbits for that star remain unchanged.

---

### Constraints

15. All stars **MUST** have at least one planet.

16. A star **MUST NOT** have more than 10 planets.

17. If `planetCount` exceeds the total number of available orbits:

* This is a **generation error**.
* Cluster generation **MUST fail**.

---

### Determinism Guarantees

18. Planet placement **MUST NOT** depend on:

* system coordinates,
* star properties,
* orbit geometry.

19. Planet placement **MUST depend only on**:

* input PRNG state,
* number of stars in the system,
* `planetCount`.

This guarantees replayable, identical planet layouts for identical inputs.

---

### Orbit Occupancy Test

```pseudocode
function IsOccupiedOrbit(orbitLetter):
    assert orbitLetter is a single ASCII letter
    return orbitLetter is uppercase
```

---

### Canonical Properties

20. Orbit index → letter mapping is fixed and invariant.

21. Orbit occupancy is encoded **solely by letter casing**.

22. Orbit index and letter mapping **MUST NOT** depend on:

* star type,
* system location,
* planet type,
* PRNG state.

---

### Invalid Inputs

23. Any orbit index outside `1..10` is invalid and **MUST cause an error**.

24. Any orbit letter outside `'a'..'j'` or `'A'..'J'` is invalid and **MUST cause an error**.

---
