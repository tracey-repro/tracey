# Tracey Command Overview

A summary of the main Tracey commands. Run `tracey <command> --help` for full flag details.


## `tracey init`

Initialize a Tracey experiment in a directory. Creates the `.tracey/` metadata directory.

```bash
tracey init .
tracey init . -d "RNA-seq preprocessing pipeline"
```


## `tracey autocap`: background capture

### Add capture rules

```bash
tracey autocap add step1.py                           # single script
tracey autocap add "*.py"                             # glob
tracey autocap add train.py --final                   # mark as workflow final node
tracey autocap add train.py --inputs data/train.csv --outputs outputs/model.pkl

# dynamic discovery: capture any python/executable launched from a directory
tracey autocap add . --discover python --recursive
tracey autocap add tools/ --discover executables
```

Tracey infers inputs/outputs from Python scripts via AST analysis and from shell scripts via static inference. Runtime tracing supplements this at execution time.

### Manage the daemon

```bash
tracey autocap hook install --shell all    # one-time shell hook setup
tracey autocap start                       # start background capture daemon
tracey autocap stop --all                  # stop daemon
tracey autocap status                      # check if daemon is running
tracey autocap doctor                      # run diagnostics
```

### List and remove rules

```bash
tracey autocap list
tracey autocap list --long
tracey autocap remove preprocess
```


## `tracey icap`: interactive capture

Open an interactive shell session. Everything run inside is captured as a single named node. End the session with `Ctrl-]`.

```bash
tracey icap explore_step
tracey icap train_node --no-streams       # skip stdin/stdout capture
```


## `tracey node`: inspect runs

```bash
tracey node list                           # list all captured runs
tracey node list --long                    # with I/O detail
tracey node show preprocess 1             # full detail for one run
tracey node note preprocess 1 "baseline"  # annotate a run
tracey node delete preprocess --run 2     # delete one run
tracey node set-final report              # set the workflow terminal node
```


## `tracey workflow`: derive the DAG

```bash
tracey workflow                            # full DAG to configured final node
tracey workflow --to report               # explicit target node
tracey workflow --from preprocess --to report --path-mode shortest
tracey workflow --from preprocess --to report --path-mode all
tracey workflow --select evaluate=2       # pin a node to a specific run
tracey workflow --output .tracey/custom.yaml
```


## `tracey graph`: visualize

```bash
tracey graph                               # PNG from all captured nodes
tracey graph --source workflow             # PNG from workflow YAML
tracey graph --source workflow --format svg
tracey graph --output figures/pipeline.png
```


## `tracey replay`: re-execute a past run

Restores snapshotted scripts and inputs into an isolated directory and re-runs the recorded command. Works even if the script has since been edited.

```bash
tracey replay preprocess 1 --dry-run      # preview what would be restored
tracey replay preprocess 1
tracey replay preprocess 1 --dir /tmp/replay-dir
```

## `tracey pack`: build a container

```bash
tracey pack                                             # generate .def and build .sif
tracey pack --generate-def-only                        # write .def only, no build
tracey pack --execute build                            # build from existing .def
tracey pack --include-baseline-outputs                 # bundle output snapshots in image
tracey pack --include-metadata                         # bundle workflow metadata in image
tracey pack --config .tracey/workflow_eval2.yaml       # use a specific workflow
tracey pack --container-name my_pipeline.sif
```

When the container is run, it executes the packaged workflow in a writable temp directory and prints the output path:

```bash
apptainer run tracey.sif
```

If baseline outputs were bundled, the container automatically compares fresh outputs and reports matches/differences.
