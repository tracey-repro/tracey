# Demo: HPC / Slurm Workflow

Autocap of a 3-stage Python pipeline where each stage runs on a compute node via `sbatch`. This demo shows how Tracey captures work that never runs on the login node, and how the packaged container automatically translates Slurm commands for portability.

**Pipeline:** `seed.txt → stage1.txt → stage2.txt → final.txt`

Each stage is a Python script submitted as a separate Slurm batch job.


## How Tracey captures Slurm jobs

When Tracey's shell hooks are installed, an environment variable (`BASH_ENV`) points to a hook script that is automatically sourced by every non-interactive bash session, including batch scripts on compute nodes.

On the compute node, the hook:
1. Re-executes the batch script under `strace -f`
2. Writes a trace log to `.tracey/logs/trace/` on the shared filesystem

The autocap daemon on the login node polls that directory, parses completed logs, and merges runtime-traced I/O into the node record. The result is a capture that knows exactly which files were read and written on the compute node, at confidence 1.0, because it came from syscall-level tracing.

If `strace` is unavailable on the compute node, Tracey falls back to file-diff detection on the shared filesystem.


## Walkthrough

> Tracey must be installed to run this demo. See [Get Access](../../README.md#get-access).  
> This demo requires Slurm access. Adjust `#SBATCH` account/partition lines in the `batch/` scripts for your site.

```bash
tracey init .

tracey autocap add s1_preprocess.py
tracey autocap add s2_compute.py
tracey autocap add s3_summarize.py --final

tracey autocap hook install --shell all   # one-time setup
tracey autocap start

# Submit the three stages as separate jobs
sbatch batch/step01.sbatch
sbatch batch/step02.sbatch
sbatch batch/step03.sbatch

# Wait for jobs to complete
squeue -u "${USER}"

tracey autocap stop --all
```

After the jobs finish, the node runs are captured:

```bash
tracey node list
# Node               Run  Mode    Command
# s3_summarize         1  auto    python s3_summarize.py
# s2_compute           1  auto    python s2_compute.py
# s1_preprocess        1  auto    python s1_preprocess.py
```

```bash
tracey workflow
tracey pack
apptainer run tracey.sif
```


## Container portability: Slurm command translation

When packaging, Tracey automatically rewrites Slurm-specific commands so the container runs without a Slurm installation:

| In the workflow | In the container |
|---|---|
| `sbatch script.sh` | `bash script.sh` |
| `module load python` | *(dropped)* |
| `squeue`, `scancel`, `sinfo` | *(dropped)* |

The batch scripts are written to handle running outside Slurm: `SLURM_SUBMIT_DIR` has a fallback, and `module load` lines are optional. Running `apptainer run tracey.sif` executes the pipeline directly, no Slurm required.
