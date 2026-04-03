
# Shell Scripting — Loops (Simple & Student‑Friendly)

## ✅ `for` Loop
### List of values
```bash
for item in apple banana cherry; do
    echo "Fruit: $item"
done
```

### Number ranges
```bash
for i in {1..10}; do echo "$i"; done
for i in {0..20..5}; do echo "$i"; done
```

### C‑style
```bash
for (( i=0; i<5; i++ )); do echo "i=$i"; done
for (( i=10; i>=1; i-- )); do echo "$i..."; done
```

### Loop over files
```bash
for file in *.sh; do
    echo "Script: $file"
    chmod +x "$file"
done
```

### Loop over command output
```bash
for user in $(cut -d: -f1 /etc/passwd); do echo "$user"; done
```

Better (handles spaces):
```bash
while IFS= read -r user; do
    echo "$user"
done < <(cut -d: -f1 /etc/passwd)
```

### Loop over array
```bash
servers=("web01" "web02" "db01")
for s in "${servers[@]}"; do echo "Pinging $s"; done
```

---

## ✅ `while` Loop
```bash
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    (( count++ ))
done
```

### Read file line by line
```bash
while IFS= read -r line; do
    echo "$line"
done < file.txt
```

### Infinite loop
```bash
while true; do
    read -p "cmd: " c
    [[ $c == q ]] && break
    echo "You typed: $c"
done
```

---

## ✅ `until` Loop (opposite of while)
```bash
count=1
until [ $count -gt 5 ]; do
    echo "$count"
    (( count++ ))
done
```

---

## ✅ `break` & `continue`
```bash
for i in {1..10}; do
    [[ $i -eq 6 ]] && break
    echo $i
done

for i in {1..10}; do
    (( i%2==0 )) && continue
    echo $i
done
```

---

## ✅ Useful Loop Tricks
### Run tasks in parallel
```bash
for s in "${servers[@]}"; do
    ssh "$s" uptime &
done
wait
```

### Retry loop
```bash
MAX=3
try=1
while [ $try -le $MAX ]; do
    echo "Try $try"
    if some_command; then break; fi
    (( try++ ))
    sleep 2
done
```

### Loop with index
```bash
for i in "${!servers[@]}"; do
    echo "$i: ${servers[$i]}"
done
```

---

## ✅ Practical Examples
### Backup directory
```bash
SRC="/home/user/docs"
DEST="/backup/$(date +%F)"
mkdir -p "$DEST"
for f in "$SRC"/*; do
    cp "$f" "$DEST"
    echo "Backed up: $(basename "$f")"
done
```

### Health check servers
```bash
FAILED=()
for s in "${servers[@]}"; do
    if ping -c1 "$s" &>/dev/null; then
        echo "UP: $s"
    else
        FAILED+=("$s")
    fi
done
```

---

## ✅ Key Takeaways
- `for` loops handle lists, ranges, arrays.
- `while` runs while condition is **true**.
- `until` runs while condition is **false**.
- Use `while read -r` to loop over file lines.
- `break` stops a loop; `continue` skips iteration.
- Use `&` for parallel tasks + `wait`.
- Always quote arrays: `"${arr[@]}"`.
