# Build System and Platform Integration

## Overview

The existing build system uses **GNU Autotools** (`configure.ac`, `Makefile.am`) with a C++11 compiler requirement. HIP integration must be added as an optional feature that does not affect the default build when the HIP toolchain is absent.

---

## Autotools Changes

### `configure.ac` Additions

Add a `--enable-hip` option with automatic detection:

```
# HIP GPU acceleration (optional)
AC_ARG_ENABLE([hip],
  [AS_HELP_STRING([--enable-hip], [Enable HIP GPU acceleration (requires hipcc)])],
  [enable_hip=$enableval],
  [enable_hip=auto])

if test "x$enable_hip" != "xno"; then
  AC_PATH_PROG([HIPCC], [hipcc])
  if test -n "$HIPCC"; then
    AC_DEFINE([LCS_ENABLE_HIP], [1], [Enable HIP GPU acceleration])
    have_hip=yes
  else
    if test "x$enable_hip" = "xyes"; then
      AC_MSG_ERROR([--enable-hip specified but hipcc not found])
    fi
    have_hip=no
  fi
else
  have_hip=no
fi
AM_CONDITIONAL([HAVE_HIP], [test "x$have_hip" = "xyes"])
```

This provides three modes:

| Option | Behavior |
|--------|----------|
| `--enable-hip` | Require HIP; fail if `hipcc` not found |
| `--disable-hip` | Never use HIP, even if available |
| *(default)* | Auto-detect `hipcc`; use HIP if found |

### `Makefile.am` Additions

```makefile
if HAVE_HIP
  HIPCC = @HIPCC@
  HIP_CXXFLAGS = -std=c++11 $(shell hipconfig --cpp_config)
  HIP_SOURCES = \
    src/gpu/gpu_context.hip.cpp \
    src/gpu/gpu_elections.hip.cpp \
    src/gpu/gpu_opinion.hip.cpp \
    src/gpu/gpu_politics.hip.cpp \
    src/gpu/gpu_activities.hip.cpp \
    src/gpu/gpu_mapgen.hip.cpp

  # Custom rule for .hip.cpp files
  .hip.cpp.o:
  	$(HIPCC) $(HIP_CXXFLAGS) -c -o $@ $<

  crimesquad_SOURCES += $(HIP_SOURCES)
  crimesquad_LDFLAGS += $(shell hipconfig --ldflags)
else
  crimesquad_SOURCES += src/gpu/gpu_stub.cpp
endif
```

When HIP is disabled, only `gpu_stub.cpp` is compiled, providing empty implementations of all `gpu_run_*()` functions.

---

## Platform Support Matrix

| Platform | HIP Backend | Compiler | Status |
|----------|-------------|----------|--------|
| Linux + AMD GPU | ROCm | hipcc (Clang-based) | Primary target |
| Linux + NVIDIA GPU | CUDA (via HIP) | hipcc → nvcc | Supported |
| Windows + AMD GPU | ROCm for Windows | hipcc | Future consideration |
| Windows + NVIDIA GPU | CUDA (via HIP) | hipcc → nvcc | Future consideration |
| macOS | — | — | Not supported (no ROCm/CUDA) |
| Any platform, no GPU | — | g++/clang++ | CPU fallback (always works) |

### Linux (Primary Target)

- **ROCm installation**: Required for AMD GPUs. Available via package managers on Ubuntu, Fedora, RHEL.
- **CUDA installation**: Required for NVIDIA GPUs when using HIP's CUDA backend. HIP translates HIP API calls to CUDA at compile time.
- **hipcc**: The HIP compiler driver. Detects the backend (ROCm or CUDA) automatically based on the `HIP_PLATFORM` environment variable.

### Windows

Windows support is deferred to a future phase. The existing Windows build uses PDCurses and MSVC/MinGW. HIP for Windows is available but less mature than Linux. The CPU fallback ensures Windows builds are unaffected.

### macOS

No HIP support exists for macOS. The CPU fallback covers this platform entirely.

---

## Dependencies

### Required (when HIP is enabled)

| Dependency | Version | Purpose |
|------------|---------|---------|
| ROCm or CUDA toolkit | ROCm 5.0+ or CUDA 11.0+ | GPU runtime and driver |
| hipcc | Matching ROCm/CUDA version | HIP compiler driver |
| HIP runtime libraries | Bundled with ROCm/CUDA | Device management, memory, kernel launch |

### Not Required

- No additional third-party GPU libraries (no hipBLAS, hipCUB, etc.) — all kernels use raw HIP API.
- No changes to existing dependencies (ncurses, SDL2, etc.).

---

## Compile-Time Configuration

### Preprocessor Defines

| Define | Set By | Purpose |
|--------|--------|---------|
| `LCS_ENABLE_HIP` | `configure` | Master switch for HIP code inclusion |
| `LCS_GPU_AVAILABLE` | `gpu_common.h` | Derived from `LCS_ENABLE_HIP`; used in game code |

### Build Configurations

| Configuration | Command | Result |
|---------------|---------|--------|
| CPU-only (default) | `./configure && make` | No GPU code compiled; stubs used |
| CPU-only (explicit) | `./configure --disable-hip && make` | Same as above |
| GPU-enabled (auto) | `./configure && make` *(with hipcc in PATH)* | HIP detected and enabled |
| GPU-enabled (explicit) | `./configure --enable-hip && make` | HIP required; error if not found |

---

## Runtime Configuration

A new `init.txt` option controls GPU usage at runtime:

```
# GPU acceleration (requires HIP-enabled build)
# gpu=On    — Use GPU when available
# gpu=Off   — Force CPU-only execution
#gpu=On
```

This allows users to disable GPU acceleration without recompiling, useful for debugging or when GPU drivers cause issues.

---

## CI/CD Considerations

### Existing CI

The current build should remain unaffected. CI environments without HIP toolchains will produce CPU-only builds automatically via the auto-detection mechanism.

### Future GPU CI

When GPU-enabled CI is available:

1. Add a build matrix entry with ROCm or CUDA installed.
2. Run the determinism test suite (CPU vs GPU output comparison).
3. Run performance benchmarks to track kernel timing regressions.

GPU CI is not required for Phase 0 or Phase 1 — development can use local GPU hardware for validation.

---

## File Organization Summary

```
configure.ac               # Add --enable-hip detection
Makefile.am                 # Add HIP sources, custom compile rule, stub fallback
src/gpu/
├── gpu_common.h            # Preprocessor guards, macros, shared RNG
├── gpu_context.h           # GPU context public interface
├── gpu_context.hip.cpp     # HIP device init/shutdown (compiled by hipcc)
├── gpu_elections.hip.cpp   # Election kernel
├── gpu_opinion.hip.cpp     # Opinion kernel
├── gpu_politics.hip.cpp    # Congressional vote kernel
├── gpu_activities.hip.cpp  # Activity resolution kernel
├── gpu_mapgen.hip.cpp      # Flood-fill kernel
└── gpu_stub.cpp            # CPU fallback stubs (compiled by g++/clang++)
init.txt                    # Add gpu=On/Off runtime option
```
