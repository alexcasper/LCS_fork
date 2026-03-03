# Data Parallelism and Kernel Designs

## Overview

This document details the GPU kernel designs for each candidate subsystem identified in [candidate_systems.md](candidate_systems.md). Each kernel is described in terms of its thread mapping, input/output data, and parallel execution strategy.

---

## Kernel 1: Election Simulation

### Context

Elections simulate ~1000 voters deciding races for Senate (100 seats), House (435 seats), and presidency. Each voter independently evaluates candidates based on public opinion and personal alignment.

**Reference**: `analysis/MECHANICS/politics.md` — Elections section

### Thread Mapping

- **Grid**: 1D grid of 1000 threads (one per voter)
- **Block size**: 256 threads per block → 4 blocks
- **Per-thread work**: Evaluate all races for this voter, accumulate votes via atomic adds

### Data Layout

```
Input (Host → Device):
  attitude[24]          : Current public opinion array (read-only, shared)
  candidate_align[N]    : Alignment of each candidate (read-only)
  rng_seeds[1000]       : Per-voter RNG seed
  voter_params           : Party-line %, swing-voter %, opinion weights

Output (Device → Host):
  vote_tallies[N]       : Atomic vote count per candidate
```

### Parallel Strategy

Each thread simulates one voter:

1. Determine voter type (party-line ~40%, swing ~20%, opinion-driven ~40%) using thread-local RNG.
2. For each race, compare candidate alignments against the voter's effective opinion.
3. Atomically increment the chosen candidate's tally.

After kernel completion, the host reads `vote_tallies` and assigns election winners.

### Synchronization

- **Atomic adds** on `vote_tallies[]` — contention is low because 1000 threads write to 100–435 buckets.
- No inter-thread communication needed within the voter decision logic.

---

## Kernel 2: Public Opinion Batch Update

### Context

After the news cycle, multiple stories apply opinion shifts to the `attitude[24+]` array. Each story targets specific opinion categories with independent impact calculations.

**Reference**: `analysis/MECHANICS/news_and_media.md` — Public Opinion Impact

### Thread Mapping

- **Grid**: 2D — `stories × opinion_categories`
- **Block size**: 32 × 8 (stories × categories) — tuned for typical story counts (5–20 per cycle)
- **Per-thread work**: Compute one story's impact on one opinion category

### Data Layout

```
Input (Host → Device):
  stories[S]            : Story metadata (priority, type, violence, reputation)
  story_targets[S][24]  : Boolean mask — which categories each story affects
  attitude[24]          : Current opinion values (read-only snapshot)
  violence_threshold    : Precomputed tolerance value

Output (Device → Host):
  opinion_deltas[24]    : Net change per opinion category (reduction result)
```

### Parallel Strategy

1. Each thread computes the impact of one story on one opinion category.
2. If `story_targets[s][c]` is false, the thread writes 0.
3. Otherwise, apply the impact formula: `base_shift × priority_modifier × reputation_modifier`, checking violence threshold for backlash.
4. A **parallel reduction** across stories accumulates deltas per category.
5. Host applies `opinion_deltas` to the global `attitude[]` array, clamping to [0, 100].

### Synchronization

- Shared memory reduction within each block for per-category delta accumulation.
- Final reduction across blocks via atomic adds on `opinion_deltas[24]`.

---

## Kernel 3: Congressional Vote Resolution

### Context

Monthly processing has Congress vote on bills. Each of 100 senators and 435 representatives independently decides based on their alignment, public opinion influence, and a random factor.

**Reference**: `analysis/MECHANICS/politics.md` — Congress section

### Thread Mapping

- **Grid**: 1D grid of 535 threads (100 senators + 435 representatives)
- **Block size**: 128 threads → 5 blocks
- **Per-thread work**: One politician's vote on one bill

### Data Layout

```
Input (Host → Device):
  politician_alignment[535] : Current alignment values (-2 to +2)
  attitude[24]              : Public opinion snapshot
  law_current[24]           : Current law alignment values
  bill_target_law           : Which law the bill addresses
  rng_seeds[535]            : Per-politician RNG seed

Output (Device → Host):
  votes[535]                : Per-politician vote (yea/nay)
  — or —
  vote_counts[2]            : Aggregated yea/nay totals (via atomic reduction)
```

### Parallel Strategy

1. Each thread reads its politician's alignment and the relevant public opinion.
2. Moderates are influenced by opinion; extremists vote on alignment alone.
3. A random factor adds variance to moderate decisions.
4. Each thread writes its vote; a final atomic reduction sums yea/nay totals.

Multiple bills per month can be batched into a 2D grid: `politicians × bills`.

---

## Kernel 4: Daily Activity Batch Resolution

### Context

Each game-day, all squad members' assigned activities are resolved through skill checks. Each character's outcome is independent given the current game state snapshot.

**Reference**: `analysis/MECHANICS/activities.md` — all activity types

### Thread Mapping

- **Grid**: 1D grid of N threads (one per active character with an assigned activity)
- **Block size**: 64 (character counts are typically small)
- **Per-thread work**: Resolve one character's daily activity

### Data Layout

```
Input (Host → Device):
  characters[N]         : Struct with skill values, activity type, location, juice
  activity_params[T]    : Per-activity-type parameters (difficulty, skill used, income formula)
  rng_seeds[N]          : Per-character RNG seed

Output (Device → Host):
  results[N]            : Outcome struct (success/fail, income generated, heat generated, skill XP)
```

### Parallel Strategy

1. Each thread reads its character's skills and activity type.
2. Performs the relevant skill check (difficulty lookup + random roll).
3. On success, computes income, opinion impact, or training progress.
4. Writes results struct; host merges outcomes into global state (funds, heat, skill arrays).

### Note on Shared State

Activities that modify shared state (fund balance, location heat) write to per-character output buffers. The host sequentially merges these to avoid race conditions on global state. This keeps the kernel simple and deterministic.

---

## Kernel 5: Site Map Restriction Propagation

### Context

After procedural map generation, an iterative flood-fill propagates "unrestricted" status across the 70×23×10 tile grid until convergence.

**Reference**: `analysis/ENGINE/site_system.md` — Restriction Propagation

### Thread Mapping

- **Grid**: 3D grid matching the tile grid: 70 × 23 × 10 = 16,100 threads
- **Block size**: 8 × 8 × 2 = 128 threads per block
- **Per-thread work**: Check neighbors and update restriction flag for one tile

### Data Layout

```
Input/Output (Device — ping-pong buffers):
  tilemap_A[70][23][10] : Current restriction state (bit flags)
  tilemap_B[70][23][10] : Next iteration state
  changed_flag          : Global atomic flag — did any tile change this iteration?

Host-side control:
  Loop: launch kernel → check changed_flag → swap buffers → repeat until converged
```

### Parallel Strategy

This uses a **Jacobi-style iterative update**:

1. Each thread reads its tile's restriction state from buffer A.
2. Checks all 6 neighbors (±x, ±y, ±z) in buffer A.
3. If any neighbor is unrestricted, marks this tile as unrestricted in buffer B.
4. If the tile's state changed, atomically sets `changed_flag = 1`.
5. Host checks `changed_flag`. If set, swaps A↔B and relaunches. If not, propagation is complete.

Boundary tiles (edges of the grid) skip out-of-bounds neighbor checks.

### Convergence

Typical convergence requires 5–15 iterations for LCS site maps. Each iteration is a full kernel launch with a grid-wide synchronization point (host-side swap check).

---

## Kernel Sizing Summary

| Kernel | Threads | Blocks (est.) | Launches per Invocation |
|--------|---------|---------------|------------------------|
| Election simulation | 1,000 | 4 | 1 |
| Opinion batch update | ~500 | 2–4 | 1 |
| Congressional votes | 535 | 5 | 1 per bill (batchable) |
| Activity resolution | 10–200 | 1–4 | 1 |
| Map flood fill | 16,100 | 126 | 5–15 (iterative) |

Thread counts are modest by GPU standards (modern GPUs handle millions). This means:

- **Occupancy will be low** for most kernels. The GPU will not be fully utilized.
- **The primary benefit is latency hiding** and freeing the CPU for other work (rendering, I/O) during GPU computation.
- **Batching multiple independent computations** (e.g., all monthly bills in one launch) improves utilization.

For maximum benefit, kernels should be launched asynchronously so the CPU can continue processing non-GPU work while the GPU computes.
