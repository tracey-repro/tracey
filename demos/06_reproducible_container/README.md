# Demo: Reproducible Container + Baseline Verification

When sharing a packaged workflow, you may want collaborators to be able to verify that their results match yours. Tracey supports this with two pack flags:

- `--include-baseline-outputs`: bundles snapshotted outputs from your captured run into the container image
- `--include-metadata`: bundles workflow and node-run metadata inside the container

When the container is run, it automatically compares fresh outputs against the bundled baseline and reports whether they match.

**Pipeline:** `raw.txt → numbers.txt → summary.json → final_report.txt`

## Walkthrough

> Tracey must be installed to run this demo. See [Get Access](../../README.md#get-access).

### Step 1: Capture the pipeline

```bash
tracey init .

tracey autocap add s1_generate.py
tracey autocap add s2_summarize.py
tracey autocap add s3_report.py --final

tracey autocap start
python3 s1_generate.py
python3 s2_summarize.py
python3 s3_report.py
tracey autocap stop

tracey workflow
tracey graph --source workflow
```

---

### Step 2: Pack with baseline outputs and metadata

```bash
tracey pack --include-baseline-outputs --include-metadata
# INFO:    Build complete: tracey.sif
```

---

### Step 3: Inspect what was packaged

The container stores scripts, inputs, baseline outputs, and metadata under `/workflow` (read-only):

```bash
apptainer exec tracey.sif ls /workflow
# _tracey_baseline_outputs/   _tracey_pack/
# numbers.txt   raw.txt   s1_generate.py   s2_summarize.py   s3_report.py   summary.json

apptainer exec tracey.sif ls /workflow/_tracey_baseline_outputs
# final_report.txt   numbers.txt   summary.json
```

---

### Step 4: Run and compare against the baseline

```bash
apptainer run tracey.sif
```

```
tracey pack: comparing outputs to packaged baseline outputs
OK      : numbers.txt
OK      : summary.json
OK      : final_report.txt
tracey pack: output diff summary: 3/3 matched
Output files are in: /tmp/tmp.xxxxx
```

If a result differs from the baseline, the container reports `DIFF` for that file. If a file is missing entirely, it reports `MISSING`. This gives collaborators immediate feedback on whether their environment reproduces your results.

---

### Step 5: Extract metadata from the container

The `--include-metadata` flag bundles workflow YAML and per-node run files inside the container. A collaborator can extract and inspect them without having access to the original experiment:

```bash
mkdir -p extracted
apptainer exec --bind "$PWD/extracted:/out" tracey.sif bash -lc '
  cp -a /workflow/. /out/
'
# Now on the host:
ls extracted/_tracey_pack/
# workflow.yaml   pack.yaml   nodes/

ls extracted/_tracey_pack/nodes/
# s1_generate_run_1.yaml   s2_summarize_run_1.yaml   s3_report_run_1.yaml
```

The extracted node YAML files contain the full provenance record for each captured run: commands, inputs, outputs, environment hints, and timestamps.



## What this enables

A container built with `--include-baseline-outputs --include-metadata` is a self-describing artifact: it carries the workflow that was run, the provenance of every step, and the expected outputs. Anyone who runs it immediately knows whether their environment reproduces your results, no side-channel communication required.
