# Tracey

**Transparent Recording & Automated Capture Engine for reproducibilitY**

Tracey is a CLI for recording, inspecting, replaying, and packaging reproducible computational workflows, without having to modify any of your existing scripts.


## The Problem

Computational science relies on pipelines: sequences of scripts that preprocess data, train models, evaluate results, and generate reports. These pipelines are hard to reproduce. Scripts change between runs, intermediate files get overwritten, and the exact sequence of steps that produced a result is rarely recorded. Sharing or transferring a workflow to a collaborator or a new cluster is even harder.

Tracey solves this by sitting quietly in the background and recording what you run based on rules you define, building a dependency graph from those records, and packaging the whole workflow into a portable container, all without touching your code.

## How It Works

Three ideas power Tracey:

**1. Capture without instrumentation.** You run your scripts exactly as you always have. Tracey watches in the background based on rules you define and records each execution: what ran, what files it read, what files it wrote, system level trace, when, and by whom. No decorators, no wrappers, no changes to your code.

**2. Derive the DAG from what actually ran.** Once steps are captured, Tracey builds a dependency graph by matching outputs of one step to inputs of the next. The graph reflects reality, the runs you actually did, not a hand-written specification that may have drifted.

**3. Pack to a portable container.** From the derived workflow, Tracey generates an Apptainer/Singularity container definition, stages all scripts and files, and builds a self-contained image. Anyone with the container can reproduce the results on any machine, no environment setup required.


## Key Capabilities

| Capability | What it does |
|---|---|
| `autocap` | Background capture: register rules for scripts you want tracked, start the daemon, then run your pipeline normally |
| `icap` | Interactive capture: open a session, run exploratory commands, close, the full session is recorded as one node |
| Dynamic discovery | Point Tracey at a directory; it captures every Python script or executable launched there, including ones that didn't exist when capture started |
| HPC / Slurm support | Tracey captures scripts submitted via `sbatch` on compute nodes, using `BASH_ENV` hooks and trace logs on the shared filesystem |
| Multi-language support | Language-agnostic capture: Python, C, C++, Java, shell: any executable Tracey can watch, it can capture |
| DAG visualization | Generate a graph image of all captured nodes and their dependencies (`tracey graph`) |
| Replay | Restore in a dedicated directory any past run from snapshots and re-execute it exactly, even after the script has since changed |
| Workflow derivation | Derive a clean sub-DAG from all captured runs, with run pinning and subgraph selection |
| Container packaging | Bundle the workflow, scripts, inputs, and optionally baseline outputs into a portable Apptainer image (`tracey pack`) |
| Baseline verification | Bundle output snapshots inside the container; re-running the container compares fresh outputs against the baseline automatically |


## Example: Climate Analysis Pipeline

Consider an example where a researcher detects temperature anomalies from a 25-day climate record. The pipeline preprocesses raw sensor data, extracts features, trains a linear model on a Slurm cluster, evaluates it (twice, after a script change), and generates a final report.

You can use tracey to capture every step in the background, including the compute-node training job submitted via `sbatch`, an exploratory dead-end that was abandoned, and a mid-pipeline script edit. None of the pipeline scripts were modified.

**Main pipeline**: this was derived by tracey from captured runs:

![Main pipeline DAG](demos/01_climate_pipeline/figures/main_pipeline.png)

**Full experiment**: this was also derived by tracey from all captured nodes, including the abandoned exploration branch:

![Full experiment graph including abandoned branch](demos/01_climate_pipeline/figures/full_experiment.png)

See the full walkthrough and more details in [demos/01_climate_pipeline](demos/01_climate_pipeline/README.md).



## Demos

| Demo | What it shows |
|---|---|
| [Climate Analysis Pipeline](demos/01_climate_pipeline/README.md) | Full showcase: autocap, icap, Slurm, multi-run nodes, replay, subgraph workflow, DAG visualization, packaging |
| [Simple 3-Stage Pipeline](demos/02_simple_pipeline/README.md) | Entry-point walkthrough: autocap + icap + pack for a straightforward linear pipeline |
| [Polyglot Workflow (C, C++, Java, Shell)](demos/03_polyglot_c_cpp_java/README.md) | Single icap session capturing a 4-language pipeline; container packages without compilers installed |
| [HPC / Slurm Workflow](demos/04_hpc_slurm/README.md) | Autocap of a 3-stage Python pipeline submitted via `sbatch`; shows compute-node capture and Slurm-to-bash container translation |
| [Dynamic Discovery](demos/05_dynamic_discovery/README.md) | Capture scripts you haven't named in advance, including scripts created after capture starts |
| [Reproducible Container + Baseline Verification](demos/06_reproducible_container/README.md) | Pack baseline outputs into the container; re-running compares fresh results against the original automatically |

---

## Learn More

Visit our website for more information: [https://tracey.rcc.uchicago.edu/](https://tracey.rcc.uchicago.edu/)



## Get Access

Tracey is currently available to a select group of collaborators and early users. If you are interested in trying Tracey, please reach out:

**Email:** xxx@xxx.com

We will get back to you with access instructions.
