
# Shell Scripting — Conditionals (Simple & Student‑Friendly)

## ✅ Basic `if` Statement
```bash
if [ condition ]; then
    commands
elif [ other_condition ]; then
    commands
else
    commands
fi
```

✅ Always keep **spaces inside brackets**.

---

## ✅ `[ ]` vs `[[ ]]`
- `[ ]` → POSIX, works in all shells
- `[[ ]]` → bash only, safer & more powerful

```bash
if [[ "$name" == "Alice" ]]; then echo "Hi"; fi
if [[ "$str" =~ ^[0-9]+$ ]]; then echo "Numbers"; fi
```

---

## ✅ String Comparison
```bash
if [ "$a" = "$b" ]; then echo "equal"; fi
if [ "$a" != "$b" ]; then echo "not equal"; fi

if [[ "$a" < "$b" ]]; then echo "a comes first"; fi
if [[ -z "$a" ]]; then echo "empty"; fi
if [[ -n "$a" ]]; then echo "not empty"; fi
```

---

## ✅ Numeric Comparison
Use: `-eq -ne -lt -le -gt -ge`

```bash
age=25
if [ $age -ge 18 ]; then echo "Adult"; fi
if (( age >= 18 )); then echo "Adult"; fi
```

---

## ✅ File Tests
```bash
if [ -e "$p" ]; then echo "exists"; fi
if [ -f "$p" ]; then echo "file"; fi
if [ -d "$p" ]; then echo "directory"; fi
if [ -r "$p" ]; then echo "readable"; fi
if [ -w "$p" ]; then echo "writable"; fi
if [ -x "$p" ]; then echo "executable"; fi
```

---

## ✅ Logical Operators
```bash
# AND
if [[ -f "$f" && -r "$f" ]]; then ...; fi

# OR
if [[ $x -eq 1 || $x -eq 2 ]]; then ...; fi

# NOT
if [[ ! -f "$f" ]]; then ...; fi
```

---

## ✅ Practical Examples
### Check root
```bash
if [[ $EUID -ne 0 ]]; then
    echo "Run as root" >&2
    exit 1
fi
```

### Check command
```bash
if ! command -v git &>/dev/null; then
    echo "git missing"
fi
```

### Ensure directory
```bash
dir="/var/log/myapp"
if [[ ! -d "$dir" ]]; then
    mkdir -p "$dir"
fi
```

---

## ✅ `case` Statement
Cleaner alternative to many `elif` conditions.
```bash
case "$var" in
    yes) echo "yes" ;;
    no) echo "no" ;;
    maybe|not_sure) echo "uncertain" ;;
    *) echo "unknown" ;;
esac
```

### Example: Day
```bash
case "$day" in
    Monday|Tuesday|Wednesday|Thursday|Friday) echo "Weekday" ;;
    Saturday|Sunday) echo "Weekend" ;;
    *) echo "Unknown" ;;
esac
```

---

## ✅ One‑liners
```bash
[ -f "/etc/hosts" ] && echo "exists"
[ -f "$file" ] && echo "found" || echo "not found"
```

---

## ✅ Key Takeaways
- Quote variables: `"$var"`
- Prefer `[[ ]]` in bash
- Use numeric operators for numbers, string operators for strings
- File tests: `-f`, `-d`, `-e`, `-r`, `-w`, `-x`
- `case` simplifies multiple comparisons
