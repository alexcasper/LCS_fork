# Save System and File I/O

## Overview

Liberal Crime Squad uses a binary save format with embedded XML serialization for complex objects. The file I/O layer abstracts platform-specific paths, supporting Linux, macOS, and Windows through a prefix-flag system.

## Save File Format

Save files use a **hybrid binary + XML** format stored as `.dat` files:

1. A 4-byte **version integer** is written first for compatibility checking.
2. Primitive values (integers, arrays, flags) are serialized with raw `fwrite()` / `fread()` calls.
3. Complex objects (creatures, items, vehicles) are serialized to XML strings, then written as a **length-prefixed binary blob** (size integer followed by XML character data).

### Serialization Order

| Section              | Contents                                                     |
|----------------------|--------------------------------------------------------------|
| Version header       | Format version integer                                       |
| RNG seeds            | Random number generator state for deterministic replay       |
| Game clock           | Day, month, year, and current game mode                      |
| Statistics           | Recruits, kills, kidnappings, funds raised, etc.             |
| Squads               | Squad names, member references, loot, activity status        |
| Creature pool        | All creatures serialized to XML (player and NPC)             |
| Locations            | Site loot (XML), changes, properties, heat, siege info, maps |
| Vehicles             | Vehicle definitions serialized to XML                        |
| Political state      | Attitudes array, law alignments, senate/house/court/exec     |
| Financial ledger     | Income and expense tracking                                  |
| News stories         | Stories with associated crimes and opinion view impacts       |
| Game options         | Encounter warnings, music enabled, slogan                    |

## Versioning

```cpp
const int version = 41010;              // Current save format version
const int lowestloadversion = 40100;    // Minimum loadable version
const int lowestloadscoreversion = 31203; // Minimum loadable score version
```

On load, the first integer is compared against `lowestloadversion`. If the save is too old, the file is deleted and the load fails gracefully. There is no forward compatibility — newer saves cannot be read by older game versions.

## Save and Load Functions

| Function       | Purpose                                                    |
|----------------|------------------------------------------------------------|
| `savegame()`   | Writes full game state to binary file (~280 lines of data) |
| `load()`       | Reads and reconstructs game state from binary file         |
| `reset()`      | Deletes the save file on endgame or version mismatch       |

### Object Deserialization

Complex objects use a factory pattern during load:

1. Read the size-prefixed XML string from the binary stream.
2. Pass the XML to `create_item()`, which determines the object type and constructs the appropriate class.
3. Invalid types (removed or renamed in newer versions) are logged and discarded.

## Error Handling

| Condition              | Response                                              |
|------------------------|-------------------------------------------------------|
| File not found         | `LCSOpenFile()` returns NULL; load returns failure    |
| Version too old        | Save file deleted, game state reset                   |
| Invalid item type      | Warning logged, item removed from collection          |
| Invalid vehicle type   | Warning logged, vehicle removed                       |
| Missing creature ref   | Pointer remains NULL, reference skipped               |

Diagnostic messages are written to the game log via `gamelog.log()`.

## Autosave

Autosave is controlled by a boolean flag read from `init.txt`:

```
autosave:on
```

When enabled, the game calls `savegame()` at the end of each daily turn cycle and on title screen transitions. Autosave overwrites the main save file — there is no separate autosave slot.

## File I/O Abstraction Layer

All file operations go through `lcsio.cpp`, which provides platform-aware path resolution:

| Function              | Purpose                                        |
|-----------------------|------------------------------------------------|
| `LCSOpenFile()`       | Opens a file with C `fopen()` and path prefix  |
| `LCSOpenFileCPP()`    | Opens a file with C++ `fstream` and path prefix |
| `LCSCloseFile()`      | Closes a C file handle                         |
| `LCSDeleteFile()`     | Removes a file via `remove()`                  |
| `LCSRenameFile()`     | Renames a file across directories              |
| `LCSSaveFiles()`      | Enumerates `.dat` save files using `tinydir`   |

### Path Prefix Flags

```cpp
enum LCSIO_FLAGS {
    LCSIO_PRE_ART  = 1,  // Art/data directory
    LCSIO_PRE_HOME = 2   // User home directory
};
```

### Directory Resolution

**Home directory:**

| Platform     | Path                | Notes                              |
|--------------|---------------------|------------------------------------|
| Linux/macOS  | `$HOME/.lcs/`       | Created on first use (mode 0750)   |
| Windows      | `./` (current dir)  | No creation needed                 |

**Art directory** — searched in priority order until a test file (`newspic.cpc`) is found:

1. `INSTALL_DATA_DIR/lcs/art/` (compile-time path, if defined)
2. `/usr/local/share/lcs/art/`
3. `/usr/share/lcs/art/`
4. `/usr/games/share/lcs/art/`
5. `/usr/games/lcs/art/`
6. `./art/`
7. `../art/`

### Cross-Platform Compatibility

The I/O layer uses `std::string` internally with forward-slash path separators. Directory creation calls `mkdir()` on POSIX and `_mkdir()` on Windows. Initialization is lazy-loaded on the first file operation via an `initialized` flag, ensuring paths are resolved exactly once.
