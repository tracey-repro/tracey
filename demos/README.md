# Tracey Demos

Each demo is a self-contained walkthrough showing Tracey in a different scenario. They are ordered from comprehensive to focused.

| # | Demo | Highlights |
|---|---|---|
| 1 | [Climate Analysis Pipeline](01_climate_pipeline/README.md) | Full showcase across all major features: autocap, icap, Slurm/HPC, multi-run nodes with script evolution, replay, subgraph workflow derivation, DAG visualization, and container packaging |
| 2 | [Simple 3-Stage Pipeline](02_simple_pipeline/README.md) | Entry-point walkthrough: a linear pipeline captured with autocap and icap, with run inspection, replay, and packaging |
| 3 | [Polyglot Workflow (C, C++, Java, Shell)](03_polyglot_c_cpp_java/README.md) | A 4-language pipeline captured in a single icap session; the resulting container packages the workflow without requiring compilers to be installed |
| 4 | [HPC / Slurm Workflow](04_hpc_slurm/README.md) | Autocap of a 3-stage Python pipeline submitted via `sbatch`; shows how Tracey captures compute-node execution and translates Slurm commands for container portability |
| 5 | [Dynamic Discovery](05_dynamic_discovery/README.md) | Point Tracey at a directory and capture every script launched there — including scripts that are created after capture starts |
| 6 | [Reproducible Container + Baseline Verification](06_reproducible_container/README.md) | Bundle output snapshots inside the container; re-running it compares fresh results against the original baseline and reports matches and differences |

---

> Tracey must be installed to run these demos. See [Get Access](../README.md#get-access) to request access.
