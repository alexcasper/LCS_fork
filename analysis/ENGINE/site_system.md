# Site Map System

## Overview

Liberal Crime Squad represents target locations as 3D tile grids. Site maps are generated through a three-tier fallback system: hand-crafted CSV maps, config-driven procedural templates, and legacy random generation. All approaches use a seeded RNG for reproducible layouts.

## Grid Architecture

### Dimensions

| Axis | Constant | Size | Description           |
|------|----------|------|-----------------------|
| X    | `MAPX`   | 70   | Width (centered at 35)|
| Y    | `MAPY`   | 23   | Depth                 |
| Z    | `MAPZ`   | 10   | Height (floors)       |

The full map is declared as `siteblockst levelmap[MAPX][MAPY][MAPZ]`.

### Tile Structure

Each tile is a `siteblockst` containing three fields:

```cpp
struct siteblockst {
    short special;   // Special location enum (-1 = none)
    int flag;        // Bit-packed tile properties (23 flags)
    char siegeflag;  // Siege-related state
};
```

### Tile Flags

Tile properties are stored as bit flags on the `flag` field:

| Category       | Flags                                                                         |
|----------------|-------------------------------------------------------------------------------|
| Navigation     | `SITEBLOCK_EXIT`, `SITEBLOCK_BLOCK` (impassable wall), `SITEBLOCK_DOOR`      |
| Access control | `SITEBLOCK_LOCKED`, `SITEBLOCK_KLOCK` (key lock), `SITEBLOCK_CLOCK` (combination lock), `SITEBLOCK_RESTRICTED` |
| Contents       | `SITEBLOCK_LOOT` (random loot present)                                        |
| Visibility     | `SITEBLOCK_KNOWN` (explored by player)                                        |
| Environment    | `SITEBLOCK_GRASSY`, `SITEBLOCK_OUTDOOR`, `SITEBLOCK_DEBRIS`, `SITEBLOCK_CHAINLINK`, `SITEBLOCK_METAL` |
| Damage         | `SITEBLOCK_BLOODY`, `SITEBLOCK_BLOODY2`                                       |
| Graffiti       | `SITEBLOCK_GRAFFITI`, `SITEBLOCK_GRAFFITI_CCS`, `SITEBLOCK_GRAFFITI_OTHER`   |
| Fire state     | `SITEBLOCK_FIRE_START`, `SITEBLOCK_FIRE_PEAK`, `SITEBLOCK_FIRE_END`          |
| Alarm          | `SITEBLOCK_ALARMED`                                                           |

## Special Locations

Special tiles mark interactive points of interest. There are 42 special location types (excluding `SPECIAL_NONE`), including:

| Category     | Examples                                                        |
|--------------|-----------------------------------------------------------------|
| Traversal    | `STAIRS_UP`, `STAIRS_DOWN`                                      |
| Objectives   | `NUCLEAR_ONOFF`, `BANK_VAULT`, `INTEL_SUPERCOMPUTER`, `CORPORATE_FILES` |
| Controls     | `POLICESTATION_LOCKUP`, `PRISON_CONTROL` (3 security levels)    |
| NPCs         | `APARTMENT_LANDLORD`, `CLUB_BOUNCER`, `BANK_TELLER`, `CCS_BOSS` |
| Environment  | `RESTAURANT_TABLE`, `CAFE_COMPUTER`, `PARK_BENCH`               |
| Security     | `SECURITY_CHECKPOINT`, `SECURITY_METALDETECTORS`                |

## Map Generation Pipeline

Site initialization in `initsite()` follows a three-tier fallback:

```
initsite()
  ├─→ readMap()          [CSV maps — highest priority]
  ├─→ build_site()       [Config-driven procedural generation]
  └─→ generateroom()     [Legacy random generation — fallback when the global `oldMapMode` flag is set]
```

The location's `mapseed` value seeds the RNG before generation, ensuring the same location always produces the same layout.

### Tier 1: CSV Maps

Hand-crafted maps are edited with the DAME level editor and stored as paired CSV files:

| File Pattern                          | Contents              |
|---------------------------------------|-----------------------|
| `mapCSV_[Name]_Tiles.csv`            | Tile flags per cell   |
| `mapCSV_[Name]_Specials.csv`         | Special location IDs  |

Multi-floor buildings use numbered suffixes (`_2_Tiles.csv`, `_3_Tiles.csv`). The loader reads floors sequentially until a file is missing, allowing variable-height buildings.

**Safehouse handling**: When a location is used as a safehouse, the loader strips high-security flags (`LOCKED`, `RESTRICTED`, `ALARMED`) and clears objective markers.

### Tier 2: Config-Driven Templates

The `build_site()` function reads named templates from the site configuration system. Templates support several command types:

| Command   | Purpose                                                    |
|-----------|------------------------------------------------------------|
| `TILE`    | Sets, adds, or subtracts tile flags in a defined region    |
| `SCRIPT`  | Runs generation algorithms (ROOM, HALLWAY_YAXIS, STAIRS, STAIRS_RANDOM) |
| `SPECIAL` | Places special locations at random with defined frequency  |
| `UNIQUE`  | Places unique one-off specials (distinct from SPECIAL)     |
| `LOOT`    | Defines loot placement rules                               |

The `initsite()` switch in `build_site()` references 24 distinct template names. Templates can inherit from others using the `USE` command.

### Tier 3: Legacy Procedural Generation

The `generateroom()` function uses a **recursive binary space partition** algorithm:

1. Start with a solid rectangular region.
2. Randomly split the space with a wall along the X or Y axis.
3. Place a door at a random point on the wall (~33% chance locked).
4. Recurse on each sub-region until the minimum size (2×2) is reached or a random early-stop triggers for 3×3+ regions.

### Hallway Generation

`generatehallway_y()` creates Y-axis corridors with apartments on both sides:

- A door is spawned every 4 tiles along the hallway.
- `generateroom()` is called for each side room.
- The result simulates apartment buildings and office floors.

### Stairwell Placement

| Function                 | Strategy                                           |
|--------------------------|----------------------------------------------------|
| `generatestairs()`       | Deterministic placement, alternating sides per floor |
| `generatestairsrandom()` | Intelligent placement in secure/unsecure zone boundaries |

## Post-Generation Processing

After the base map is generated, several finishing passes run:

### Restriction Propagation

An iterative flood-fill algorithm unrestricts tiles adjacent to already-unrestricted tiles. Doors between restricted and unrestricted zones are locked. Doors between unrestricted zones are unlocked. The pass repeats until the restriction map stabilizes.

### Loot Placement

During a post-generation pass, tiles marked `SITEBLOCK_RESTRICTED` have a ~10% chance (`!LCSrandom(10)`) of receiving the `LOOT` flag for random item generation during gameplay. Several site types are explicitly excluded from getting random loot.

### Special Objective Placement

Site-specific objectives (lockups, vaults, supercomputers) are placed in appropriate restricted zones based on the location type.

### Graffiti Quota

Certain location types enforce a minimum graffiti count (parks: 5 tags, crack houses: 30 tags) applied during generation.

### Persistent Changes

The location's `changes` vector stores semi-permanent modifications (damage, graffiti, fire states) that are reapplied each time the site is regenerated, ensuring player actions persist across visits.
