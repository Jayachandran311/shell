# Shell Scripting — Introduction

## What is a Shell?

A **shell** is a command-line interpreter that acts as the bridge between the user and the Linux kernel. It reads commands (typed or from a script file) and executes them.

A **shell script** is a plain text file containing a sequence of shell commands that are executed top to bottom.

---

## Types of Shells

| Shell | Full Name | Notes |
|-------|-----------|-------|
| `sh` | Bourne Shell | Original Unix shell (POSIX standard) |
| `bash` | Bourne Again Shell | Most common on Linux, default on Ubuntu |
| `zsh` | Z Shell | Default on macOS, feature-rich |
| `ksh` | Korn Shell | Enterprise scripting |
| `fish` | Friendly Interactive Shell | User-friendly, not POSIX |
| `dash` | Debian Almquist Shell | Minimal, fast, used for `/bin/sh` on Debian |

Check your current shell:
```bash
echo $SHELL        # shows default shell
echo $0            # shows current active shell
cat /etc/shells    # lists all installed shells
```

---

## The Shebang Line (`#!`)

The very first line of every script — tells the OS **which interpreter** to use.

```bash
#!/bin/bash              # use bash at fixed path
#!/bin/sh                # use POSIX sh (portable)
#!/usr/bin/env bash      # find bash via PATH (most portable)
#!/usr/bin/env python3   # works for Python too
```

> **Best practice:** Use `#!/usr/bin/env bash` for maximum portability across different systems.

---

## Your First Script

Create a file `hello.sh`:

```bash
#!/usr/bin/env bash
# My first shell script

echo "Hello, World!"
echo "Today is: $(date)"
echo "You are: $(whoami)"
```

---

## Running Scripts

```bash
# Step 1 — Make it executable (only once)
chmod +x hello.sh

# Step 2 — Run it
./hello.sh

# Alternatively run without chmod:
bash hello.sh
sh hello.sh

# Source — runs in current shell (shares variables)
source hello.sh
. hello.sh       # shorthand for source
```

> **Difference:** `./hello.sh` spawns a child process. `source hello.sh` runs in the current shell, so any variables set will persist after the script ends.

---

## Comments

```bash
#!/usr/bin/env bash

# This is a single-line comment

echo "Hello"  # Inline comment — everything after # is ignored

# Multi-line: just use multiple # lines
# Line 1
# Line 2
# Line 3
```

---

## Exit Codes

Every command and script returns an **exit code**:
- `0` — Success
- Non-zero (`1`, `2`, `127`, etc.) — Error

```bash
ls /etc          # runs fine → exit code 0
ls /fakedir      # fails     → exit code 2

echo $?          # print exit code of the LAST command
```

Custom exit codes in your script:
```bash
#!/usr/bin/env bash
echo "Starting..."
# ... do work ...
exit 0    # explicit success

# exit 1  → general error
# exit 2  → misuse of shell command
# exit 127 → command not found
```

---

## Debugging Scripts

```bash
# Run with full trace (print each command before executing)
bash -x hello.sh

# Check for syntax errors without running
bash -n hello.sh

# Inside the script:
set -x          # turn on trace
set +x          # turn off trace

# Recommended safety options (put at very top of every script):
set -e          # exit immediately on any error
set -u          # treat unset variables as errors
set -o pipefail # catch errors in pipelines
```

Combined shorthand:
```bash
#!/usr/bin/env bash
set -euo pipefail
```

---

## Script Structure Best Practices

```bash
#!/usr/bin/env bash
set -euo pipefail

# ─── Constants ─────────────────────────────
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"

# ─── Functions ─────────────────────────────
log() { echo "[$(date '+%H:%M:%S')] $*"; }
die() { echo "ERROR: $*" >&2; exit 1; }

# ─── Main logic ────────────────────────────
main() {
    log "Script started"
    # ... your code here ...
    log "Done"
}

main "$@"
```

---

## Key Takeaways

> - Always start with `#!/usr/bin/env bash`
> - Use `set -euo pipefail` to catch errors early
> - Use `chmod +x` before running with `./`
> - Use `bash -x` to debug, `bash -n` to check syntax
> - Exit `0` = success, non-zero = failure
