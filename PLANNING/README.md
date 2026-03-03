# GPU (HIP) Implementation Planning

## Purpose

This folder contains planning documents for introducing **AMD HIP** (Heterogeneous-computing Interface for Portability) GPU acceleration into Liberal Crime Squad. HIP was chosen because it compiles for both AMD (ROCm) and NVIDIA (CUDA) backends, providing the widest hardware reach from a single codebase.

## Motivation

The current engine is single-threaded C++11 with all computation on the CPU. Several subsystems process large batches of independent data — political simulations with 1000+ voters, encounter generation across dozens of locations, and monthly processing of hundreds of characters. These workloads are natural candidates for data-parallel GPU execution.

GPU acceleration goals:

1. **Faster batch simulations** — Elections, public opinion shifts, and monthly processing involve many independent calculations that map well to GPU kernels.
2. **Scalable map generation** — Procedural site generation with flood-fill and BSP algorithms can benefit from parallel tile processing.
3. **Future-proofing** — Establishing a GPU compute layer now enables more ambitious simulation features (larger populations, real-time opinion modeling, expanded political systems).

## Document Index

| Document | Description |
|----------|-------------|
| [candidate_systems.md](candidate_systems.md) | Analysis of which engine subsystems are suitable for GPU acceleration |
| [architecture.md](architecture.md) | HIP integration architecture and host/device boundary design |
| [data_parallelism.md](data_parallelism.md) | Kernel designs and data parallelism strategies for each candidate system |
| [migration_strategy.md](migration_strategy.md) | Phased migration plan from CPU-only to HIP-accelerated |
| [build_integration.md](build_integration.md) | Build system changes, platform support, and dependency management |

## Constraints

- HIP code must be **optional** — the game must still compile and run on CPU-only systems with no HIP toolchain installed.
- GPU acceleration should be **transparent** — game behavior and save compatibility must be preserved.
- All planning is based on the existing analysis in the `analysis/` folder.
