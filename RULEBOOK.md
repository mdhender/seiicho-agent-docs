# SEIICHO

Seiicho is a recreation of Empyrean Cluster.

The goal of the game is to take control of a star cluster consisting of 100 solar systems.
Up to ten systems are inhabited; each inhabited system has one inhabited world.

## Cluster objects

### Coordinates

Coordinates are integers (x, y, z), where x, y, and z range from 1 to 31.
The center of the cluster is at (16, 16, 16).

### Systems
The "cluster" is a cube with that contains 100 systems.
The length of each edge is 31 units.
The game master may specify the minimum distance between systems when creating a new game.

Systems are identified by their coordinates, formatted as "xx-yy-zz".
For example, the system at (28, 2, 18) is "28-02-18."

### Jump Points
All systems have an entry/exit point (called the jump point) that ships must pass through.
The jump point is identified as "{systemId}/0" in reports.
For example, the jump point for system 28-02-18 is "28-02-18/0."

### Stars
There are 100 systems.
Each system contains 1 to 4 stars.

1. The first system created contains 4 stars each.
2. The next 8 systems contain 3 stars each.
4. The next 16 systems contain 2 stars.
5. The remaining 75 systems contain a single star.

Stars are identified as "{systemId}/{sequence}" in reports.
For example, the first star in system 28-02-18 is "28-02-18/1."
The second star is "28-02-18/2."

### Orbits
Every star contains 10 orbits.

Orbits may be empty or contain a gas giant, failed core, terrestrial planet, or planetoid belt.
All of these are called "planets" for convenience.

Orbits are identified by a letter, with empty orbits using "a" through "j," and non-empty orbits using "A" through "J."
For example, the third orbit in 28-02-18/1 is "28-02-18/1c" if empty, "28-02-18/1C" if not.

## Player objects

Players control Nations. 
Cluster, Systems, Stars, Orbits, Planets

Coordinates

Empires, Colonies, Ships, People

Resources, Items

Farms, Factories, Mines

## Stuff

### PRNG
Seiicho is designed to be deterministic: turns can be replayed with identical results because the state of the PRNGs used is saved with the game data and actions are always resolved in the same order.

All actors (objects) in the game are assigned a unique PRNG instance when created.
The PRNG state is saved and restored with the actor.

## Victory Conditions

## Implementation Details

This section documents differences between the rulebook concepts and the current implementation.

### Jump Points vs 11th Orbit

The rulebook describes jump points as a system-level concept (`{systemId}/0`). However, conceptually each star could have its own 11th orbit ("K") as an entry/exit point for ships.

**Current implementation**: Jump points are system-level, tracked via `Location.StarSeq == 0` or `Ship.SystemID` being set. Ships at jump points use the format `"28-02-18/0"`.

**Alternative**: Each star has an 11th orbit "K" (always empty, lowercase) that ships use to enter/exit. This would change movement to be star-specific rather than system-specific.

**Status**: Revisit when implementing movement orders.

### Orbit Letter Casing

The rulebook specifies lowercase letters (`a-j`) for empty orbits and uppercase (`A-J`) for non-empty orbits.

**Current implementation**: Only non-empty orbits are tracked in the database, always using uppercase. Empty orbits are not stored as entities.

**Status**: Current approach is sufficient since players only interact with non-empty orbits (planets).

## License
Empyrean Cluster is owned by Jay Columbo and used with his permission.

## Glossary

***Cadre***
A Cadre is a group of people temporarily banded together to act as Construction Workers, Spies, or Rebels.

***Failed Core***
A failed core is a type of "planet" with a Habitability Rating of 0.

***Gas Giant***
A gas giant is a type of "planet" with a Habitability Rating of 0.

***Habitability Rating***
A number ranging from 0 to 25 that limits the number of open Farms and people the planet may support without life support.

***Jump Point***
All ships must exit and enter systems via their jump points.
Jump points are identified by their system's coordinates plus a sequence number of 0 (e.g., 12-01-31/0).

***Nation***
Each player controls a single nation in the game.

***Planet***
A "planet" is a gas giant, failed core, terrestrial planet, or planetoid belt.
Planets are identified as their orbit but with the letter converted to uppercase (e.g., 12-01-31/2C).

***Population***
Population represents the people in a Nation.
There are four categories of Population: Unemployables, Unskilled, Professionals, and Soldiers.

***Orbit***
Orbits may contain planets are identified by their star and a one-letter sequence (e.g., the 3rd orbit is 12-01-31/2c).

***Planetoid Belt***
A planetoid belt is a type of "planet" with a Habitability Rating of 0.
It may be icy, stony, or metallic.

***Professional***
A Professional trains workers, pilots ships, manages Farms, Factories, and Mines. 

***Race***
All nations originating from the same planet are treated as being the same race for determining victory.

***Soldier***
A Soldier is a member of the military or police forces.
They may be drafted into cadres of Spies.
Disbanded Soldiers are converted to Unskilled Workers.
5% of Soldiers retire and are converted to Professionals annually (1.25% per turn).

***Star***
Stars have 10 orbits and are identified by their system's coordinates plus a sequence number (e.g. 12-01-31/2).

***System***
Systems have one or more stars and are identified by their coordinates, formatted as xx-yy-zz (e.g., 12-01-31).

***Terrestrial Planet***
A terrestrial planet is a type of "planet" with a Habitability Rating ranging from 0 to 25.

***Turn***
Each turn represents one-quarter of a year.

***Unemployable***
An Unemployable is any person that is not an Unskilled Worker, Professional, or Soldier.
Excess Unemployables may be converted to Unskilled Workers.

***Unit***
Units are items that are controlled or consumed by players.
This represents Population, Weapons, Production, and Miscellaneous.

***Unskilled Worker***
An Unskilled Worker is the base of the labor pool.
They may be drafted into cadres of Construction Workers.
