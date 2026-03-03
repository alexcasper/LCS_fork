# Migration Strategy

## Phased Approach

The migration from CPU-only to HIP-accelerated code follows four phases. Each phase is independently shippable — the game remains fully functional after every phase, with GPU acceleration as a progressive enhancement.

---

## Phase 0: Foundation (Prerequisites)

**Goal**: Establish the GPU infrastructure without changing any game logic.

### Tasks

1. **Add `src/gpu/` directory** with the source layout described in [architecture.md](architecture.md).
2. **Implement `gpu_context`** — device detection, initialization, shutdown, and the `available()` runtime check.
3. **Implement `gpu_common.h`** — preprocessor guards (`LCS_ENABLE_HIP`), error-checking macros (`HIP_CHECK`), and the shared xorshift64 RNG implementation.
4. **Implement `gpu_stub.cpp`** — CPU fallback stubs for every `gpu_run_*()` function signature.
5. **Update `configure.ac` and `Makefile.am`** — add HIP toolchain detection and conditional compilation rules (see [build_integration.md](build_integration.md)).
6. **Add `gpu_context_init()` / `gpu_context_shutdown()`** calls to `src/game.cpp` main entry point, guarded by `#if LCS_GPU_AVAILABLE`.
7. **Verify**: Game compiles and runs identically with `LCS_ENABLE_HIP` both defined and undefined. No behavioral changes.

### Acceptance Criteria

- `./configure && make` succeeds with no HIP toolchain installed (CPU-only build).
- `./configure --enable-hip && make` succeeds when HIP toolchain is available.
- `gpu_context::available()` correctly reports device presence.
- All existing tests pass.

---

## Phase 1: Election and Political Kernels

**Goal**: GPU-accelerate the highest-value, lowest-risk subsystems — election simulation and congressional votes.

### Tasks

1. **Extract CPU election logic** into a standalone function `cpu_run_election()` with a clean interface (input arrays → output tallies). This refactor is needed regardless of GPU work, as the current logic is embedded in the monthly processing loop.
2. **Implement `gpu_elections.hip.cpp`** with the election simulation kernel from [data_parallelism.md](data_parallelism.md).
3. **Implement `gpu_politics.hip.cpp`** with the congressional vote kernel.
4. **Add dispatch logic** in `src/monthly/monthly.cpp` — call GPU path when available, CPU path otherwise.
5. **Implement deterministic RNG** — verify that CPU and GPU paths produce identical election outcomes for the same game seed.
6. **Test**: Run elections with both paths and compare results byte-for-byte.

### Acceptance Criteria

- Election results are identical between CPU and GPU paths for any given seed.
- Monthly political processing completes successfully on both paths.
- Save files produced by GPU-path games are loadable by CPU-only builds.
- No regressions in game behavior.

---

## Phase 2: Opinion Processing

**Goal**: GPU-accelerate the public opinion batch update pipeline.

### Tasks

1. **Extract CPU opinion logic** into `cpu_update_opinion()` with clean input/output boundaries.
2. **Implement `gpu_opinion.hip.cpp`** with the opinion batch update kernel.
3. **Add dispatch logic** in `src/news/news.cpp` or the relevant opinion processing function.
4. **Handle the reduction step** — verify that parallel accumulation of opinion deltas produces the same result as sequential application.
5. **Test**: Compare `attitude[]` array state after processing identical story sets on both paths.

### Acceptance Criteria

- Opinion shifts match between CPU and GPU paths for identical story inputs.
- News cycle processing completes successfully on both paths.
- Game saves remain compatible.

---

## Phase 3: Activity Resolution and Map Generation

**Goal**: GPU-accelerate the remaining medium-priority subsystems.

### Tasks

1. **Implement `gpu_activities.hip.cpp`** with the daily activity batch resolution kernel.
2. **Implement `gpu_mapgen.hip.cpp`** with the Jacobi-style flood-fill kernel for restriction propagation.
3. **Add dispatch logic** in `src/daily/daily.cpp` and `src/sitemode/sitemap.cpp`.
4. **Benchmark**: Measure actual wall-clock improvement. Activity resolution may not show meaningful speedup due to small character counts — if so, keep the CPU path as default and make GPU opt-in.
5. **Test**: Verify activity outcomes and map restriction states match between paths.

### Acceptance Criteria

- Activity results are identical between CPU and GPU paths.
- Generated site maps have identical restriction states between paths.
- No regressions.

---

## Phase Summary

| Phase | Scope | Risk | Effort | Value |
|-------|-------|------|--------|-------|
| 0 — Foundation | Build system + stubs | Low | Medium | Enables all future phases |
| 1 — Elections | Election + congress kernels | Low | Medium | Accelerates highest-volume computation |
| 2 — Opinion | Opinion batch kernel | Low | Low | Accelerates daily news processing |
| 3 — Activities + Maps | Activity + flood-fill kernels | Medium | Medium | Completes coverage of parallelizable systems |

---

## Rollback Strategy

Every phase maintains the CPU fallback path. If GPU acceleration introduces issues:

1. **Runtime fallback**: Set `gpu_context::mark_unavailable()` to disable GPU at runtime without recompilation.
2. **Compile-time fallback**: Build without `--enable-hip` to exclude all GPU code.
3. **Per-kernel fallback**: Each `gpu_run_*()` function can be individually redirected to its CPU stub by setting per-subsystem flags.

No game logic is removed or replaced — GPU paths are strictly additive.

---

## Testing Strategy

### Unit Tests

Each kernel gets a dedicated test that:

1. Seeds the RNG with a known value.
2. Runs the CPU path and records outputs.
3. Runs the GPU path with the same seed and inputs.
4. Asserts byte-for-byte equality of outputs.

### Integration Tests

- Full game month simulation comparing save file state between CPU and GPU paths.
- Election outcome comparison across 100 random seeds.
- Opinion drift comparison across multi-month simulations.

### Performance Tests

- Wall-clock benchmarks for each kernel vs. CPU baseline.
- GPU utilization metrics (occupancy, memory bandwidth) to identify optimization opportunities.
- Latency measurements for host↔device transfers to verify transfer overhead does not negate compute gains.
