# Engine Architecture

## Overview

Liberal Crime Squad is built in **C++11** and uses a classic roguelike architecture: a single-threaded game loop, terminal-based rendering via **ncurses** (or PDCurses on Windows), optional **SDL2** for graphics and audio, and XML-driven data loading.

## Source Structure

```
src/
├── game.cpp              # Main entry point and game loop
├── creature/             # Character system (Creature class, augmentations)
├── combat/               # Combat engine (fight, chase, kidnapping)
├── basemode/             # Headquarters management and squad activities
├── sitemode/             # Tactical site exploration and encounters
├── items/                # Weapons, armor, clips, loot definitions
├── daily/                # Daily cycle processing
├── monthly/              # Monthly events, elections, politics
├── locations/            # World map and location management
├── news/                 # News story generation and media system
├── politics/             # Political alignment, law tracking, public opinion
├── common/               # Shared utilities (equipment, high-level display helpers, actions)
└── cursesgraphics.cpp    # CP437 character translation utilities for curses backends
```

## Game Modes

The engine organizes gameplay into distinct modes, each with its own main loop:

| Mode           | Entry Point           | Description                                    |
|----------------|-----------------------|------------------------------------------------|
| **Title**      | `mode_title()`        | Main menu: new game, load, high scores         |
| **Base**       | `mode_base()`         | Squad management, activities, and planning      |
| **Site**       | `mode_site()`         | Tactical operations at target locations          |
| **Chase**      | Chase functions        | Vehicle and foot pursuit sequences              |

### Mode Flow

```
main() → mode_title() → [New Game / Load Game]
                              ↓
                        mode_base()  ←──────────────┐
                         ↓      ↑                    │
                   mode_site() ──┘                    │
                         ↓                            │
                   advanceday() → passmonth() ────────┘
```

## Key Classes

| Class        | File                   | Purpose                                      |
|--------------|------------------------|----------------------------------------------|
| `Creature`   | `creature/creature.h`  | Player and NPC characters                    |
| `Squad`      | Squad management        | Groups of up to 6 characters                 |
| `Weapon`     | `items/weapon.h`       | Weapon definitions and attack properties     |
| `Armor`      | `items/armor.h`        | Armor with coverage and quality tracking     |
| `Vehicle`    | Vehicle definitions     | Transportation and chase combat bonuses      |
| `Augmentation` | `creature/augmentation.h` | Character enhancement system             |

## Global State

The engine relies heavily on global state for simplicity and performance:

- `pool[]` — All characters (player and NPC) in the game.
- `squad[]` — All active squads.
- `vehicle[]` — All owned vehicles.
- `location[]` — All game locations.
- `activesquad` — Pointer to the currently selected squad.
- `day`, `month`, `year` — Game clock.
- `attitude[VIEWNUM]` — Public opinion per issue (0–100).
- `law[LAWNUM]` — Legal alignment per law (-2 to +2).
- `senate[]`, `house[]`, `court[]` — Government branch alignments.

## Error Handling Patterns

The engine uses several consistent patterns for error handling across the codebase:

| Pattern                    | Usage                                                       |
|----------------------------|-------------------------------------------------------------|
| NULL return checks         | File I/O functions return NULL on failure; callers check before use |
| Factory validation         | Object deserialization validates types; invalid objects are logged and discarded |
| Graceful degradation       | Missing art/data files fall back to alternative paths or defaults |
| Version guards             | Save files are version-checked on load; incompatible files are deleted |
| Diagnostic logging         | `gamelog.log()` records warnings for invalid data during load/parse |

The engine does not use exceptions. Error states are communicated through return values and NULL pointers.

## Cross-Platform Compatibility Layer

`compat.h` provides a platform abstraction for Windows API types and functions:

| Abstraction               | Linux/macOS Implementation                       |
|---------------------------|--------------------------------------------------|
| `INT32`, `INT64`, `DWORD` | Typedef'd to native C++ integer types            |
| `stricmp()`               | Case-insensitive compare via lowercase STL strings |
| `pause_ms()`              | Sleep function using POSIX timers                |
| `alarmset()` / `alarmwait()` | Signal-based timing for frame pacing          |
| `snprintf`                | Mapped to `_snprintf` on MSVC                    |

Architecture detection macros (`LCS_WIN64`, `LCS_M_IX86`) handle 32-bit vs 64-bit differences in integer sizing.

## Dependencies

| Library        | Purpose                                 | Required |
|----------------|-----------------------------------------|----------|
| ncurses/ncursesw | Terminal rendering and input           | Yes      |
| SDL2           | Optional graphical display               | No       |
| SDL2_mixer     | Background music and sound effects       | No       |
| libogg/libvorbis | Ogg Vorbis audio decoding             | No       |
| tinydir        | Cross-platform directory traversal       | Bundled  |

The build system uses **GNU Autotools** (`configure.ac`, `Makefile.am`) with a C++11 compiler requirement. SDL2 support is optional and can be disabled with the `DONT_INCLUDE_SDL` flag.
