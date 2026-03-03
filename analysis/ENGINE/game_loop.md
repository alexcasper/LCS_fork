# Game Loop and State Management

## Main Entry Point

The game starts in `main()` (in `src/game.cpp`), which:

1. Initializes the curses display system.
2. Loads graphics resources and configuration from `init.txt`.
3. Seeds the random number generator.
4. Populates game data from XML files.
5. Calls `mode_title()` to enter the title screen loop.

## Title Mode

`mode_title()` presents the main menu with options for New Game, Load Game, High Scores, and Help. It runs a `do-while` loop awaiting player input before either creating a new game or loading a saved one, then transitions to Base Mode.

## Base Mode

`mode_base()` is the core management loop — an infinite `while(true)` that serves as the game's hub:

- Renders the base UI showing the current squad, political alignment, and available actions.
- Handles keyboard input to dispatch player commands:

| Key | Action                          |
|-----|---------------------------------|
| `w` | Wait / advance to next day      |
| `f` | Deploy squad on an operation     |
| `a` | Activate and manage squads       |
| `z` | Fundraising activities           |
| `r` | Review mode (characters, stats)  |
| `l` | View the Liberal Agenda          |
| `x` | Exit to title screen             |

- Checks for siege conditions and game-ending states each iteration.
- When the player deploys a squad, control transfers to Site Mode.

## Site Mode

`mode_site()` handles tactical operations at target locations:

- Initializes a 3D tile map (`MAPX × MAPY × MAPZ` grid).
- Places the player squad at the entry point (`locx, locy, locz`).
- Runs a turn-based loop where the player explores, encounters NPCs, engages in combat, and acquires loot.
- Sleeper agents provide pre-mapped site knowledge.
- Siege mode alters the map with enemy placements, fire, and traps.

When the squad exits or is defeated, control returns to Base Mode.

## Daily Cycle

`advanceday()` processes the passage of one day:

- Updates squad movements and ongoing activities.
- Processes vehicle assignments.
- Handles loot distribution for disbanded squads.
- Advances character health, healing, and activity timers.
- Triggers autosave at day boundaries.

## Monthly Cycle

`passmonth()` runs at the start of each new month:

- Advances the calendar: increments `month`; if month reaches 13, resets to 1 and increments `year`.
- Processes political changes: law proposals, congressional votes, Supreme Court rulings.
- Runs elections (Senate, House, Presidential) on the appropriate schedule.
- Evaluates win and loss conditions.
- Displays the political alignment summary screen.

## State Persistence

The game supports saving and loading via serialized game state. The autosave system writes at each day boundary. Save files capture the full global state: all characters, squads, vehicles, locations, political data, and the game clock.

## Configuration

`init.txt` provides runtime settings:

| Setting      | Options                              | Description                    |
|--------------|--------------------------------------|--------------------------------|
| `pagekeys`   | brackets / AZERTY / page             | Keyboard layout preference     |
| Autosave     | on / off                             | Automatic save each day        |
| Display fix  | ClearType options                    | Terminal rendering workarounds  |
