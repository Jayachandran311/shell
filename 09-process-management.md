
# Shell Scripting — Process Management (Simple & Student‑Friendly)

## ✅ Running Commands in Background
```bash
long_running_command &    # run in background
sleep 60 &

echo "Background PID: $!"   # PID of last background job

nohup ./script.sh > output.log 2>&1 &  # survive terminal close
disown $!
```

---
## ✅ Job Control
```bash
long_task &       # send to background
jobs              # list jobs
jobs -l           # show PIDs

fg %1             # bring job 1 to foreground
bg %2             # resume job 2 in background
```
Keyboard shortcuts:
- **Ctrl+Z** → pause & send to background (stopped)
- **Ctrl+C** → interrupt/kill foreground job

---
## ✅ `wait` — Wait for Processes
```bash
backup_server web01 &
backup_server web02 &
wait               # wait for all

echo "Done"
```

Wait for specific PID:
```bash
mv bigfile /archive &
PID=$!
wait $PID && echo "OK" || echo "Failed"
```

---
## ✅ Signals & `trap`
### Common Signals
| Signal | Meaning |
|--------|---------|
| SIGINT | Ctrl+C |
| SIGTERM | kill (default) |
| SIGHUP | terminal closed |
| SIGKILL | force kill, cannot be trapped |
| EXIT | script ending |

### Trap cleanup
```bash
TMPFILE=$(mktemp)
cleanup() {
  rm -f "$TMPFILE"
  echo "Cleaned up"
}
trap cleanup EXIT
```

Handle specific signals:
```bash
trap cleanup EXIT INT TERM
```

Ignore / restore:
```bash
trap '' SIGINT
trap - SIGINT
```

Graceful shutdown:
```bash
RUNNING=true
shutdown(){ RUNNING=false; }
trap shutdown SIGINT SIGTERM

while $RUNNING; do
  echo "Working..."; sleep 5
done
```

---
## ✅ Sending Signals (`kill`)
```bash
kill PID          # SIGTERM
kill -9 PID       # SIGKILL (force)
kill -HUP PID     # reload
kill -0 PID       # check if exists

killall nginx
pkill -f "python script.py"
```

---
## ✅ Check if Process Running
```bash
if kill -0 $PID 2>/dev/null; then
  echo "Running"
fi

pgrep nginx >/dev/null && echo "nginx running"
```

Wait for process stop:
```bash
while kill -0 $PID 2>/dev/null; do sleep 1; done
```

---
## ✅ Subshells
```bash
x=10
(
  x=99
  echo "Inside: $x"
)
echo "Outside: $x"   # still 10
```

Temporary dir changes:
```bash
(cd /tmp && ls)
```

Process substitution:
```bash
diff <(sort a.txt) <(sort b.txt)
```

---
## ✅ `exec` — Replace Current Shell
```bash
exec other_program        # replaces current shell
exec > logfile.log        # redirect stdout for rest of script
exec 2> errors.log        # redirect stderr

# Log everything to file
exec > >(tee -a /var/log/app.log) 2>&1
```

---
## ✅ Parallel Processing
```bash
process(){ echo "Doing $1"; sleep 2; }
items=(a b c d e)
pids=()

for item in "${items[@]}"; do
  process "$item" &
  pids+=($!)
done

errors=0
for pid in "${pids[@]}"; do
  wait $pid || ((errors++))
done

echo "Errors: $errors"
```

---
## ✅ Key Takeaways
- Use `&` to background a job; use `wait` to synchronize
- `trap` ensures cleanup happens on exit/signals
- `kill` sends signals; `kill -9` cannot be ignored
- `pgrep` / `pkill` work by process name
- Subshells run in `()` and don’t affect parent variables
- `exec` replaces shell & can set global redirection
