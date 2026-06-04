# Demo: Simple 3-Stage Pipeline

An entry-point walkthrough showing the core Tracey loop: capture a pipeline with autocap and icap, inspect the runs, replay a past run, visualize the DAG, and package into a container.

**Pipeline:** `seed.txt → stage1.txt → stage2.txt → stage3_summary.txt → final_report.txt`


## Pipeline steps

| Script | Stage | Capture mode |
|--------|-------|-------------|
| `s1_collect.py` | Stage 1 | autocap |
| `s2_enrich.py` | Stage 2 | autocap |
| `s3_prepare_report.py` + `s3_publish.py` | Stage 3 | icap (single session) |

Stage 3 runs two scripts captured together as one logical node.


## Walkthrough

> Tracey must be installed to run this demo. See [Get Access](../../README.md#get-access).

### Part A: Autocap for Stage 1 and Stage 2

```bash
tracey init .

tracey autocap add s1_collect.py
tracey autocap add s2_enrich.py

tracey autocap hook install --shell all   # one-time setup
tracey autocap start

python s1_collect.py
# s1_collect: wrote 4 lines to stage1.txt

python s2_enrich.py
# s2_enrich: wrote 4 lines to stage2.txt

python s2_enrich.py   # run again to show multiple runs
# s2_enrich: wrote 4 lines to stage2.txt

tracey autocap stop --all
```

---

### Part B: Icap for Stage 3

Stage 3 runs two scripts as one logical node — both are captured under the same session.

```bash
tracey icap stage3
```

Inside the session:

```
python s3_prepare_report.py
# s3_prepare_report: wrote stage3_summary.txt

python s3_publish.py
# s3_publish: wrote final_report.txt

# press Ctrl-] to end session
# Saved node run: .tracey/nodes/stage3_run_1.yaml
```

---

### Part C: Inspect runs, add notes, replay

```bash
tracey node note s2_enrich 2 "reran for testing"

tracey node list
# Node               Run  Mode         Note
# stage3               1  interactive  -
# s2_enrich            2  auto         reran for testing
# s2_enrich            1  auto         -
# s1_collect           1  auto         -

tracey node show s2_enrich 2
# Node: s2_enrich  |  Run: 2
# Note   : reran for testing
# inputs : stage1.txt
# outputs: stage2.txt
# snaps  : 1 script, 1 input, 1 output

# Replay run 1 of s2_enrich in isolation
tracey replay s2_enrich 1
# Replay dir : .tracey/replays/s2_enrich_run_1
# s2_enrich: wrote 4 lines to stage2.txt
# Replay finished with exit code 0
```

---

### Part D: DAG image, workflow, and container

```bash
# Visualize all captured runs as a DAG image
tracey graph
# Graph image written to: .tracey/graph.png
```

```bash
# Derive workflow YAML
tracey workflow --to stage3

# Build container
tracey pack
# INFO:    Build complete: tracey.sif
# s1_collect: wrote 4 lines to stage1.txt
# s2_enrich: wrote 4 lines to stage2.txt
# s3_prepare_report: wrote stage3_summary.txt
# s3_publish: wrote final_report.txt
# Output files are in: /tmp/tmp.xxxxx
```

The container runs the full pipeline on any machine with `apptainer run tracey.sif`.
