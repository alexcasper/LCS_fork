# AI, Encounters, and Detection

## Overview

Liberal Crime Squad uses several interconnected systems for NPC behavior: weighted random encounter generation, a tiered stealth/detection pipeline, per-round creature advancement with medical and fire subsystems, and a multi-stage siege escalation state machine.

## Encounter Generation

NPC encounters are generated via **weighted random selection** in `newencounter.cpp`:

1. Each site type defines an encounter pool — an array of creature types with integer weights.
2. `getrandomcreaturetype()` sums all weights, generates a random number in `[0, sum)`, then iterates through the pool subtracting weights until the random value is exhausted.
3. Encounter size is `1 + random(6)` creatures per group, scaled by the current alarm state.

### Heat-Based Escalation

When the site alarm timer (`postalarmtimer`) exceeds thresholds, the normal encounter pool is overridden with progressively heavier responses:

| Alarm Level | Encounter Override                     |
|-------------|----------------------------------------|
| Low         | Normal site encounter pool             |
| Medium      | Police and SWAT units                  |
| High        | National Guard and military responses  |
| Critical    | Heavy military and special operations  |

The `sitecrime` accumulator tracks illegal actions during a site visit. When it crosses thresholds, alarm timers activate with skill checks available to suppress them.

## Stealth and Detection

Detection uses a **two-layer check** system in `stealth.cpp`:

```
Encounter → Stealth Check → [Pass: undetected]
                           → [Fail] → Disguise Check → [Pass: blended in]
                                                      → [Fail: alarm raised]
```

### Stealth Difficulty Modifiers

| Factor              | Modifier                                          |
|---------------------|---------------------------------------------------|
| Enemy type          | Dogs = Heroic, Secret Service = Formidable, Cops = Easy |
| Scary weapons       | Increases difficulty; bypasses disguise entirely   |
| Party size          | +3 per squad member beyond the first              |
| Alarm timer active  | +3 to +6 bonus to detection difficulty            |
| Restricted area     | Disguise check unavailable                        |

### Skill Training

Stealth and disguise skills only train on "near-miss" checks — when the result plus one equals the difficulty target. This prevents trivially easy situations from granting skill experience.

### Civilian Alienation

When civilians witness crimes during site operations, they undergo an alignment shift toward conservative values. Sufficient witness alienation triggers a full site alarm and converts nearby civilians to hostile status.

## Creature Advancement

`creatureadvance()` in `advance.cpp` is the per-round state progression loop that processes all entities:

### Medical Subsystem

Each round, the system identifies the squad member with the highest First Aid skill and performs:

1. **Bleeding check** — Attempts to stabilize bleeding wounds on injured squad members.
2. **Skill roll** — Success stops bleeding; failure allows continued blood loss.
3. **Automatic triage** — Prioritizes the most critically wounded.

### Fire Mechanics

Fire follows a three-state machine applied per tile:

```
FIRE_START → FIRE_PEAK → FIRE_END → (tile cleared)
```

- Fire spreads stochastically to adjacent tiles and to the floor above.
- Characters on burning tiles take burn damage each round.
- Specialized armor (firefighter gear) reduces burn damage by 50–75%.

### Incapacitation Tracking

The system tracks per-creature state each round:

| State         | Effect                                           |
|---------------|--------------------------------------------------|
| Stunned       | Creature skips their action for the round        |
| Bleeding      | Health drains from each bleeding body part        |
| Incapacitated | Creature is down but alive; can be retrieved      |
| Dead          | Removed from active combat                        |

Incapacitated and dead squad members are automatically retrieved by adjacent allies (`squadgrab_immobile`).

## Siege System

Sieges use a **multi-stage escalation finite state machine** in `siege.cpp`:

### Hunt Phase

Before a siege begins, a countdown timer (`timeuntillocated`) runs for 2–7 days. Location heat accelerates the countdown. Sleeper agents provide a warning one day before the siege starts.

### Escalation Levels

| Level | Force Deployed       | Special Mechanics                           |
|-------|----------------------|---------------------------------------------|
| 0     | SWAT teams           | Standard tactical assault                   |
| 1     | National Guard       | Heavier weapons and armor                   |
| 2     | Tanks                | Compound wall interactions; tank traps apply |
| 3     | Bombing + SEAL teams | Aerial bombardment and elite special forces |

### Siege Types

| Type           | Trigger                          | Escalation Pattern       |
|----------------|----------------------------------|--------------------------|
| `SIEGE_POLICE` | Location heat threshold          | Standard 0→3 progression |
| `SIEGE_CCS`    | `endgamestate` progression       | CCS-specific forces      |
| `SIEGE_CIA`    | `offended_cia` flag              | CIA-specific response    |

### Casualty System

During active sieges, trapped location residents face random hit chances each round. Casualties are tracked and displayed as formatted lists. Compound fortifications (tank traps, reinforced walls) provide defensive bonuses that reduce casualty rates at higher escalation levels.

## Activity Scheduling

The daily activity dispatcher in `activities.cpp` processes squad member assignments:

### Dispatch Pattern

Each activity type has a dedicated `doActivity*()` handler that receives the list of assigned squad members. Activities execute with probabilistic success rates:

| Activity           | Trigger Rate | Skill Factor                              |
|--------------------|--------------|-------------------------------------------|
| Solicit donations  | ~1/3 per day | Persuasion determines fund amount         |
| Graffiti           | Per day      | Art skill determines opinion impact       |
| Prostitution       | ~1/3 per day | Seduction determines funds (10–1210)      |
| Teaching/Learning  | Per day      | Increase rate inversely scaled to current skill |

### Exposure Model

Failed activities can trigger criminalization and arrest attempts. Each activity carries an exposure risk that feeds into the location heat system.

### Public Opinion Coupling

Activity results feed into `change_public_opinion()` with skill-scaled impact values. Higher-skilled characters produce larger opinion shifts per successful activity.
