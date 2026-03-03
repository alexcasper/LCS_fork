# Site Operations

## Overview

Site operations are the tactical heart of Liberal Crime Squad. When a squad deploys to a target location, the game enters **Site Mode** — a turn-based exploration and infiltration system built on a 3D tile map. Players navigate rooms and corridors, encounter NPCs, engage in combat, acquire loot, and complete objectives.

## Site Mode Entry

When a squad is deployed, `mode_site()` initializes the operation:

1. The site type is set from the target location.
2. Alarm state resets: `sitealarm = 0`, `sitealarmtimer = -1`, `postalarmtimer = 0`.
3. The squad spawns at the map entrance tile.
4. If a sleeper agent is embedded at the location, the map is pre-revealed.

## Map Generation

Site maps are generated using a priority system:

1. **CSV maps** — Hand-crafted layouts exported from a tile editor (e.g., Bank.csv, WhiteHouse.csv) are loaded first.
2. **Procedural scripts** — If no CSV exists, the engine uses generation scripts defined in `sitemaps.txt`.
3. **Random rooms** — As a final fallback, rooms are placed randomly within the grid.

### Tile Types

The map is a 3D grid (`MAPX × MAPY × MAPZ`) where each tile carries flag-based properties:

| Tile Flag              | Description                                      |
|------------------------|--------------------------------------------------|
| `SITEBLOCK_BLOCK`      | Impassable wall                                  |
| `SITEBLOCK_DOOR`       | Passable door (may be locked, metal, or alarmed) |
| `SITEBLOCK_EXIT`       | Map entrance or exit point                       |
| `SITEBLOCK_LOOT`       | Item pickup location                             |
| `SITEBLOCK_KNOWN`      | Tile explored by the squad                       |
| `SITEBLOCK_RESTRICTED` | Triggers alarm on unauthorized entry             |
| `SITEBLOCK_FIRE_*`     | Fire tiles (start, peak, and end states)         |

### Procedural Generation Scripts

| Script                        | Description                                      |
|-------------------------------|--------------------------------------------------|
| `SITEMAPSCRIPT_ROOM`          | Recursive dungeon-style room generator           |
| `SITEMAPSCRIPT_HALLWAY_YAXIS` | Linear hallways with rooms branching off         |
| `SITEMAPSCRIPT_STAIRS`        | Multi-floor stairwell connections                |
| `SITEMAPSCRIPT_STAIRS_RANDOM` | Random stair placement per floor                 |

Apartment buildings use a specialized generator: 1–6 random floors, left/right room layouts with doors, a landlord placed on floor 1, and staircase connections between floors.

## Stealth and Detection

Site operations begin in a covert state. The squad must avoid detection to operate freely.

### Detection Check

The `noticecheck()` function scans all NPCs in the current encounter for potential witnesses. It compares the best party member's **Stealth** skill against a difficulty determined by the NPC type:

| NPC Type               | Detection Difficulty |
|------------------------|---------------------|
| Generic civilians      | Very Easy (3)       |
| Police, SWAT, gangs    | Easy (5)            |
| Bouncers, security     | Challenging (7)     |
| CEOs, judges, anchors  | Hard (9)            |
| Secret Service         | Formidable (13)     |
| Guard dogs             | Heroic (15)         |

### Detection Modifiers

| Condition                      | Effect                          |
|--------------------------------|---------------------------------|
| `sitealarmtimer == 1`          | +6 to detection difficulty      |
| `sitealarmtimer > 1`           | +3 to detection difficulty      |
| Party size > 1                 | +3 per additional member        |

### Detection Sequence

When a suspicious situation is triggered (naked member, visible weapon, restricted area, or active alarm timer):

1. **Stealth check** — Best party Stealth skill vs. difficulty.
2. **Disguise check** — If Stealth fails, best party Disguise skill vs. difficulty.
3. **Detection** — If both fail, `sitealarm` is set to 1 and combat begins.

Visible weapons bypass disguise checks entirely — a weapon score of 2 or higher causes instant detection.

### Rejection Reasons

At guarded locations (clubs, restricted areas), the squad can be rejected for:

| Reason                    | Description                           |
|---------------------------|---------------------------------------|
| `REJECTED_NUDE`           | Squad member is naked                 |
| `REJECTED_WEAPONS`        | Visible weapons detected              |
| `REJECTED_BLOODYCLOTHES`  | Bloodstained clothing                 |
| `REJECTED_DAMAGEDCLOTHES` | Worn-out or damaged clothing          |
| `REJECTED_DRESSCODE`      | Does not meet location dress code     |
| `REJECTED_SMELLFUNNY`     | Suspicious odor                       |
| `REJECTED_UNDERAGE`       | Member is underage                    |
| `REJECTED_CROSSDRESSING`  | Gender-restricted venue violation     |

Sleeper agents embedded as bouncers automatically admit the squad.

## Encounters

### Enemy Spawning

`prepareencounter()` generates 1–6 enemies per encounter based on the site type. Each site has weighted enemy tables:

| Site Type       | Primary Enemies          | Secondary Enemies            |
|-----------------|--------------------------|------------------------------|
| Army Base       | Soldiers, Military Police | Military Officers            |
| White House     | Secret Service            | Military Officers            |
| Police Station  | SWAT, Cops               | Death Squads (if law = -2)   |
| Corporate HQ    | Mercenaries              | —                            |
| Crack House     | Gang members, teenagers  | Crackheads, bums             |
| Juice Bar       | College students, hippies | Artists, workers             |
| Cigar Bar       | Corporate managers       | Lawyers, judges, socialites  |

CCS (Conservative Crime Squad) locations add CCS Vigilantes and additional civilian types to encounters.

### Reinforcement System

After the alarm is triggered, a reinforcement timer (`postalarmtimer`) escalates over time:

| Timer Value | Effect                                             |
|-------------|-----------------------------------------------------|
| 0–60        | Suspicion phase — intermittent checks               |
| 10–40       | Sporadic spawns (1-in-5 to 1-in-3 chance per round) |
| 60–80       | Moderate escalation with warnings                   |
| > 80        | Full reinforcement teams deploy (site-specific)     |

Reinforcement composition depends on the site: police stations send SWAT teams, military bases send soldiers, and corporate sites send mercenaries.

## Site Objectives

Each site type has unique special tiles activated with the 'U' (Use) action:

| Special                         | Site Type         | Description                               |
|---------------------------------|-------------------|-------------------------------------------|
| `SPECIAL_NUCLEAR_ONOFF`         | Nuclear Plant     | Control nuclear plant operations           |
| `SPECIAL_POLICESTATION_LOCKUP`  | Police Station    | Release imprisoned liberals               |
| `SPECIAL_PRISON_CONTROL`        | Prison            | Access prison control systems              |
| `SPECIAL_CORPORATE_FILES`       | Corporate HQ      | Loot confidential documents               |
| `SPECIAL_RADIO_BROADCASTSTUDIO` | Radio Station     | Broadcast liberal messages                 |
| `SPECIAL_NEWS_BROADCASTSTUDIO`  | News Station      | Broadcast on television                    |
| `SPECIAL_BANK_VAULT`            | Bank              | Access the bank vault                      |
| `SPECIAL_BANK_TELLER`           | Bank              | Rob the bank teller                        |
| `SPECIAL_BANK_MONEY`            | Bank              | Steal cash reserves                        |
| `SPECIAL_LAB_COSMETICS_CAGEDANIMALS` | Cosmetics Lab | Free caged animals                        |
| `SPECIAL_LAB_GENETIC_CAGEDANIMALS`   | Genetics Lab  | Free genetically modified animals         |
| `SPECIAL_DISPLAY_CASE`          | Various           | Acquire unique loot items                  |
| `SPECIAL_SECURITY_CHECKPOINT`   | Various           | Security screening point                   |
| `SPECIAL_SECURITY_METALDETECTORS` | Various         | Metal detector checkpoint                  |

Broadcast studios require a **Persuasion** or **Seduction** skill check to use effectively.

## NPC Interaction

During site operations, the squad can talk to NPCs when not in active combat:

### Dialogue Skills

| Skill       | Attribute | Use                                 |
|-------------|-----------|-------------------------------------|
| Persuasion  | Charisma  | Convince NPCs on ideological issues |
| Seduction   | Charisma  | Romance-based social approach       |
| Disguise    | —         | Bluff during alarm situations       |

### Special Dialogues

- **Bank tellers**: Quiet robbery via note or demand vault access.
- **Dogs/Monsters**: Unique responses (cannot be convinced).
- **Workers and servants**: Can be freed from oppressive conditions.
- **During alarm**: Only bluffing and intimidation are available.

Drawing a weapon during a conversation immediately triggers `sitealarm = 1`.

## Alienation

The `sitealienate` variable tracks whether the squad's actions have turned the public against them:

| Value | State                   | Effect                              |
|-------|-------------------------|-------------------------------------|
| 0     | No alienation           | Normal operations                   |
| 1     | Masses alienated        | Civilian NPCs become hostile        |
| 2     | Everyone alienated      | All NPCs hostile; forces alarm      |

Excessive violence or destruction of property can alienate the masses, making stealth impossible and turning the site into open combat.

## Key State Variables

| Variable          | Type  | Purpose                                          |
|-------------------|-------|--------------------------------------------------|
| `sitealarm`       | bool  | Whether active combat is underway                |
| `sitealarmtimer`  | int   | Suspicion countdown (-1 = undetected, 0+ = suspicious) |
| `postalarmtimer`  | int   | Reinforcement escalation timer (0–100+)          |
| `sitealienate`    | 0/1/2 | Public alienation level                          |
| `sitecrime`       | int   | Cumulative crime points from this operation      |
| `encounter_timer` | int   | Consecutive combat rounds counter                |
| `encounter[]`     | array | Active NPCs in the current encounter             |
| `levelmap[][][][]`| 3D    | Tile map with flags and special markers          |
