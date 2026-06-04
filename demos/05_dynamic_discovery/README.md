# Demo: Dynamic Discovery

Tracey can capture scripts without you naming them in advance. Point it at a directory and it captures every matching script or executable launched from there, including ones that are created after capture starts.

Two discovery modes are available:

| Mode | What it captures |
|------|-----------------|
| `--discover python` | Any `python script.py` invocation for scripts under the scoped directory |
| `--discover executables` | Any executable file (shell scripts, binaries) launched from the scoped directory |

Both modes support `--recursive` to include subdirectories.


## Example: Python script discovery

```bash
tracey init .

# Capture any Python script run from the py_scripts/ directory (recursively)
tracey autocap add py_scripts --discover python --recursive

tracey autocap list --long
# discover_python_py_scripts
#   dynamic: yes (python)
#   scope  : py_scripts
#   recur  : yes
```

Start capture and run an existing script:

```bash
tracey autocap start
python py_scripts/alpha.py

tracey node list
# Node               Run  Mode    Command
# alpha                1  auto    python py_scripts/alpha.py
```

Now create a new script at runtime — after capture is already running — and run it:

```bash
cat > py_scripts/beta.py <<'EOF'
from pathlib import Path
Path("beta.txt").write_text("beta\n")
EOF

python py_scripts/beta.py

tracey node list
# Node               Run  Mode    Command
# beta                 1  auto    python py_scripts/beta.py   <-- discovered at runtime
# alpha                1  auto    python py_scripts/alpha.py

tracey autocap stop --all
```

`beta.py` was never registered. Tracey captured it the moment it was launched.


## Example: Executable discovery

```bash
tracey init .

# Capture any executable launched from the tools/ directory (recursively)
tracey autocap add tools --discover executables --recursive

tracey autocap start
./tools/existing_exec.sh

tracey node list
# existing_exec        1  auto    bash ./tools/existing_exec.sh

# Add a new executable at runtime
cat > tools/runtime_exec.sh <<'EOF'
#!/usr/bin/env bash
echo runtime > runtime_exec.txt
EOF
chmod +x tools/runtime_exec.sh

./tools/runtime_exec.sh

tracey node list
# runtime_exec         1  auto    bash ./tools/runtime_exec.sh  <-- discovered at runtime
# existing_exec        1  auto    bash ./tools/existing_exec.sh

tracey autocap stop --all
```


## What makes this useful

In real pipelines, it is common to not know ahead of time exactly which scripts will run, utility scripts are added mid-experiment, helper tools are invoked by other scripts, or the pipeline evolves as the research progresses.

Dynamic discovery means you do not have to maintain a perfect list of capture rules. Register the scope once, and Tracey handles the rest.

Discovery mode only captures scripts that are actually launched, file creation alone does not trigger a capture. Non-matching files (for example, a shell script under a `--discover python` rule) are ignored.
