# HIP Integration Architecture

## Design Principles

1. **Optional acceleration** — HIP is a compile-time option. When the HIP toolchain is not available, the build falls back to CPU-only code paths with identical behavior.
2. **Minimal invasion** — The existing engine code is modified as little as possible. GPU kernels wrap existing logic rather than replacing it.
3. **Host-driven execution** — The CPU remains the orchestrator. GPU kernels are launched for batch computations and results are copied back to the existing global state arrays.
4. **Deterministic behavior** — GPU random number generation must be seeded to produce the same results as the CPU path for a given game seed, preserving save compatibility.

---

## Proposed Source Layout

```
src/
├── gpu/                        # All HIP-specific code lives here
│   ├── gpu_common.h            # Device/host macros, error checking, RNG state
│   ├── gpu_context.h/.cpp      # HIP device initialization and shutdown
│   ├── gpu_elections.hip.cpp   # Election simulation kernel
│   ├── gpu_opinion.hip.cpp     # Public opinion batch processing kernel
│   ├── gpu_politics.hip.cpp    # Congressional/court vote kernels
│   ├── gpu_activities.hip.cpp  # Daily activity batch resolution kernel
│   ├── gpu_mapgen.hip.cpp      # Site map flood-fill kernel
│   └── gpu_stub.cpp            # CPU fallback stubs (used when HIP is disabled)
```

Files with `.hip.cpp` extension are compiled by `hipcc`. The `gpu_stub.cpp` provides identical function signatures with CPU-only implementations, selected at build time via preprocessor guards.

---

## Host/Device Boundary

### Data Flow Pattern

All GPU-accelerated subsystems follow a uniform data flow:

```
  CPU (Host)                          GPU (Device)
  ──────────                          ────────────
  1. Prepare input buffers      →     
     (copy from global state)         
                                      2. Execute kernel
                                         (parallel computation)
  3. Read output buffers        ←     
     (merge back to global state)     
```

### Memory Management

| Strategy | Description |
|----------|-------------|
| **Pinned host memory** | Use `hipHostMalloc()` for input/output buffers to enable fast DMA transfers |
| **Device-local arrays** | Kernel working memory allocated with `hipMalloc()`, freed after kernel completes |
| **Reusable buffers** | Long-lived buffers for recurring computations (elections, opinion updates) allocated once in `gpu_context` and reused across invocations |

### Typical Buffer Layout — Election Kernel Example

```
Host → Device:
  - voter_mood[1000]        : float array of per-voter opinion snapshots
  - candidate_alignment[N]  : int array of candidate political alignments
  - rng_seeds[1000]         : uint64 per-voter RNG seed derived from game seed

Device → Host:
  - vote_results[N]         : int array of vote tallies per candidate
```

---

## GPU Context Lifecycle

```
main()
  └─→ gpu_context_init()           // Detect HIP device, allocate reusable buffers
        │
        ├─→ [game loop runs]       // Kernels launched as needed
        │     ├─→ gpu_run_election()
        │     ├─→ gpu_update_opinion()
        │     └─→ gpu_process_votes()
        │
        └─→ gpu_context_shutdown()  // Free buffers, release device
```

`gpu_context_init()` is called once at startup. If no HIP device is found, the context sets an internal `gpu_available` flag to `false`, and all subsequent `gpu_run_*()` calls silently delegate to CPU fallbacks.

---

## Conditional Compilation

```cpp
// gpu_common.h

#ifdef LCS_ENABLE_HIP
  #include <hip/hip_runtime.h>
  #define LCS_GPU_AVAILABLE 1
#else
  #define LCS_GPU_AVAILABLE 0
#endif
```

Call sites use a runtime check combined with compile-time guards:

```cpp
// In monthly political processing
#if LCS_GPU_AVAILABLE
  if (gpu_context::available()) {
    gpu_run_election(senate_races, voter_mood, rng_seed);
  } else {
    cpu_run_election(senate_races, voter_mood, rng_seed);
  }
#else
  cpu_run_election(senate_races, voter_mood, rng_seed);
#endif
```

---

## RNG Strategy

The existing engine uses `rand()` seeded once at startup. To maintain determinism on the GPU:

1. Each kernel thread receives a **per-thread seed** derived from the global game seed plus a thread-unique offset.
2. The GPU uses a simple, reproducible RNG (e.g., xorshift64) rather than `rand()`.
3. The CPU fallback uses the same xorshift64 implementation to guarantee identical results regardless of execution path.

This ensures that a given game seed produces the same political outcomes whether running on CPU or GPU.

---

## Error Handling

HIP API calls are wrapped in a checking macro:

```cpp
#define HIP_CHECK(call)                                      \
  do {                                                        \
    hipError_t err = (call);                                  \
    if (err != hipSuccess) {                                  \
      gamelog.log("HIP error: " + std::string(hipGetErrorString(err))); \
      gpu_context::mark_unavailable();                        \
      /* Fall back to CPU path */                             \
    }                                                         \
  } while (0)
```

On any GPU error, the context marks the device unavailable and all subsequent calls route to CPU fallbacks. This provides graceful degradation without crashing the game.

---

## Integration Points with Existing Code

| Existing File | Change | Purpose |
|---------------|--------|---------|
| `src/game.cpp` | Add `gpu_context_init()` / `gpu_context_shutdown()` calls | Lifecycle management |
| `src/monthly/monthly.cpp` | Add GPU dispatch for election and vote processing | Political simulation acceleration |
| `src/daily/daily.cpp` | Add GPU dispatch for batch activity resolution | Activity processing acceleration |
| `src/news/news.cpp` | Add GPU dispatch for opinion batch updates | Opinion processing acceleration |
| `src/sitemode/sitemap.cpp` | Add GPU dispatch for flood-fill pass | Map generation acceleration |
| `configure.ac` / `Makefile.am` | Add HIP detection and conditional compilation rules | Build system integration |

All changes to existing files are guarded by `#if LCS_GPU_AVAILABLE` to ensure zero impact when HIP is disabled.
