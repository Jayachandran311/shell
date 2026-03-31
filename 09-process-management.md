# Shell Scripting — Process Management

## Running Commands in Background

```bash
# Run in background with &
long_running_command &
sleep 60 &

# Get PID of last background process
echo "Background PID: $!"

# Run in background and disown (continues even after terminal closes)
nohup ./script.sh > output.log 2>&1 &
disown $!
```

---

## Job Control

```bash
long_task &        # send to background
jobs               # list background jobs
jobs -l            # with PIDs

fg %1              # bring job 1 to foreground
fg %               # bring most recent job to foreground
bg %2              # send stopped job 2 to background

# Ctrl+Z — suspend foreground process (sends it to background as stopped)
# Ctrl+C — kill foreground process (SIGINT)
```

---

## `wait` — Synchronize Parallel Jobs

```bash
# Start multiple background jobs
for server in web01 web02 web03; do
    backup_server "$server" &
done

# Wait for ALL background jobs to finish
wait
echo "All backups complete"

# Wait for a specific PID
mv bigfile /archive &
MOVE_PID=$!
# ... do other things ...
wait $MOVE_PID && echo "Move complete" || echo "Move failed"
```

---

## Signals and `trap`

### Common Signals

| Signal | Number | Default Action | Meaning |
|--------|--------|---------------|---------|
| `SIGINT` | 2 | Terminate | Ctrl+C |
| `SIGTERM` | 15 | Terminate | `kill` default signal |
| `SIGHUP` | 1 | Terminate | Terminal closed |
| `SIGKILL` | 9 | Terminate | Cannot be caught |
| `EXIT` | — | — | Script exiting (pseudo-signal) |

### `trap` — Catch signals and run cleanup

```bash
#!/usr/bin/env bash

TMPFILE=$(mktemp)

cleanup() {
    echo "Cleaning up..."
    rm -f "$TMPFILE"
    echo "Done."
}

# Run cleanup on EXIT (normal completion, error, or Ctrl+C)
trap cleanup EXIT

# Or handle specific signals
trap cleanup EXIT INT TERM

# Ignore a signal
trap '' SIGINT    # ignore Ctrl+C

# Restore default behavior
trap - SIGINT     # restore default for SIGINT

# Example: lock file cleanup
LOCKFILE="/tmp/myapp.lock"
trap "rm -f $LOCKFILE; exit" EXIT INT TERM

echo $$ > "$LOCKFILE"
echo "Running with lock: $LOCKFILE"
# ... work ...
```

### Graceful shutdown

```bash
#!/usr/bin/env bash
RUNNING=true

shutdown() {
    echo "Received shutdown signal. Stopping..."
    RUNNING=false
}
trap shutdown SIGTERM SIGINT

echo "Service running (PID: $$)"
while $RUNNING; do
    # Main loop work
    echo "Doing work... $(date)"
    sleep 5
done

echo "Service stopped cleanly"
```

---

## Sending Signals (`kill`)

```bash
kill PID             # send SIGTERM (ask to terminate)
kill -9 PID          # send SIGKILL (force kill, cannot ignore)
kill -TERM PID       # same as kill PID
kill -HUP PID        # reload config (many daemons honour this)
kill -0 PID          # check if process exists (no signal sent)

killall nginx        # kill all processes named 'nginx'
pkill -f "python script.py"   # kill by process name/pattern
```

---

## Checking Process Status

```bash
# Check if a process is running
if kill -0 $PID 2>/dev/null; then
    echo "Process $PID is running"
fi

# Check by name
if pgrep nginx &>/dev/null; then
    echo "nginx is running"
fi

# Wait for a process to stop
while kill -0 $PID 2>/dev/null; do
    sleep 1
done
echo "Process stopped"
```

---

## Subshells

A subshell is a child process that inherits parent variables but changes don't propagate back.

```bash
x=10

(
    x=99           # change in subshell
    echo "Inside subshell: x=$x"
)

echo "Outside subshell: x=$x"   # still 10
```

Common subshell uses:
```bash
# Change directory temporarily
(cd /some/dir && ls -la)
echo "Still in original dir: $(pwd)"

# Process substitution
diff <(sort file1.txt) <(sort file2.txt)
```

---

## `exec` — Replace Current Shell

```bash
exec another_program   # replaces current shell entirely
exec > logfile.log     # redirect all stdout to file from here on
exec 2> errors.log     # redirect all stderr to file
exec >> append.log     # append mode

# Practical: use exec to set up logging for entire script
exec > >(tee -a "/var/log/myapp.log") 2>&1
echo "This goes to console AND log file"
```

---

## Parallel Processing

```bash
#!/usr/bin/env bash

process_item() {
    local item=$1
    echo "Processing: $item"
    sleep 2    # simulate work
    echo "Done: $item"
}

# Sequential — takes n*2 seconds
# items=("a" "b" "c" "d" "e")
# for item in "${items[@]}"; do process_item "$item"; done

# Parallel — takes ~2 seconds regardless of count
items=("a" "b" "c" "d" "e")
pids=()

for item in "${items[@]}"; do
    process_item "$item" &
    pids+=($!)
done

# Collect exit codes
errors=0
for pid in "${pids[@]}"; do
    wait "$pid" || (( errors++ ))
done

echo "Finished. Errors: $errors"
```

---

## Key Takeaways

> - Use `&` to background a process; `wait` to synchronize
> - Use `trap cleanup EXIT` to ensure cleanup runs on any exit path
> - `kill PID` sends SIGTERM (polite); `kill -9 PID` sends SIGKILL (force)
> - `pgrep nginx` / `pkill nginx` work by process name
> - Subshells `()` inherit variables but changes don't affect parent
> - `exec` replaces the current shell process — useful for redirecting script output
