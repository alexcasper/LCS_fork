# Candidate Systems for GPU Acceleration

## Evaluation Criteria

Each subsystem is evaluated against four criteria:

| Criterion | Description |
|-----------|-------------|
| **Data volume** | How many independent data elements are processed per invocation |
| **Parallelism** | Whether iterations are independent (no cross-iteration dependencies) |
| **Compute intensity** | Ratio of arithmetic to memory access — higher is better for GPU |
| **Impact** | How much wall-clock time the subsystem consumes today |

Ratings: ★★★ = excellent fit, ★★☆ = good fit, ★☆☆ = marginal fit, ☆☆☆ = poor fit.

---

## High Priority

### 1. Election Simulation

**Source**: `analysis/MECHANICS/politics.md` — Elections section

The election system simulates ~1000 voters per election, each making an independent voting decision based on public opinion, party alignment, and swing-voter probability. Senate (100 seats), House (435 seats), and presidential elections all follow this pattern.

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Data volume | ★★★ | 1000 voters × multiple races per election cycle |
| Parallelism | ★★★ | Each voter decision is fully independent |
| Compute intensity | ★★☆ | Moderate — alignment lookups + random rolls |
| Impact | ★★☆ | Elections run every 2–4 game-years; monthly law votes also use voter simulation |

**GPU fit**: Excellent. Each voter's decision is a self-contained computation with no shared mutable state. A single kernel launch can resolve an entire election.

### 2. Public Opinion Batch Processing

**Source**: `analysis/MECHANICS/news_and_media.md`, `analysis/ENGINE/game_loop.md`

At the end of each news cycle, multiple stories apply opinion shifts to the 24+ element `attitude[]` array. Each story independently calculates its impact across targeted opinion categories using priority scores, reputation modifiers, and violence thresholds.

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Data volume | ★★★ | 24+ opinion categories × multiple stories per cycle |
| Parallelism | ★★☆ | Stories are independent but write to shared opinion array (reduction needed) |
| Compute intensity | ★★☆ | Moderate — formula evaluation per story × category pair |
| Impact | ★★☆ | Runs daily when newsworthy events occur |

**GPU fit**: Good. Story impact calculations are independent and can run in parallel, with a final reduction step to merge opinion deltas into the shared `attitude[]` array.

### 3. Monthly Political Processing

**Source**: `analysis/ENGINE/game_loop.md` — Monthly Cycle, `analysis/MECHANICS/politics.md`

`passmonth()` processes law proposals (1000-voter simulations), congressional votes (100 senators + 435 representatives), and Supreme Court rulings (9 justices). Each politician's vote is an independent alignment-based decision.

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Data volume | ★★★ | 535 congress members + 1000 voters + 9 justices per month |
| Parallelism | ★★★ | Each vote is independent given current public opinion state |
| Compute intensity | ★★☆ | Alignment comparison + random roll per vote |
| Impact | ★★☆ | Runs once per game-month |

**GPU fit**: Excellent. Congressional votes and proposition simulations are embarrassingly parallel.

---

## Medium Priority

### 4. Daily Activity Resolution

**Source**: `analysis/MECHANICS/activities.md`, `analysis/ENGINE/ai_and_encounters.md`

The daily cycle processes all squad member activities: recruitment attempts, fundraising, training, hacking, and sleeper agent actions. Each character's activity is a skill check with independent random rolls.

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Data volume | ★★☆ | Tens to low hundreds of characters per day |
| Parallelism | ★★☆ | Mostly independent, but some activities affect shared state (funds, heat) |
| Compute intensity | ★☆☆ | Simple skill roll + table lookup per character |
| Impact | ★★☆ | Runs every game-day |

**GPU fit**: Moderate. Character counts are typically low (tens, not thousands), limiting GPU utilization. Worth batching only when squad sizes grow large or if activity resolution is expanded.

### 5. Site Map Generation — Flood Fill

**Source**: `analysis/ENGINE/site_system.md` — Restriction Propagation

The post-generation restriction propagation pass uses an iterative flood-fill algorithm across the `MAPX × MAPY × MAPZ` (70 × 23 × 10 = 16,100 tiles) grid. Each iteration unrestricts tiles adjacent to already-unrestricted tiles, repeating until convergence.

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Data volume | ★★☆ | 16,100 tiles per site |
| Parallelism | ★★☆ | Each tile can be processed independently per iteration, but iterations depend on previous results |
| Compute intensity | ★☆☆ | Neighbor lookups + flag checks |
| Impact | ★☆☆ | Runs once per site visit |

**GPU fit**: Moderate. The iterative convergence pattern maps to a parallel wavefront approach, but the grid is small enough that CPU performance is likely adequate. Useful as a proof-of-concept kernel.

### 6. Encounter Generation

**Source**: `analysis/ENGINE/ai_and_encounters.md` — Encounter Generation

Weighted random encounter generation selects creature types from probability pools. When processing multiple simultaneous encounters (e.g., during siege escalation), each encounter draw is independent.

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Data volume | ★☆☆ | 1–6 creatures per encounter, typically one encounter at a time |
| Parallelism | ★★★ | Each creature selection is independent |
| Compute intensity | ★☆☆ | Weighted random selection — simple scan |
| Impact | ★☆☆ | Per-encounter during site mode |

**GPU fit**: Low individually, but could be batched with other per-round combat computations.

---

## Low Priority / Not Recommended

### 7. Combat Resolution

**Source**: `analysis/MECHANICS/combat.md`

Combat is turn-based with typically 1–12 combatants per side. Each attack involves sequential steps (target selection → attack roll → hit location → damage → armor → wound). Target selection depends on the current state of all combatants.

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Data volume | ★☆☆ | Small combatant counts (1–12 per side) |
| Parallelism | ★☆☆ | Sequential dependencies between attack steps; target selection reads shared state |
| Compute intensity | ★☆☆ | Low — few arithmetic operations per step |
| Impact | ★☆☆ | Real-time per-round during site mode |

**GPU fit**: Poor. Combatant counts are too small and the sequential attack pipeline does not parallelize well. Keep on CPU.

### 8. Save/Load Serialization

**Source**: `analysis/ENGINE/save_system.md`

Serialization is inherently sequential (ordered binary writes/reads). XML generation within saves is string manipulation, not compute-bound.

**GPU fit**: None. Keep on CPU.

### 9. Rendering (ncurses/PDCurses)

**Source**: `analysis/ENGINE/rendering.md`

The terminal rendering layer uses ncurses API calls that are inherently serial and host-side. The display is text-based with small buffer sizes (80×25).

**GPU fit**: None. Terminal rendering is not a GPU workload.

---

## Summary Matrix

| System | Priority | Data Volume | Parallelism | GPU Fit |
|--------|----------|-------------|-------------|---------|
| Election simulation | High | ★★★ | ★★★ | Excellent |
| Public opinion processing | High | ★★★ | ★★☆ | Good |
| Monthly political processing | High | ★★★ | ★★★ | Excellent |
| Daily activity resolution | Medium | ★★☆ | ★★☆ | Moderate |
| Site map flood fill | Medium | ★★☆ | ★★☆ | Moderate |
| Encounter generation | Medium | ★☆☆ | ★★★ | Low–Moderate |
| Combat resolution | Low | ★☆☆ | ★☆☆ | Poor |
| Save/Load | Low | — | — | None |
| Rendering | Low | — | — | None |
