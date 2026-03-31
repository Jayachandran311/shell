# Shell Scripting — Advanced Scripting

## Arithmetic Operations

```bash
# Double parentheses arithmetic
a=10; b=3

echo $(( a + b ))     # 13
echo $(( a - b ))     # 7
echo $(( a * b ))     # 30
echo $(( a / b ))     # 3  (integer division)
echo $(( a % b ))     # 1  (modulo)
echo $(( a ** b ))    # 1000 (exponentiation)

# Increment / decrement
(( a++ ))    # a = 11
(( a-- ))    # a = 10
(( a += 5 )) # a = 15
(( a *= 2 )) # a = 30

# Compare in arithmetic context
if (( a > 20 )); then echo "big"; fi

# Floating point — use bc or awk
result=$(echo "scale=2; 10 / 3" | bc)        # 3.33
result=$(awk 'BEGIN { printf "%.2f", 10/3 }') # 3.33
```

---

## String Processing

### `grep`

```bash
grep "pattern" file.txt           # basic search
grep -i "pattern" file.txt        # case-insensitive
grep -r "pattern" directory/      # recursive
grep -n "pattern" file.txt        # show line numbers
grep -l "pattern" /etc/*.conf     # list matching files
grep -v "pattern" file.txt        # invert (lines NOT matching)
grep -c "pattern" file.txt        # count matching lines
grep -E "regex+" file.txt         # extended regex
grep -o "matched_part" file.txt   # only print matched part

# Useful combos
grep "ERROR" app.log | grep -v DEBUG     # errors but not debug
grep -E "^[0-9]{4}-[0-9]{2}" logfile    # lines starting with date
```

### `sed` — Stream Editor

```bash
# Substitute first occurrence per line
sed 's/old/new/' file.txt

# Substitute ALL occurrences per line
sed 's/old/new/g' file.txt

# Case-insensitive
sed 's/old/new/gi' file.txt

# Edit file in-place
sed -i 's/old/new/g' file.txt
sed -i.bak 's/old/new/g' file.txt  # with backup

# Delete lines
sed '5d' file.txt            # delete line 5
sed '/pattern/d' file.txt    # delete lines matching pattern
sed '/^#/d' file.txt         # delete comment lines
sed '/^$/d' file.txt         # delete empty lines

# Print specific lines
sed -n '5,10p' file.txt      # print lines 5-10
sed -n '/START/,/END/p'      # print between markers

# Multiple operations
sed -e 's/foo/bar/g' -e '/^#/d' file.txt
```

### `awk` — Text Processing Power Tool

```bash
# Print specific column
awk '{print $1}' file.txt         # first field
awk '{print $NF}' file.txt        # last field
awk -F: '{print $1}' /etc/passwd  # use : as field separator

# Filter and print
awk '$3 > 1000' /etc/passwd              # lines where field 3 > 1000
awk '/nginx/ {print $1}' access.log      # lines matching pattern

# Calculations
awk '{sum += $1} END {print "Sum:", sum}' numbers.txt
awk 'NR > 1 {total += $5} END {avg = total/(NR-1); print "Avg:", avg}' data.csv

# Print formatted report
awk -F: '{printf "%-20s %s\n", $1, $7}' /etc/passwd

# Built-in variables
# NR  = current record (line) number
# NF  = number of fields in current line
# FS  = field separator (default: space)
# OFS = output field separator
# $0  = entire current line
```

---

## Regular Expressions (Regex) in Bash

```bash
# Match regex with [[ ]]
if [[ "$string" =~ ^[0-9]+$ ]]; then echo "All digits"; fi
if [[ "$email" =~ ^[^@]+@[^@]+\.[^@]+$ ]]; then echo "Valid email format"; fi
if [[ "$ip" =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then echo "IP format"; fi

# Capture groups
if [[ "2024-03-31" =~ ^([0-9]{4})-([0-9]{2})-([0-9]{2})$ ]]; then
    year="${BASH_REMATCH[1]}"
    month="${BASH_REMATCH[2]}"
    day="${BASH_REMATCH[3]}"
    echo "Year=$year Month=$month Day=$day"
fi
```

---

## Script Arguments with `getopts`

A proper way to parse flags/options:

```bash
#!/usr/bin/env bash
set -euo pipefail

usage() {
    echo "Usage: $0 [-h] [-v] [-n NAME] [-p PORT]"
    echo "  -h  Help"
    echo "  -v  Verbose"
    echo "  -n  Name (required)"
    echo "  -p  Port (default: 8080)"
    exit 0
}

VERBOSE=false
NAME=""
PORT=8080

while getopts "hvn:p:" opt; do
    case "$opt" in
        h) usage ;;
        v) VERBOSE=true ;;
        n) NAME="$OPTARG" ;;
        p) PORT="$OPTARG" ;;
        ?) echo "Unknown option: -$OPTARG"; usage ;;
    esac
done

shift $(( OPTIND - 1 ))   # remaining positional args

[[ -z "$NAME" ]] && { echo "Error: -n NAME is required"; usage; }

$VERBOSE && echo "Verbose mode ON"
echo "Name: $NAME, Port: $PORT"
echo "Remaining args: $@"
```

Run: `./script.sh -v -n myapp -p 3000 extra_arg`

---

## Colored Output Helper

```bash
#!/usr/bin/env bash

# Color codes
RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'
BLUE='\033[0;34m'; CYAN='\033[0;36m'; BOLD='\033[1m'; NC='\033[0m'

info()    { echo -e "${BLUE}[INFO]${NC}  $*"; }
success() { echo -e "${GREEN}[OK]${NC}    $*"; }
warn()    { echo -e "${YELLOW}[WARN]${NC}  $*"; }
error()   { echo -e "${RED}[ERROR]${NC} $*" >&2; }
die()     { error "$*"; exit 1; }
header()  { echo -e "\n${BOLD}${CYAN}=== $* ===${NC}\n"; }

header "Deployment Starting"
info "Checking prerequisites..."
success "Docker found: $(docker --version)"
warn "Disk usage is 75%"
error "Database connection failed"
```

---

## Configuration Files

### Parse INI-style config

```bash
# config.ini
# [database]
# host=localhost
# port=5432

parse_ini() {
    local file=$1
    local section=$2
    local key=$3
    
    awk -F= -v section="[$section]" -v key="$key" '
        $0 == section { in_section=1; next }
        /^\[/ { in_section=0 }
        in_section && $1 == key { gsub(/^ +| +$/, "", $2); print $2; exit }
    ' "$file"
}

DB_HOST=$(parse_ini config.ini database host)
echo "DB Host: $DB_HOST"
```

### Environment file (.env)

```bash
# Load .env file
if [[ -f .env ]]; then
    set -o allexport
    source .env
    set +o allexport
fi
```

---

## Error Handling Best Practices

```bash
#!/usr/bin/env bash
set -euo pipefail

# Trap all errors
trap 'echo "Error at line $LINENO: $BASH_COMMAND" >&2' ERR

# Check return codes
if ! output=$(some_command 2>&1); then
    echo "Command failed: $output" >&2
    exit 1
fi

# Check required variables
: "${REQUIRED_VAR:?Variable REQUIRED_VAR must be set}"

# Validate numeric input
[[ "$1" =~ ^[0-9]+$ ]] || { echo "Argument must be a number"; exit 1; }
```

---

## Key Takeaways

> - Use `$(( ))` for integer arithmetic; `bc` or `awk` for floating point
> - Master `grep`, `sed`, `awk` — they're critical for text processing
> - Use `[[ "$var" =~ regex ]]` for regex matching in bash; `${BASH_REMATCH[@]}` for captures
> - Use `getopts` to parse command-line flags properly
> - Use `set -euo pipefail` + `trap ... ERR` for robust error handling
> - Keep scripts modular: break into functions, source shared libraries
