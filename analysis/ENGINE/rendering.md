# Rendering and Display

## Terminal Rendering

Liberal Crime Squad uses **ncurses** (or **PDCurses** on Windows) for all display output. The game renders entirely in a text terminal, following the classic roguelike tradition.

### Initialization

The rendering system initializes in `src/cursesgraphics.cpp`:

1. `initscr()` — Opens the curses screen.
2. `curs_set(0)` — Hides the cursor.
3. `raw_output()` — Enables raw input mode (no line buffering).
4. `keypad()` — Enables special key detection (arrows, function keys).
5. Color pairs are initialized as an 8×8 color matrix (foreground × background), providing 64 color combinations.

### Drawing Primitives

All rendering uses standard curses functions:

| Function     | Purpose                                  |
|--------------|------------------------------------------|
| `erase()`    | Clears the screen for a new frame        |
| `move(y, x)` | Positions the cursor                    |
| `addstr()`   | Writes text at the cursor position       |
| `set_color()` | Sets the foreground/background color pair |
| `refresh()`  | Flushes the frame buffer to the terminal  |

### UI Elements

- **Box drawing** uses special characters (`\x11`, `\x10`) for borders and frames.
- **Full-screen redraws** occur each frame — the engine erases and redraws the entire display rather than updating individual regions.
- **Color coding** conveys alignment, health status, and UI state throughout the interface.

## Input Handling

Input is read through curses' `getch()` with keypad mode enabled. The `init.txt` configuration allows selecting keyboard layouts:

| Layout     | Page Navigation Keys |
|------------|---------------------|
| `brackets` | `[` and `]`         |
| `AZERTY`   | AZERTY-compatible    |
| `page`     | Page Up / Page Down  |

## Optional SDL2 Graphics

The engine supports an optional **SDL2** graphics layer for enhanced display:

- Enabled at compile time; can be disabled with the `DONT_INCLUDE_SDL` preprocessor flag.
- When active, SDL2 provides a graphical window instead of (or alongside) the terminal.
- **SDL2_mixer** handles audio playback for background music and sound effects.

### Audio System

Music is loaded from the `art/` directory in two supported formats:

| Format     | Library      | Description              |
|------------|-------------|--------------------------|
| Ogg Vorbis | libogg/libvorbis | Compressed audio files |
| MIDI       | SDL2_mixer   | Sequenced music          |

The audio system plays background music during gameplay and can switch tracks based on game context (combat, base management, news events).

## Graphics Assets

Pre-rendered pixel art is stored in `.cpc` (Compressed Pixel Character) format and loaded into multi-dimensional arrays for display:

- **Large letters**: 27 characters × 5 × 7 × 4 pixel grids — used for title screens and headlines.
- **News headers**: 6 variants × 80 × 5 × 4 — newspaper masthead graphics.
- **News illustrations**: 20 images × 78 × 18 × 4 — story accompanying graphics.

Animation data is stored in `.cmv` files for sequenced visual effects.

## Cross-Platform Support

| Platform | Display Library | Audio Library | Notes                        |
|----------|----------------|---------------|------------------------------|
| Linux    | ncurses/ncursesw | SDL2_mixer  | Primary development target    |
| Windows  | PDCurses       | SDL2_mixer    | DLLs bundled in repository    |
| macOS    | ncurses        | SDL2_mixer    | Via Homebrew dependencies     |

Windows builds bundle `pdcurses.dll`, `SDL2.dll`, `SDL2_mixer.dll`, and Vorbis libraries directly in the repository root.
