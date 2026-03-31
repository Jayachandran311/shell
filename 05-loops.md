# Shell Scripting — Loops

## `for` Loop

### Iterate over a list of values

```bash
for item in apple banana cherry; do
    echo "Fruit: $item"
done
```

### Iterate over a range of numbers

```bash
for i in {1..10}; do
    echo "Number: $i"
done

# With step
for i in {0..20..5}; do   # 0 5 10 15 20
    echo $i
done
```

### C-style `for` loop

```bash
for (( i = 0; i < 5; i++ )); do
    echo "i = $i"
done

# Countdown
for (( i = 10; i >= 1; i-- )); do
    echo "$i..."
done
echo "Liftoff!"
```

### Iterate over command output

```bash
# Loop over files
for file in *.sh; do
    echo "Script: $file"
    chmod +x "$file"
done

# Loop over lines of a command
for user in $(cut -d: -f1 /etc/passwd); do
    echo "User: $user"
done

# Better way to loop over lines (handles spaces)
while IFS= read -r user; do
    echo "User: $user"
done < <(cut -d: -f1 /etc/passwd)
```

### Iterate over an array

```bash
servers=("web01" "web02" "db01" "db02")

for server in "${servers[@]}"; do
    echo "Pinging: $server"
    ping -c 1 "$server" &>/dev/null && echo "  OK" || echo "  UNREACHABLE"
done
```

---

## `while` Loop

Runs while condition is **true**.

```bash
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    (( count++ ))
done
```

### Read a file line by line (preferred method)

```bash
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/hosts

# Process a file with a variable
filename="data.txt"
while IFS= read -r line; do
    echo "Processing: $line"
done < "$filename"
```

### Infinite loop (with break)

```bash
while true; do
    read -p "Enter command (q to quit): " cmd
    case "$cmd" in
        q|quit|exit) echo "Goodbye!"; break ;;
        help) echo "Available: q, help" ;;
        *) echo "Unknown command: $cmd" ;;
    esac
done
```

### While loop reading from a command

```bash
# Process output line by line
while IFS= read -r line; do
    echo "Server: $line"
done < <(dig +short example.com)
```

---

## `until` Loop

Runs while condition is **false** (opposite of `while`).

```bash
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    (( count++ ))
done

# Wait for a service to become available
until curl -sf http://localhost:8080/health; do
    echo "Waiting for service..."
    sleep 2
done
echo "Service is up!"
```

---

## `break` and `continue`

```bash
# break — exit loop immediately
for i in {1..10}; do
    if [ $i -eq 6 ]; then
        echo "Breaking at $i"
        break
    fi
    echo $i
done
# Output: 1 2 3 4 5 Breaking at 6

# continue — skip rest of loop body
for i in {1..10}; do
    if (( i % 2 == 0 )); then
        continue   # skip even numbers
    fi
    echo $i
done
# Output: 1 3 5 7 9

# break with level (nested loops)
for i in {1..3}; do
    for j in {1..3}; do
        if [ $j -eq 2 ]; then
            break 2   # break out of BOTH loops
        fi
        echo "$i,$j"
    done
done
```

---

## Loop Control Tricks

### Process multiple items in parallel

```bash
servers=("web01" "web02" "web03" "db01")

for server in "${servers[@]}"; do
    ssh "$server" "uptime" &   # & sends to background
done
wait   # wait for all background jobs to finish
echo "All servers checked"
```

### Retry loop

```bash
MAX_RETRIES=3
attempt=1

while [ $attempt -le $MAX_RETRIES ]; do
    echo "Attempt $attempt..."
    if some_command; then
        echo "Success!"
        break
    fi
    (( attempt++ ))
    sleep 2
done

if [ $attempt -gt $MAX_RETRIES ]; then
    echo "Failed after $MAX_RETRIES attempts" >&2
    exit 1
fi
```

### Loop with index

```bash
fruits=("apple" "banana" "cherry")

for i in "${!fruits[@]}"; do   # ${!array[@]} = indices
    echo "$i: ${fruits[$i]}"
done
# 0: apple
# 1: banana
# 2: cherry
```

---

## Practical Examples

### Backup files in a directory

```bash
#!/usr/bin/env bash
set -euo pipefail

SOURCE="/home/alice/documents"
DEST="/backup/$(date +%Y-%m-%d)"
mkdir -p "$DEST"

for file in "$SOURCE"/*; do
    if [ -f "$file" ]; then
        cp "$file" "$DEST/"
        echo "Backed up: $(basename "$file")"
    fi
done
echo "Backup complete → $DEST"
```

### Health check all servers

```bash
#!/usr/bin/env bash
SERVERS=("192.168.1.10" "192.168.1.11" "192.168.1.12")
FAILED=()

for server in "${SERVERS[@]}"; do
    if ping -c 1 -W 2 "$server" &>/dev/null; then
        echo "✓ $server — UP"
    else
        echo "✗ $server — DOWN"
        FAILED+=("$server")
    fi
done

echo ""
echo "Down servers: ${#FAILED[@]}"
for s in "${FAILED[@]}"; do echo "  - $s"; done
```

### Rename files in bulk

```bash
# Add prefix to all .txt files
for file in *.txt; do
    mv "$file" "backup_${file}"
done

# Convert .jpeg to .jpg
for file in *.jpeg; do
    mv "$file" "${file%.jpeg}.jpg"
done
```

---

## Key Takeaways

> - `for item in list` — iterate over words/values
> - `for (( i=0; i<n; i++ ))` — C-style numeric loop  
> - `while condition` — run while true; `until condition` — run while false
> - Use `while IFS= read -r line; done < file` to safely loop over file lines
> - `break` exits the loop; `continue` skips to next iteration
> - `&` in a loop runs iterations in parallel; use `wait` to sync
> - Always quote array elements: `"${array[@]}"`
