# Semantics for Coding Agent

## PRNG & Deterministic Generation (Locked)

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

### PRNG ownership pipeline (Normative)
```pseudocode
function GenerateGame(sys_rng):
    points := GenerateClusterChebyshevShuffleDraw(sys_rng)
    systems := []

    for i in 0..99:
        sys_rng_i := sys_rng.Child()
        system := GenerateSystem(sys_rng_i, points[i], i)
        systems.append(system)

    return systems

function GenerateSystem(sys_rng_i, point, i):
    starCount := StarCountFromIndex(i)
    stars := []

    for starSeq in 1..starCount:
        star_rng := sys_rng_i.Child()
        star := GenerateStar(star_rng, starSeq)
        stars.append(star)

    return System{point, stars}

function GenerateStar(star_rng, starSeq):
    planetCount := star_rng.IntN(4) + star_rng.IntN(4) + star_rng.IntN(4) + 1

    candidateOrbits := [1,2,3,4,5,6,7,8,9,10]
    star_rng.Shuffle(10, swap)

    occupied := boolean[1..10] all false
    for k in 1..planetCount:
        occupied[candidateOrbits[k]] = true

    planets := []
    for orbitIndex in 1..10:
        if occupied[orbitIndex]:
            planet_rng := star_rng.Child()
            planet := Planet{orbitIndex: orbitIndex, rng: planet_rng}
            planets.append(planet)

    # Planet Type Assignment MUST use planet.rng; MUST NOT call star_rng.Child().
    AssignPlanetTypes(planets, occupied)

    for planet in planets:
        GenerateDeposits(planet.rng, planet.type)
        GenerateHabitability(planet.rng, planet.type, planet.orbitIndex)

    return Star{planets}
```

---

## Location-Based Identifiers (Locked)

### Overview

All **systems, stars, orbits, planets, and orbital rings** are identified **solely by location-based identifiers**.
Identifiers are **canonical, deterministic, and stable** once assigned.

No identifier may depend on discovery order, sorting, or runtime state beyond the rules defined here.

### Coordinate Space

1. The cluster coordinate space is a three-dimensional integer grid.

2. Coordinates are represented as integer triples `(x, y, z)` where:

    * `x`, `y`, and `z` each range from **1 to 31**, inclusive.

   * The canonical enumeration order MUST be `id` increasing from `0` to `31^3-1`, which corresponds to `x` varying fastest, then `y`, then `z`.

3. The center of the cluster is fixed at:

   ```
   (16, 16, 16)
   ```

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

### Identifier Constraints

28. Identifier components are **case-sensitive**:

* orbit letters are lowercase
* planet letters are uppercase

29. Identifiers are **purely structural**:

* they encode location only,
* they do not imply physical presence, habitability, or usability.

30. Any identifier not conforming to the formats defined in this section is **invalid**.

31. Ring index MUST be in `0..100`; otherwise parsing/generation MUST panic.
32. Star sequence index MUST be in `1..starCount` for the system; otherwise MUST panic.
33. Orbit letter must be `a..j` exactly; planet letter `A..J` exactly; otherwise MUST panic.

---

## Cluster Generation (Candidate)

### Overview

The cluster coordinate space is the inclusive integer lattice:

    [1..31] × [1..31] × [1..31]

(31 units per axis).

Cluster generation proceeds in a fixed hierarchical order:

    systems → stars → orbits → planets

All generation steps are **deterministic** and **PRNG-driven**.

### Cluster Location Coordinates Assignment

1. The cluster is initialized with exactly 100 lattice points in [1..31]^3 such that for any two accepted points A and B:

       chebyshev(A,B) > 1

   where:

       chebyshev(A,B) = max(|Ax-Bx|, |Ay-By|, |Az-Bz|).

2. Candidate lattice points are produced by:
   a. Enumerating all lattice cells in [1..31]^3 into a list of length 31^3 using the canonical encoding:

          id = x0 + 31*y0 + 31*31*z0

   where (x0,y0,z0) are 0-based coordinates in [0..30].
   When outputting lattice points, coordinates MUST be converted to 1-based as (x0+1, y0+1, z0+1).

   b. Shuffling that list using rng.Shuffle (i.e., Fisher–Yates) with the provided rng.
   c. Scanning the shuffled list in order.

3. All randomness used by this algorithm **MUST** come from the provided rng.

4. Acceptance rule:
    - If a candidate point is within Chebyshev distance ≤ 1 of any previously accepted point, it is rejected.
    - Otherwise it is accepted.

   (Implementations MAY implement this rule by maintaining a blocked set and, upon accepting a point, marking its entire 3×3×3 neighborhood (Chebyshev radius 1), clipped at boundaries, as blocked.)

5. The algorithm continues scanning candidates until either:
    - 100 points have been accepted (success), or
    - the candidate list is exhausted (failure).

6. Failure to accept 100 points is a generation failure and **MUST** abort cluster generation by panicking.

7. The returned 100 lattice points are ordered by acceptance order; this order defines the systems’ generation index order i = 0..99.

### Cluster Generation Signature

```pseudocode
function GenerateClusterChebyshevShuffleDraw(
    rng  # *prng.Rand
) -> LatticePoint[100]
```

---

## System Generation (Candidate)

### Generation Scope and Order
1.	System generation MUST generate exactly 100 systems.
2.	Systems MUST be generated in generation index order `i = 0..99`.
3.	System generation MUST consume cluster output as an input array points `[100]LatticePoint`, provided in acceptance order.
4.	System `i` MUST use coordinates from `points[i]`.
5.	Implementations MUST NOT reorder, sort, filter, or otherwise permute points.

### System Coordinate Assignment
6.	Each system MUST be assigned a unique 3D location equal to `points[i]`.
7.	Each coordinate component `(X, Y, Z)` MUST be an integer in the inclusive range `[1..31]`.
8.	If any coordinate component is outside `[1..31]`, system generation MUST panic.

Invariant:
* All system locations are unique.
* System locations are entirely determined by cluster generation output and generation index.

### System PRNG Derivation
9.	For each system `i`, a system-local PRNG MUST be derived as:

    sys_rng := rng.Child()

10.	System-local PRNGs MUST be derived in generation index order `i = 0..99`.
11.	All randomness associated with a system MUST be generated using `sys_rng` or PRNGs derived from it.
12.	Implementations MUST NOT consume randomness for a system from the root `rng` after `sys_rng` has been derived.

Invariant:
* Given identical input rng state and identical points, the sequence of sys_rng instances is identical.

### Star Count per System
13.	The number of stars in a system MUST be determined solely by the system’s generation index `i`, according to the following fixed mapping:

| Generation Index `i` | Star Count |
|----------------------|------------|
| `i == 0`             | 4          |
| `1 ≤ i ≤ 8`          | 3          |
| `9 ≤ i ≤ 24`         | 2          |
| `i ≥ 25`             | 1          |

14.	This mapping is fixed, invariant, and MUST NOT change based on:

* system coordinates,
* PRNG state,
* star properties,
* or any other game state.

Invariant:
* For a given generation index `i`, star count is always identical across runs.

### Star PRNG Derivation
15.	For each star in a system, a star-local PRNG MUST be derived as:

    star_rng := sys_rng.Child()

16.	Star-local PRNGs MUST be derived in star sequence order `1..starCount`.
17.	All randomness associated with a star (including star properties and star-local generation) MUST use `star_rng`.
18.	Implementations MUST NOT derive star-local PRNGs from the root `rng`.

Invariant:
* Star-local PRNG derivation order is independent of system coordinates and depends only on generation index and star sequence.

### Summary of Invalid-Input Behavior
* Invalid system coordinate values MUST cause a panic.
* Reordered, missing, or malformed points input MUST cause a panic.
* Any deviation from the specified PRNG derivation order violates determinism and is non-conformant.

---

## Star Generation (Candidate)

### Scope
1.	Star generation MUST create only the orbital structure of a star.
2.	Star physical characteristics (including but not limited to temperature, mass, color, luminosity) MUST NOT be generated and MUST NOT affect gameplay.

Invariant:
* Star generation affects only orbit and planet placement.

### Orbit Definitions
3.	Each star MUST have exactly 10 orbits, indexed 1..10.
4.	Orbit indices MUST map canonically to letters as follows:

    1  → a / A
    2  → b / B
    …
    10 → j / J

5.	Lowercase letters MUST denote empty orbits.
6.	Uppercase letters MUST denote occupied orbits (planets).
7.	Empty orbits are implicit and MUST NOT be stored as entities.

Invariant:
* Orbit index ↔ letter mapping is fixed and invariant across all stars and systems.

### Planet Count Determination
8.	The number of occupied orbits (`planetCount`) for a star MUST be determined using the star’s PRNG as:

    planetCount := star_rng.IntN(4) + star_rng.IntN(4) + star_rng.IntN(4) + 1

9.	The resulting `planetCount` MUST be in the inclusive range `1..10`.

Invariant:
* Given identical `star_rng` state, `planetCount` is identical across runs.

### Candidate Orbit Construction
10.	Star generation MUST construct a list of candidate orbits containing exactly one entry for each orbit index `1..10`.

    candidateOrbits := []
    for orbitIndex = 1 .. 10:
        candidateOrbits.append(orbitIndex)

11.	Each candidate orbit MUST be uniquely identified by its `orbitIndex`.

### Orbit Shuffling
12.	`candidateOrbits` MUST be shuffled using `star_rng.Shuffle` (Fisher–Yates).
13.	The shuffle MUST be deterministic given the same `star_rng` state.
14.	Implementations MUST NOT use any shuffle algorithm other than Fisher–Yates driven by `star_rng`.

### Planet Assignment
15.	Planets MUST be assigned to the first `planetCount` entries in the shuffled `candidateOrbits` list.
16.	Orbit occupancy MUST be determined solely by this assignment.
17.	Planet entities MUST be created in increasing `orbitIndex` order from `1..10`, considering only occupied orbits.
18.	For each planet entity created, a planet-local PRNG MUST be derived as `planet.rng := star_rng.Child()` in this planet entity creation order.
19.	All remaining orbits MUST remain empty.

Invariant:
* Planet placement depends only on `star_rng` state and orbit indices, not on system location or other stars.

### Constraints and Invalid-Input Behavior
20.	Every star MUST have at least one planet.
21.	No star MUST have more than 10 planets.
22.	If `planetCount < 1` or `planetCount > 10`, star generation MUST panic.
23.	Any orbit index outside `1..10` MUST be treated as invalid and MUST panic if encountered.

### Summary Invariants
* Orbit count per star is always exactly 10.
* Planet count per star is always in the range `1..10`.
* Orbit index ↔ letter mapping is fixed and invariant.
* All randomness flows strictly from `star_rng` and its derived child PRNGs.
* Given identical `star_rng` state, star generation is fully deterministic.

---

## Orbits (Candidate)

Invariant:
* This section defines the canonical orbit index ↔ letter mapping used by Location-Based Identifiers, Star Generation, and Planet Type Assignment.

### Orbit Structure
1.	Each star MUST have exactly 10 orbits.
2.	Orbits MUST be indexed using integers in the inclusive range `1..10`.
3.	Orbit indices MUST map canonically to letters as follows:

    1  → a / A
    2  → b / B
    …
    10 → j / J

4.	Lowercase letters MUST denote empty orbits.
5.	Uppercase letters MUST denote occupied orbits.
6.	An occupied orbit MUST be considered a planet.

Invariant:
* The number of orbits per star is always exactly 10.

### Canonical Encoding Rules
7.	Orbit index → letter mapping MUST be fixed and invariant.
8.	Orbit occupancy MUST be encoded solely by letter casing.
9.	Implementations MUST NOT encode orbit occupancy using:
* additional flags,
* numeric annotations,
* or any mechanism other than letter casing.

Invariant:
* Given the same orbit index and occupancy state, the encoded orbit letter is always identical.

### Independence Constraints
10.	Orbit index assignment and letter mapping MUST NOT depend on:
* star type,
* system location,
* planet type,
* PRNG state,
* generation order,
* or any other game state.

Invariant:
* Orbit identity and representation are independent of all non-orbit properties.

### Invalid-Input Behavior
11.	Any orbit index outside the inclusive range `1..10` MUST be treated as invalid and MUST cause generation to panic.
12.	Any orbit letter outside the sets `'a'..'j'` or `'A'..'J'` MUST be treated as invalid and MUST cause generation to panic.
13.	Any orbit letter that is not a single ASCII character MUST be treated as invalid and MUST cause generation to panic.

### Orbit Occupancy Test (Normative)

```pseudocode
function IsOccupiedOrbit(orbitLetter):
    assert orbitLetter is a single ASCII letter
    return orbitLetter is uppercase
```

### Summary Invariants
* Each star has exactly 10 orbits.
* Orbit indices are always `1..10`.
* Orbit occupancy is represented exclusively by letter casing.
* Orbit representation is fully deterministic and context-independent.
* Invalid orbit indices or letters always cause generation to panic.

---

## Planet Type Assignment (Candidate)

### Definitions
1.	Orbits MUST be indexed from 1 (innermost) to 10 (outermost).
2.	Only occupied orbits MUST receive a planet type.
3.	Valid planet types are exactly:
* rocky
* gas-giant
* asteroid-belt
4.	An occupied orbit together with an assigned planet type MUST be considered a planet.

Invariant:
* Unoccupied orbits never have a planet type.

### Generation Order and PRNG Usage
5.	Planets MUST be processed in increasing orbit index order (1..10).
6.	Each planet entity MUST already have an assigned `planet.rng` derived during Star Generation (Rule 18 of Star Generation).
7.	Planet type assignment MUST use `planet.rng`.
8.	Implementations MUST NOT call `star_rng.Child()` (or any other RNG derivation) during planet type assignment.

Invariant:
* Given identical `star_rng` state and orbit occupancy, planet type assignment is deterministic.

### Planet entity creation order (normative)
```pseudocode
# Given: occupied[1..10] boolean for a star, and star_rng already exists.
# Create planet entities and their RNGs deterministically.

function CreatePlanetsForStar(star_rng, occupied[1..10]) -> planets[]:
    planets := []

    for orbitIndex in 1..10:
        if occupied[orbitIndex] == true:
            planet := new Planet()
            planet.orbitIndex = orbitIndex
            planet.rng = star_rng.Child()
            planets.append(planet)

    return planets
```

### Hard Override Rules
9.	If a star has exactly one occupied orbit, that planet MUST be assigned the type `gas-giant`.
10.	When Rule 9 applies, no further planet-type rules MUST be evaluated for that star.

### Zone Classification
11.	Orbits MUST be classified into zones by orbit index:

| Orbit Index | Zone   |
|-------------|--------|
| 1..3        | Inner  |
| 4..6        | Middle |
| 7..10       | Outer  |

Invariant:
* Zone boundaries are fixed and invariant.

### Planet Type Assignment Rules

(Applied only if Rule 9 does not apply)

#### Inner Zone (Orbits 1–3)
12.	Any occupied orbit in the Inner zone MUST be assigned type `rocky`.

#### Middle Zone (Orbits 4–6)
13.	For each occupied orbit `o` in the Middle zone, planet type MUST be assigned as follows:

    if o + planet.rng.IntN(3) >= 7:
        gas-giant
    else:
        rocky

#### Outer Zone (Orbits 7–10)
14.	The default planet type for an occupied orbit in the Outer zone MUST be `gas-giant`.
15.	If an occupied orbit `o` in the Outer zone has its next inner orbit (`o-1`) unoccupied, then orbit `o` MUST be assigned type `asteroid-belt` instead.

Invariant:
* Outer-zone asteroid belts arise only from adjacency to an empty inner orbit.

### Sanity Constraints (Post-Assignment Corrections)

#### Asteroid Belt Limit
16.	A star MUST NOT have more than 2 asteroid belts.
17.	If more than 2 asteroid belts are assigned:

* Excess belts MUST be converted in increasing orbit index order.
* If the orbit index is ≥ 9, the belt MUST be converted to `gas-giant`.
* Otherwise, it MUST be converted to `rocky`.

### Variety Guarantee
18.	If a star has 3 or more planets and no `rocky` planets, the innermost occupied orbit MUST be reassigned to type `rocky`.

### Determinism and Independence Constraints
19.	Planet type assignment MUST NOT depend on:

* system coordinates,
* star physical properties,
* absolute distances or geometry beyond orbit index and adjacency.

20.	Planet type assignment MUST depend only on:

* input PRNG state,
* orbit index,
* orbit occupancy,
* neighboring orbit occupancy.

Invariant:
* Two implementations with identical inputs and PRNG state produce identical planet types.

### Invalid-Input Behavior
21.	Any occupied orbit index outside `1..10` MUST cause planet type assignment to panic.
22.	Any planet type outside the defined set {`rocky`, `gas-giant`, `asteroid-belt`} MUST cause generation to panic.

### Summary Invariants
* Planet types are assigned only to occupied orbits.
* Inner-zone planets are always `rocky` (unless overridden by single-planet rule).
* Outer-zone belts arise only from gaps.
* No star has more than two asteroid belts.
* Determinism is guaranteed by strict PRNG usage and fixed rule order.

---

## Natural Resource Deposits Generation (Candidate)

### Scope and PRNG Usage
1.	Natural resource deposits MUST be generated only for planets.
2.	All randomness used in deposit generation MUST use `planet.rng`.
3.	No other PRNG source MUST be consulted at any step.

Invariant:
* Given identical `planet.rng` state and planet type, resource generation is deterministic.

### Definitions and Constants
4.	Valid resource kinds are exactly:
* METALLICS
* NON_METALLICS
* FUEL
* GOLD
5.	Valid yield ranges MUST be:
* METALLICS, NON_METALLICS: 1..10
* FUEL: 1..6
* GOLD: 1..3
6.	Valid quantity ranges MUST be:
* METALLICS, NON_METALLICS, FUEL: 1..99,999,999
* GOLD: 1..9,999,999

7. All deposit arithmetic MUST be performed using integer math.
 
* floor_div(a, b) MUST return a / b for non-negative integers.
* ceil_div(a, b) MUST return (a + b - 1) / b for non-negative integers.

Invariant:
* No generated deposit may violate its defined yield or quantity range prior to later modifiers.

### Step 1 — Deposit Count
8.	For each planet, the number of deposits MUST be determined by exactly one roll, based on planet type:

| Planet Type     | Deposit Count Rule  |
|-----------------|---------------------|
| `rocky`         | `1d12 + 2` (3..14)  |
| `gas-giant`     | `1d18 + 6` (7..24)  |
| `asteroid-belt` | `1d10 + 6` (7..16)  |

9. Deposit generation MUST create exactly this number of deposits before any later conversions.

### Step 2 — Deposit Kind Assignment
10.	For each deposit, a single roll of `1d100` MUST be used to assign its resource kind.
11.	Resource kind assignment MUST follow the exact tables below.

Rocky planets:

| Roll   | Resource      |
|--------|---------------|
| 01     | GOLD          |
| 02–16  | FUEL          |
| 17–61  | METALLICS     |
| 62–100 | NON_METALLICS |

Gas-giant planets:

| Roll   | Resource       |
|--------|----------------|
| 01     | GOLD           |
| 02–66  | FUEL           |
| 67–83  | METALLICS      |
| 84–100 | NON_METALLICS  |

Asteroid-belt planets:

| Roll   | Resource       |
|--------|----------------|
| 01–02  | GOLD           |
| 03–12  | FUEL           |
| 13–77  | METALLICS      |
| 78–100 | NON_METALLICS  |

### Step 3 — Quantity Determination
12.	For deposits of type METALLICS, NON_METALLICS, or FUEL, quantity MUST be determined as:

    quantity := sum of 99 rolls of 1..1,000,000

13.	For deposits of type GOLD, quantity MUST be determined as:

    quantity := sum of 9 rolls of 1..1,000,000

14.	Quantities MUST fall within the ranges defined in Rule 6 prior to modifiers.

### Step 4 — Yield Determination
15.	Yield MUST be determined as follows:

* METALLICS, NON_METALLICS: 1d10
* FUEL: 1d6
* GOLD: 1d3

16.	Yield MUST be stored exactly as rolled before modifiers.

### Step 5 — Planet-Type Modifiers

#### 5A — GOLD on Asteroid Belts
17.	If the planet type is `asteroid-belt` and the deposit kind is GOLD:

* A bonus percentage MUST be computed as 25 + 1d51.
* Quantity MUST be updated as:

    quantity := floor(quantity * (100 + bonusPct) / 100)

* Yield MUST be set to 1.

#### 5B — FUEL on Gas Giants
18.	If the planet type is `gas-giant` and the deposit kind is FUEL:

* A bonus percentage MUST be computed as 1d25.
* Quantity MUST be updated as:

    quantity := ceil_div(qty * (100+bonusPct), 100) # with overflow check

#### 5C — Yield Scaling by Planet Type
19.	After applying Rules 16–17, yield MUST be modified for all deposits as follows:

* On `asteroid-belt`: yield := ceil_div(yield, 3)
* On `gas-giant`: yield := floor(yield * 125 / 100) with a minimum of 1
* On `rocky`: no change

### Step 6 — Caps and Conversions
20.	Each deposit MUST have a computed value:

    value := quantity * yield

21.	Ties in value MUST be resolved by deposit generation order (earlier wins).

#### 6A — GOLD Cap for `rocky` and `gas-giant` planets
22.	A `rocky` or `gas-giant` planet MUST NOT have more than 4 GOLD deposits. (Asteroid belts are not limited.)
23.	If more than 4 GOLD deposits exist for a `rocky` or `gas-giant` planet:

* GOLD deposits MUST be sorted by (value ascending, generation order ascending).
* Excess deposits MUST be converted as follows, using their original yield:
  * yield 1 → convert to FUEL
  * yield 2 → convert to METALLICS
  * yield 3 → convert to NON_METALLICS
* Converted deposits MUST have:

    quantity := quantity * 10

* Yield after conversion MAY exceed normal yield ranges.

#### 6B — FUEL Cap for `rocky` and `asteroid-belt` planets
24.	A `rocky` or `asteroid-belt` planet MUST NOT have more than 12 FUEL deposits. (Gas giants are not limited.)
25.	If more than 12 FUEL deposits exist for a `rocky` or `asteroid-belt` planet:

* FUEL deposits MUST be sorted by (value ascending, generation order ascending).
* Excess deposits MUST be converted as:
  * yield in {1,3,5} → METALLICS
  * yield in {2,4,6} → NON_METALLICS

### Invalid-Input Behavior
26.	Any deposit with an undefined resource kind MUST cause generation to panic.
27.	Any `quantity` or `yield` outside defined ranges prior to modifiers MUST cause generation to panic.
28.	Any arithmetic overflow or underflow during `quantity` or `yield` computation MUST cause generation to panic.

* All intermediate computations of `quantity`, `value`, and `quantity * (100+bonusPct)` MUST be performed using at least unsigned 64-bit integer arithmetic.
* If any multiplication/addition would overflow the chosen integer type, generation MUST panic.

### Summary Invariants
* All randomness flows exclusively from `planet.rng`.
* Deposit count is fixed before any conversion.
* GOLD ≤ 4 per `rocky` or `gas-giant` planet after caps.
* FUEL ≤ 12 per `rocky` or `asteroid-belt` planet after caps.
* Conversion preserves generation order determinism.
* Given identical inputs and PRNG state, deposits are identical.

---

## Planet Habitability Generation (Candidate)

### Scope and Definitions
1.	Each planet MUST have a Habitability value.
2.	Habitability MUST be an integer in the inclusive range `[0..25]`.
3.	All randomness used to generate Habitability MUST use the planet’s deterministic PRNG `planet.rng`.
4.	No other PRNG source MUST be consulted.

Invariant:
* Given identical `planet.rng` state, orbit index, and planet type, Habitability is deterministic.

### Helper Functions (Normative)
5.	`RollNdS(n, s)` MUST return the sum of `n` independent rolls of integers in `[1..s]`.
6.	`RollPercent()` MUST return an integer in `[1..100]`.
7.	`Clamp(x, lo, hi)` MUST return:

* `lo` if x < lo
* `hi` if x > hi
* otherwise `x`

### Rocky Planet Habitability

#### Two-Phase Generation
8.	Habitability for `rocky` planets MUST be generated in two phases, in order:
* Habitable gate
* Capped bell roll

#### Rocky Habitable Gate
9.	Let `orbit` be the planet’s orbit index `(1..10)`.
10.	The `rocky` habitable gate probability `P_rocky[orbit]` MUST be defined as:

| Orbit | 1  | 2  | 3  | 4  | 5  | 6  | 7  | 8 | 9 | 10 |
|-------|----|----|----|----|----|----|----|---|---|----|
| %     | 10 | 20 | 40 | 80 | 40 | 20 | 10 | 5 | 2 | 1  |

11.	A roll `r := RollPercent()` MUST be performed.
12.	If `r > P_rocky[orbit]`, then:

* Habitability MUST be set to 0
* Habitability generation for this planet MUST terminate

13.	If `r ≤ P_rocky[orbit]`, generation MUST continue to the capped bell roll.

#### Rocky Capped Bell Roll

##### Orbit Maximum
14.	The maximum Habitability value `Hmax_rocky[orbit]` MUST be defined as:

| Orbit | 1 | 2  | 3  | 4  | 5  | 6  | 7 | 8 | 9 | 10 |
|-------|---|----|----|----|----|----|---|---|---|----|
| Hmax  | 8 | 16 | 22 | 25 | 22 | 16 | 8 | 4 | 2 | 1  |


##### Base Bell Roll
15.	A base value `b` MUST be computed as:

    b := RollNdS(3, 6) - 3

16.	`b` MUST be an integer in `[0..15]`.

##### Scaling and Assignment
17.	Let `Hmax := Hmax_rocky[orbit]`.
18.	A provisional habitability value `h` MUST be computed as:

    h := floor(b * Hmax / 15)

19.	If the planet passed the habitable gate and `h == 0`, then `h` MUST be set to 1.
20.	Final Habitability MUST be assigned as:

    Habitability := Clamp(h, 0, 25)

### Gas Giant Habitability

#### Two-Phase Generation
21.	Habitability for `gas-giant` planets MUST be generated in two phases, in order:

* Habitable gate
* Moon habitability roll

#### Gas Giant Habitable Gate
22. `P_gas[orbit]` MUST equal `ceil_div(P_rocky[orbit], 2)` when expressed as an integer percent used with RollPercent().

23.	The resulting probabilities MUST be:

| Orbit | 1 | 2  | 3  | 4  | 5  | 6  | 7 | 8 | 9 | 10 |
|-------|---|----|----|----|----|----|---|---|---|----|
| %     | 5 | 10 | 20 | 40 | 20 | 10 | 5 | 3 | 1 | 1  |

24.	A roll `r := RollPercent()` MUST be performed.
25.	If r > P_gas[orbit], then:

* Habitability MUST be set to 0
* Habitability generation for this planet MUST terminate

26.	If `r ≤ P_gas[orbit]`, generation MUST continue to the moon habitability roll.

##### Gas giant gate percent (normative)
```pseudocode
function gas_gate_percent(P_rocky_percent):
    return ceil_div(P_rocky_percent, 2)

function ceil_div(a, b):
    # a,b are non-negative integers, b > 0
    return (a + b - 1) / b
```

#### Gas Giant Moon Habitability Roll
27.	A provisional habitability value `h` MUST be computed as:

    h := RollNdS(2, 4) - orbit

28.	If `orbit ≥ 5`, then:

    h := h + RollNdS(1, 4)

29.	`h` MUST be clamped as:

    h := Clamp(h, 0, 25)

30.	If the planet passed the habitable gate and `h == 0`, then `h` MUST be set to 1.
31.	Final Habitability MUST be assigned as:

    Habitability := h

### Determinism and Independence Constraints
32.	Habitability generation MUST NOT depend on:

* system coordinates,
* star properties,
* physical distances or geometry other than orbit index,
* any PRNG other than `planet.rng`.

33.	Habitability generation MUST depend only on:

* planet type,
* orbit index,
* `planet.rng` state.

### Invalid-Input Behavior
34.	Any orbit index outside `1..10` MUST cause habitability generation to panic.
35.	Any Habitability value outside `[0..25]` after final assignment MUST cause generation to panic.
36.	Any failure to generate required random values MUST cause generation to panic.

### Summary Invariants
* Habitability is always an integer in `[0..25]`.
* Non-habitable planets always have Habitability 0.
* Habitable planets always have Habitability ≥ 1.
* Rocky planets in orbit 4 are the only planets capable of reaching Habitability 25.
* Given identical inputs and PRNG state, Habitability values are identical across implementations.

---
