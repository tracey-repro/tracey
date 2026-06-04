# Demo: Polyglot Workflow (C, C++, Java, Shell)

A 4-language pipeline captured in a single interactive session. Shell scripts orchestrate compiled C, C++, and Java tools. The resulting container packages the full pipeline, even though compilers and runtimes are not installed inside the image.

**Pipeline:** `seed.txt → c_stage.txt → cpp_stage.txt → java_stage.txt → final_report.txt`

Each stage is driven by a shell script that calls a compiled tool:

| Stage | Shell script | Language | Tool |
|-------|-------------|----------|------|
| Stage 1 | `s1_c_stage.sh` | C | `c_stage` (compiled from `c_stage.c`) |
| Stage 2 | `s2_cpp_stage.sh` | C++ | `cpp_stage` (compiled from `cpp_stage.cpp`) |
| Stage 3 | `s3_java_stage.sh` | Java | `JavaStage.class` (shell fallback if unavailable) |
| Stage 4 | `s4_finalize.sh` | Shell | - |


## Walkthrough

> Tracey must be installed to run this demo. See [Get Access](../../README.md#get-access).

```bash
# Build compiled artifacts before capture
./build_artifacts.sh

tracey init .

# Open an icap session — all 4 stages are captured under one node
tracey icap polyglot_pipeline
```

Inside the session:

```
bash s1_c_stage.sh
bash s2_cpp_stage.sh
bash s3_java_stage.sh
bash s4_finalize.sh java_stage.txt final_report.txt

# press Ctrl-] to end session
```

Inspect and package:

```bash
tracey node list --long
# Node                 Run  Mode         Commands
# polyglot_pipeline      1  interactive  4 commands

tracey workflow
tracey pack
# INFO:    Creating SIF file...
# INFO:    Build complete: tracey.sif
# s1_c_stage complete
# s2_cpp_stage complete
# s3_java_stage complete
# s4_finalize complete
# Output files are in: /tmp/tmp.xxxxx
```


## What makes this notable

**Language agnosticism.** Tracey captures what ran, not what language it was written in. It records the shell commands, traces the files they read and wrote, and snapshots the scripts and compiled binaries. The icap session boundary defines the node; the language mix inside it is irrelevant.

**Container packaging without runtime dependencies.** Tracey analyzes the workflow commands and infers toolchain requirements. For a pipeline that invokes `gcc`, `g++`, or `javac`, it automatically emits the appropriate `%post` install commands in the container definition so the image is self-sufficient. The compiled binaries are staged alongside the scripts.

**One session, one node.** All four stages are captured as a single `polyglot_pipeline` node. When you derive the workflow and package it, the container runs them in the same order.
