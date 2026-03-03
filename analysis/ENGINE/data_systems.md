# Data Systems

## Overview

Liberal Crime Squad uses an XML-based data-driven architecture for defining game content. Item types, creature types, and augmentation types are all loaded from external XML files at startup, allowing modification without recompilation.

## XML Data Files

The following XML files are loaded during initialization (in `src/game.cpp`):

| File                | Content                              |
|---------------------|--------------------------------------|
| `vehicles.xml`      | Vehicle type definitions              |
| `clips.xml`         | Ammunition / clip type definitions    |
| `weapons.xml`       | Weapon type definitions               |
| `armors.xml`        | Armor type definitions                |
| `creatures.xml`     | NPC and creature type definitions     |
| `augmentations.xml` | Character augmentation definitions    |

Each file is parsed by `populate_from_xml()`, which reads XML elements and populates the corresponding type registry (e.g., `weapontype[]`, `armortype[]`, `creaturetype[]`).

## Site Map Data

Site layouts are defined through a combination of formats:

| Format   | Files                              | Purpose                          |
|----------|------------------------------------|----------------------------------|
| `.csv`   | Bank.csv, WhiteHouse.csv, etc.    | Tile-based site map definitions   |
| `.txt`   | `sitemaps.txt`                     | Site map configuration and layout rules |

The `readConfigFile()` function loads `sitemaps.txt` to determine how procedural site generation combines with hand-crafted CSV tile data.

## Graphics and Media

| Format   | Files                              | Purpose                          |
|----------|------------------------------------|----------------------------------|
| `.cpc`   | bigletters.cpc, newstops.cpc, etc.| Compressed pixel character graphics |
| `.cmv`   | Various animation files            | Movie / animation data            |
| `.ogg`   | Music files in `art/`              | Background music (Ogg Vorbis)     |
| `.mid`   | MIDI files in `art/`               | Alternative music (MIDI format)   |

Graphics data is stored in multi-dimensional arrays:
- `bigletters[27][5][7][4]` — Large letter sprites.
- `newstops[6][80][5][4]` — News header graphics.
- `newspic[20][78][18][4]` — News illustration graphics.

## Configuration File

`init.txt` in the project root contains runtime configuration:

```
# Keyboard layout
pagekeys:brackets

# Save behavior
autosave:on
```

This file is read once at startup and influences input handling and save behavior.

## Data Installation

The build system (via `Makefile.am`) installs data files to `$(datadir)/lcs/art/`. This includes all XML definitions, graphics, map data, and audio assets. The game locates these files relative to its data directory at runtime.

## Modding

Because game content is defined in external XML files, the data system supports modding:

- **New weapons**: Add entries to `weapons.xml` with attack properties, skill requirements, and wound types.
- **New armor**: Add entries to `armors.xml` with coverage, armor ratings, and special properties.
- **New creatures**: Add entries to `creatures.xml` with attributes, skills, equipment, and AI behavior.
- **New augmentations**: Add entries to `augmentations.xml` with stat effects, costs, and requirements.
- **New site layouts**: Create CSV files and reference them in `sitemaps.txt`.
