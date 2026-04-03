
# Shell Scripting — Functions (Simple & Student‑Friendly)

## ✅ Defining Functions
Two valid forms:
```bash
# Recommended (portable)
my_func() {
    commands
}

# Bash-only alternative
function my_func {
    commands
}
```
✅ Prefer the first style for portability.

---
## ✅ Basic Example
```bash
greet() {
    echo "Hello, World!"
}

greet   # call the function
```

---
## ✅ Function Arguments
Handled like script arguments: `$1`, `$2`, `$3` …
```bash
greet() {
    echo "Hello, $1!"
}

greet "Alice"
greet "Bob"
```

Multiple arguments:
```bash
add() {
    local a=$1 b=$2
    echo $(( a + b ))
}

sum=$(add 5 10)
echo "Sum: $sum"
```

Show argument details:
```bash
show_args() {
    echo "First: $1"
    echo "All:   $@"
    echo "Count: $#"
}
```

---
## ✅ Returning Values
`return` returns only exit codes (0–255). For real values, use:

### Method 1: Echo + command substitution ✅ Best
```bash
square() {
    echo $(( $1 * $1 ))
}

result=$(square 6)
```

### Method 2: Global variable (avoid when possible)
```bash
multiply() { result=$(( $1 * $2 )); }
```

### Method 3: Nameref (bash 4.3+)
```bash
calc() {
    local -n ref=$1
    ref=$(( $2 + $3 ))
}

calc total 10 20
```

### Exit codes for true/false
```bash
is_even() {
    (( $1 % 2 == 0 ))
}

if is_even 4; then echo "Even"; fi
```

---
## ✅ Local Variables
Use `local` to avoid polluting global scope.
```bash
x=10
change() {
    local x=99
    echo "Inside: $x"
}
change
echo "Outside: $x"
```

---
## ✅ Function Libraries
Create reusable utility files:
```bash
# lib/utils.sh
log()  { echo "[INFO]  $*"; }
warn() { echo "[WARN]  $*" >&2; }
error(){ echo "[ERROR] $*" >&2; }
```

Use them:
```bash
source "lib/utils.sh"
log "Starting script..."
```

---
## ✅ Recursive Functions
```bash
factorial() {
    local n=$1
    if (( n <= 1 )); then
        echo 1
    else
        local prev=$(factorial $(( n - 1 )))
        echo $(( n * prev ))
    fi
}

factorial 5
```

---
## ✅ Practical Examples
### Check if command exists
```bash
exists() {
    command -v "$1" &>/dev/null
}
```

### Retry with backoff
```bash
retry() {
    local max=$1 delay=$2
    shift 2
    for ((i=1; i<=max; i++)); do
        "$@" && return 0
        sleep $delay
        delay=$((delay * 2))
    done
    return 1
}
```

### Confirm prompt
```bash
confirm() {
    read -p "${1:-Are you sure?} [y/N]: " ans
    [[ $ans =~ ^[Yy]$ ]]
}
```

---
## ✅ Key Takeaways
- Use portable syntax: `my_func(){}`
- Use `local` to avoid global variable issues
- Use `echo` + `$(...)` for returning values
- Use exit codes (`return 0` / `return 1`) for success/failure
- Reuse code via sourced libraries (`source utils.sh`)
- Functions share the script’s environment
