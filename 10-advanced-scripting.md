
# Shell Scripting — Advanced Scripting (Simple & Student‑Friendly)

## ✅ Arithmetic Operations
```bash
a=10; b=3

echo $(( a + b ))   # 13
echo $(( a - b ))   # 7
echo $(( a * b ))   # 30
echo $(( a / b ))   # 3   (integer division)
echo $(( a % b ))   # 1   (modulo)
echo $(( a ** b ))  # 1000 (power)

(( a++ ))           # increment
(( a-- ))           # decrement
(( a += 5 ))
(( a *= 2 ))

if (( a > 20 )); then echo "big"; fi

# Floating point via bc/awk
result=$(echo "scale=2; 10/3" | bc)
result=$(awk 'BEGIN{ printf "%.2f", 10/3 }')
```

---
## ✅ String Processing
### grep
```bash
grep "pattern" file.txt
grep -i "pattern" file.txt   # ignore case
grep -r "pattern" dir/       # recursive
grep -n "pattern" file.txt   # line numbers
grep -v "pattern" file.txt   # invert

grep -E "^[0-9]+" file.txt   # extended regex
grep -o "match" file.txt      # only the match
```

### sed
```bash
sed 's/old/new/' file.txt          # replace first
sed 's/old/new/g' file.txt         # replace all
sed -i 's/old/new/g' file.txt      # replace in place

sed '5d' file.txt                  # delete 5th line
sed '/pattern/d' file.txt          # delete matching
sed '/^#/d' file.txt               # remove comments
sed -n '5,10p' file.txt            # print 5-10
```

### awk
```bash
awk '{print $1}' file.txt          # first column
awk -F: '{print $1}' /etc/passwd   # custom delimiter
awk '$3 > 1000' /etc/passwd        # filter

awk '{sum+=$1} END{print sum}' nums.txt
awk 'NR>1 {t+=$5} END{print t/(NR-1)}' data.csv

awk -F: '{printf "%-20s %s
", $1, $7}' /etc/passwd
```

---
## ✅ Regex in Bash
```bash
if [[ "$str" =~ ^[0-9]+$ ]]; then echo digits; fi
if [[ "$email" =~ ^[^@]+@[^@]+\.[^@]+$ ]]; then echo valid; fi

if [[ "2024-03-31" =~ ^([0-9]{4})-([0-9]{2})-([0-9]{2})$ ]]; then
  year=${BASH_REMATCH[1]}
  month=${BASH_REMATCH[2]}
  day=${BASH_REMATCH[3]}
fi
```

---
## ✅ Parsing Arguments with getopts
```bash
while getopts "hvn:p:" opt; do
  case $opt in
    h) echo "Help"; exit ;;
    v) VERBOSE=true ;;
    n) NAME=$OPTARG ;;
    p) PORT=$OPTARG ;;
  esac
done
```

---
## ✅ Colored Output
```bash
RED='[0;31m'; GREEN='[0;32m'; NC='[0m'
info() { echo -e "${GREEN}[INFO]${NC} $*"; }
error(){ echo -e "${RED}[ERR]${NC}  $*" >&2; }
```

---
## ✅ Config Files
### Parse INI
```bash
parse_ini(){
  awk -F= -v sec="[$2]" -v key="$3" '
    $0==sec {f=1; next}
    /^\[/ {f=0}
    f && $1==key {gsub(/^ +| +$/, "", $2); print $2; exit}
  ' "$1"
}

DB_HOST=$(parse_ini config.ini database host)
```

### Load .env
```bash
if [[ -f .env ]]; then
  set -o allexport
  source .env
  set +o allexport
fi
```

---
## ✅ Error Handling
```bash
set -euo pipefail
trap 'echo "Error at line $LINENO: $BASH_COMMAND"' ERR

if ! out=$(cmd 2>&1); then
  echo "Failed: $out" >&2
  exit 1
fi

: "${REQUIRED_VAR:?Required!}"
[[ "$1" =~ ^[0-9]+$ ]] || { echo "Number required"; exit 1; }
```

---
## ✅ Key Takeaways
- Use `$(( ))` for arithmetic; `bc`/`awk` for floating point
- grep/sed/awk = essential text tools
- Use `[[ regex ]]` for regex matching
- Use `getopts` for clean argument parsing
- Use robust patterns: `set -euo pipefail` and traps
- Load configs safely with `.env` or parser functions
