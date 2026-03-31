# Shell Scripting — Conditionals

## The `if` Statement

```bash
if [ condition ]; then
    # commands if true
elif [ other_condition ]; then
    # commands if true
else
    # commands if all conditions false
fi
```

---

## `[ ]` — The `test` command

`[ condition ]` is equivalent to the `test` command. Note: **spaces inside brackets are required**.

```bash
# Correct
if [ "$name" = "Alice" ]; then

# Wrong — no spaces
if ["$name" = "Alice"]; then
```

---

## `[[ ]]` — Modern bash test

Bash-specific, more powerful than `[ ]`. Prefer this in bash scripts.

```bash
if [[ "$name" == "Alice" ]]; then   # == pattern matching
if [[ "$str" =~ ^[0-9]+$ ]]; then  # regex matching
if [[ -f "$file" && -r "$file" ]]; then  # && supported
```

---

## String Comparison

```bash
str1="hello"
str2="world"

if [ "$str1" = "$str2" ]; then echo "equal"; fi       # POSIX equal
if [ "$str1" != "$str2" ]; then echo "not equal"; fi

if [[ "$str1" == "$str2" ]]; then echo "equal"; fi    # bash equal
if [[ "$str1" < "$str2" ]]; then echo "str1 comes first alphabetically"; fi
if [[ "$str1" > "$str2" ]]; then echo "str1 comes after"; fi

# Check if string is empty / non-empty
if [ -z "$str1" ]; then echo "empty"; fi    # -z = zero length
if [ -n "$str1" ]; then echo "not empty"; fi  # -n = non-zero length
```

---

## Numeric Comparison

Use `-eq`, `-ne`, `-lt`, `-le`, `-gt`, `-ge` (NOT `==`, `<`, `>` for numbers in `[ ]`):

| Operator | Meaning | Example |
|----------|---------|---------|
| `-eq` | equal | `[ $a -eq $b ]` |
| `-ne` | not equal | `[ $a -ne $b ]` |
| `-lt` | less than | `[ $a -lt $b ]` |
| `-le` | less than or equal | `[ $a -le $b ]` |
| `-gt` | greater than | `[ $a -gt $b ]` |
| `-ge` | greater than or equal | `[ $a -ge $b ]` |

```bash
age=25

if [ $age -ge 18 ]; then
    echo "Adult"
else
    echo "Minor"
fi

# With double parentheses (arithmetic context)
if (( age >= 18 )); then echo "Adult"; fi
if (( age == 25 )); then echo "Twenty-five"; fi
```

---

## File Test Operators

```bash
# Check file existence and type
if [ -e "$path" ]; then   # exists (file or dir)
if [ -f "$path" ]; then   # is a regular file
if [ -d "$path" ]; then   # is a directory
if [ -L "$path" ]; then   # is a symbolic link
if [ -s "$path" ]; then   # file exists and is NOT empty

# Check permissions
if [ -r "$path" ]; then   # readable
if [ -w "$path" ]; then   # writable
if [ -x "$path" ]; then   # executable

# Compare files
if [ "$f1" -nt "$f2" ]; then  # f1 newer than f2
if [ "$f1" -ot "$f2" ]; then  # f1 older than f2
if [ "$f1" -ef "$f2" ]; then  # same file (hard link)
```

Example:
```bash
config="/etc/myapp/config.cfg"

if [ ! -f "$config" ]; then
    echo "Config file missing!" >&2
    exit 1
fi

if [ ! -r "$config" ]; then
    echo "Cannot read config file" >&2
    exit 1
fi

echo "Config found and readable"
```

---

## Logical Operators

```bash
# AND
if [ -f "$file" ] && [ -r "$file" ]; then ...   # in [ ]
if [[ -f "$file" && -r "$file" ]]; then ...      # in [[ ]]

# OR
if [ $x -eq 1 ] || [ $x -eq 2 ]; then ...
if [[ $x -eq 1 || $x -eq 2 ]]; then ...

# NOT
if [ ! -f "$file" ]; then echo "file not found"; fi
if [[ ! "$str" =~ ^[0-9]+$ ]]; then echo "not a number"; fi
```

---

## Practical `if` Examples

```bash
#!/usr/bin/env bash

# Check if user is root
if [[ $EUID -ne 0 ]]; then
    echo "This script must be run as root" >&2
    exit 1
fi

# Check if command exists
if ! command -v git &>/dev/null; then
    echo "git is not installed"
    exit 1
fi

# Check if directory exists, create if not
dir="/var/log/myapp"
if [[ ! -d "$dir" ]]; then
    mkdir -p "$dir"
    echo "Created $dir"
fi

# Validate argument
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <filename>"
    exit 1
fi
```

---

## `case` Statement

Better than chained `if/elif` when comparing one value against many patterns.

```bash
case "$variable" in
    pattern1)
        commands ;;
    pattern2 | pattern3)    # multiple patterns with |
        commands ;;
    pattern*)               # wildcard
        commands ;;
    *)                      # default (like else)
        commands ;;
esac
```

### Example — Day of week

```bash
#!/usr/bin/env bash
read -p "Enter day: " day

case "$day" in
    Monday|Tuesday|Wednesday|Thursday|Friday)
        echo "Weekday" ;;
    Saturday|Sunday)
        echo "Weekend" ;;
    *)
        echo "Unknown day" ;;
esac
```

### Example — File extension handler

```bash
file="report.pdf"

case "$file" in
    *.txt)  echo "Text file" ;;
    *.pdf)  echo "PDF document" ;;
    *.jpg|*.jpeg|*.png)  echo "Image file" ;;
    *.sh)   echo "Shell script" ;;
    *)      echo "Unknown type" ;;
esac
```

### Example — Menu

```bash
#!/usr/bin/env bash

echo "====== Menu ======"
echo "1. List files"
echo "2. Disk usage"
echo "3. Quit"
read -p "Choose [1-3]: " choice

case "$choice" in
    1) ls -la ;;
    2) df -h ;;
    3) echo "Goodbye!"; exit 0 ;;
    *) echo "Invalid choice" ;;
esac
```

---

## Ternary-style (short if)

```bash
# Single line if-then
[ -f "/etc/hosts" ] && echo "hosts file exists"

# One-liner if-else using ||
[ -f "$file" ] && echo "found" || echo "not found"

# Conditional assignment
status=$([ $exit_code -eq 0 ] && echo "OK" || echo "FAIL")
```

---

## Key Takeaways

> - Always quote variables in conditions: `"$var"`, never bare `$var`
> - Use `[[ ]]` over `[ ]` in bash — it supports `&&`, `||`, `=~`, and `<`/`>`  
> - Use `-eq`, `-ne`, `-lt`, etc. for numbers; `=`, `!=`, `<`, `>` for strings
> - `-f` checks regular file, `-d` checks directory, `-e` checks either
> - `case` is cleaner than many `elif` branches when matching one variable
> - `&&` chains commands: second only runs if first succeeds
