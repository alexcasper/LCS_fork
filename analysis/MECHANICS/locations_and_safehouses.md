# Locations and Safehouses

## Overview

Liberal Crime Squad features a multi-city world with dozens of location types. Locations serve as operation targets, shopping venues, recruitment grounds, and safehouses. The location system governs world generation, safehouse management, compound upgrades, and siege mechanics.

## World Structure

### Cities

The game supports multiple cities, each with distinct districts:

| City           | Description                              |
|----------------|------------------------------------------|
| Seattle        | Default starting city                    |
| New York City  | Major metropolitan area                  |
| Los Angeles    | West coast operations                    |
| Washington DC  | Political center                         |
| Chicago        | Midwest hub                              |
| Detroit        | Industrial city                          |
| Atlanta        | Southern operations                      |
| Miami          | Southeast operations                     |

### Districts

Each city is organized into districts that determine which location types appear:

| District      | Typical Locations                                      |
|---------------|--------------------------------------------------------|
| Downtown      | Corporate HQs, courthouses, banks, media offices       |
| University    | College campuses, juice bars, libraries                 |
| Industrial    | Factories, warehouses, sweatshops                      |
| Outskirts     | Military bases, prisons, nuclear plants                |
| Commercial    | Shops, apartment complexes, cigar bars                  |
| Travel        | Bus stations, airports (inter-city transit)             |

## Location Types

Locations fall into several functional categories:

### Operation Targets

Sites that can be raided during site operations:

| Location            | Key Feature                                |
|---------------------|--------------------------------------------|
| Corporate HQ        | Confidential documents, executive targets   |
| Police Station      | Prisoner lockup, evidence                  |
| Courthouse          | Judges, legal records                      |
| Prison              | Prisoners to free, control systems         |
| Nuclear Plant       | Environmental target, plant controls       |
| Army Base           | Heavy military presence                    |
| White House         | Ultimate political target                  |
| Intelligence HQ     | CIA/intelligence operations                |
| Radio Station       | Broadcast studio                           |
| News Station        | Television broadcast studio                |
| Genetics Lab        | Animal experiments, research data          |
| Cosmetics Lab       | Animal testing, research data              |
| Sweatshop           | Labor exploitation, workers to free        |
| Crack House         | Drug operations, gang encounters           |

### Commercial Locations

Sites for shopping and services:

| Location         | Function                                    |
|------------------|---------------------------------------------|
| Oubliette        | Black market arms dealer                    |
| Department Store | General equipment and clothing              |
| Pawn Shop        | Buy and sell miscellaneous items            |
| Car Dealership   | Vehicle purchases                           |

### Social Locations

Sites for recruitment and social interaction:

| Location     | Typical Encounters                            |
|--------------|-----------------------------------------------|
| Juice Bar    | College students, hippies, artists            |
| Cigar Bar    | Corporate managers, lawyers, socialites       |
| Internet Café| Tech-savvy contacts                           |
| Shelter      | Homeless population, potential recruits       |

## Safehouses

Safehouses are locations controlled by the LCS where squad members live, train, and plan operations.

### Safehouse Types

| Type              | Rent     | Base Heat Protection |
|-------------------|----------|---------------------|
| Homeless Shelter  | Free     | 0                   |
| Warehouse         | Low      | 0                   |
| Crack House       | Free     | 0                   |
| Lower Apartment   | Low      | 4                   |
| Middle Apartment  | Medium   | 8                   |
| Upper Apartment   | High     | 12                  |
| Business Front    | High     | 12                  |

### Rental Status

Each safehouse tracks its rental state:

| Status               | Value | Description                          |
|----------------------|-------|--------------------------------------|
| `RENTING_PERMANENT`  | 0     | Owned — no rent required             |
| Positive value       | > 0   | Renting — monthly cost equals value  |
| `RENTING_NOCONTROL`  | -1    | Lost — location no longer controlled |
| `RENTING_CCS`        | -2    | CCS-controlled — hostile safehouse   |

### Heat Protection

Heat protection determines how much criminal activity a safehouse can conceal before attracting a siege. The formula combines the base protection from the safehouse type with additional modifiers:

```
protection = base_heat_protection
           + flag_burning_bonus (2 to 6, based on law[LAW_FLAGBURNING])

final_protection = protection × 5
if final_protection > 95: final_protection = 95
```

The flag burning law bonus scales with the current legal status — more permissive free speech laws provide greater heat protection.

## Compound Upgrades

Warehouses and crack houses can be upgraded into fortified compounds with the following improvements:

| Upgrade            | Effect                                          |
|--------------------|-------------------------------------------------|
| Basic Compound     | Establishes the location as a defensible base   |
| Camera System      | Provides surveillance and early warning          |
| Tank Traps         | Prevents armored vehicle assault during sieges   |
| Booby Traps        | Damages attackers during siege assaults          |
| Generator          | Provides backup power during siege conditions    |
| Printing Press     | Enables Liberal Guardian newspaper production    |
| AA Gun             | Anti-aircraft defense against aerial bombing     |

Upgrades are cumulative — each adds to the compound's defensive capability.

## Siege Mechanics

When a safehouse accumulates too much heat, enemy forces may lay siege to it.

### Siege Types

| Type              | Trigger                                           |
|-------------------|---------------------------------------------------|
| Police Siege      | Heat exceeds heat protection                      |
| Corporate Siege   | Offending corporate interests + high heat         |
| CCS Siege         | CCS endgame state reached + high heat             |
| CIA Siege         | Offending intelligence agencies                   |
| Hick Siege        | AM Radio opinion ≤ 35% liberal + offending        |
| Firemen Siege     | Printing press active + offending fire department |

### Siege Probability

Each siege type has its own trigger probability:

| Siege Type   | Base Probability              | Cooldown Timer         |
|--------------|-------------------------------|------------------------|
| Police       | `random(500) < heat`          | `timeuntillocated`     |
| Corporate    | 1 in 600                      | `timeuntilcorps`       |
| CCS          | 1 in 60                       | `timeuntilccs`         |
| CIA          | 1 in 300                      | `timeuntilcia`         |
| Hicks        | 1 in 600                      | None                   |
| Firemen      | 1 in 90                       | `timeuntilfiremen`     |

### Heat Accumulation

Heat is generated by criminal activity at or near the safehouse:

| Source               | Heat Contribution                    |
|----------------------|--------------------------------------|
| Dead bodies          | +5 per corpse                        |
| Kidnapped non-liberal| +5 × days held                       |
| Member criminal heat | `heat / (active ? 10 : 60) + 1`     |

Heat naturally decays: if accumulated crime falls below current heat, heat decreases by 1 per cycle (minimum 0).

### Siege Escalation

Once a siege begins, it can escalate through increasingly dangerous phases:

| Escalation Level | Response                                        |
|------------------|-------------------------------------------------|
| Level 1          | National Guard replaces SWAT teams              |
| Level 2          | Tank deployed (blocked by tank traps)           |
| Level 3          | Aerial bombing + SEAL Team 6 assault            |

### Siege Planning

After a siege is triggered, there is a delay before the attack:

```
timeuntillocated += 2 + random(6)
```

This gives the player a window to prepare defenses, evacuate, or counterattack — especially if sleeper agents provide advance warning.
