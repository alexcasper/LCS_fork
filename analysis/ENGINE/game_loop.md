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

## Activity Scheduling

The daily cycle includes an activity dispatcher that processes squad member assignments. Each activity type has a dedicated handler function:

| Handler                | Activity                                      |
|------------------------|-----------------------------------------------|
| `doActivitySolicit()`  | Fundraising through donations                 |
| `doActivityGraffiti()` | Spray-painting propaganda                     |
| `doActivityProstitution()` | Underground fundraising                  |
| `doActivityTeach()`    | Training other squad members                  |
| `doActivityRecruit()`  | Recruiting new members from the public        |
| `doActivitySteal()`    | Theft operations for funding                  |

Activities execute probabilistically — not every assigned activity triggers each day. Success rates and output values scale with the relevant skill, while failure can trigger arrest attempts and increase location heat.

## Event Processing

The monthly cycle processes several event categories in sequence:

1. **Law proposals** — Laws may be put to simulated public votes (1000 voters).
2. **Congressional bills** — House and Senate vote based on member alignments.
3. **Supreme Court rulings** — Justices rule on constitutional challenges with built-in biases.
4. **Elections** — Senate/House every 2 years, presidential every 4 years.
5. **Sleeper agent updates** — Sleeper agents in institutions produce effects based on their roles.
6. **Endgame evaluation** — Win and loss conditions are checked after all political processing.

## News Generation Pipeline

At the end of each day with newsworthy events, the news engine processes stories through a pipeline:

1. **Event creation** — Squad actions, major events, and CCS activities generate news story objects.
2. **Priority scoring** — Each story receives a 0–100+ priority based on squad size, violence level, and site profile.
3. **Page assignment** — A greedy algorithm assigns the highest-priority stories to front pages.
4. **Impact calculation** — Page placement multiplies opinion impact (Page 1: ×5, Page 2: ×3, Page 3: ×2).
5. **Opinion update** — Final impact values are applied to the global `attitude[]` array.

The Liberal Guardian newspaper amplifies pro-LCS stories by an additional ×5 multiplier when player-controlled writers staff it.

## Configuration

`init.txt` provides runtime settings:

| Setting      | Options                              | Description                    |
|--------------|--------------------------------------|--------------------------------|
| `pagekeys`   | brackets / AZERTY / page             | Keyboard layout preference     |
| Autosave     | on / off                             | Automatic save each day        |
| Display fix  | ClearType options                    | Terminal rendering workarounds  |
