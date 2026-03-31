# Shell Scripting — Functions

## Defining Functions

Two equivalent syntaxes:

```bash
# Syntax 1 (recommended)
function_name() {
    commands
}

# Syntax 2
function function_name {
    commands
}
```

> **Best practice:** Use Syntax 1 — it works in both `bash` and `sh`.

---

## Basic Function Example

```bash
#!/usr/bin/env bash

greet() {
    echo "Hello, World!"
}

# Call the function (no parentheses when calling)
greet
```

---

## Function Arguments

Functions receive arguments the same way scripts do: `$1`, `$2`, `$3` etc.

```bash
greet() {
    echo "Hello, $1!"
}

greet "Alice"    # Hello, Alice!
greet "Bob"      # Hello, Bob!

# Multiple arguments
add_numbers() {
    local a="$1"
    local b="$2"
    echo $(( a + b ))
}

result=$(add_numbers 5 10)
echo "Sum: $result"    # Sum: 15
```

### Argument special variables (same as scripts)

```bash
show_args() {
    echo "Function name (not useful here): $0"
    echo "First arg: $1"
    echo "All args: $@"
    echo "Arg count: $#"
}

show_args one two three
```

---

## Returning Values

**Important:** `return` only sets the exit code (0–255). To return a value, use one of these methods:

### Method 1: Echo + Command Substitution (most common)

```bash
square() {
    local num=$1
    echo $(( num * num ))
}

result=$(square 7)
echo "7 squared = $result"    # 7 squared = 49
```

### Method 2: Modify a global variable (avoid if possible)

```bash
RESULT=""

multiply() {
    RESULT=$(( $1 * $2 ))
}

multiply 4 5
echo $RESULT    # 20
```

### Method 3: Use nameref (bash 4.3+)

```bash
calculate() {
    local -n _result=$1   # nameref to variable named by $1
    local a=$2 b=$3
    _result=$(( a + b ))
}

calculate my_var 10 20
echo $my_var    # 30
```

### Return exit codes

Use `return 0` for success, non-zero for failure:

```bash
is_even() {
    local num=$1
    if (( num % 2 == 0 )); then
        return 0   # success / true
    else
        return 1   # failure / false
    fi
}

if is_even 4; then echo "4 is even"; fi
if ! is_even 7; then echo "7 is odd"; fi
```

---

## Local Variables

Always use `local` for function-internal variables to avoid polluting global scope.

```bash
x=10   # global

change_x() {
    local x=99   # local — only inside this function
    echo "Inside function: x = $x"
}

change_x
echo "Outside function: x = $x"   # still 10
```

---

## Function Libraries

Create a file of reusable functions and source it:

```bash
# lib/utils.sh
log()     { echo "[$(date '+%H:%M:%S')] INFO  $*"; }
warn()    { echo "[$(date '+%H:%M:%S')] WARN  $*" >&2; }
error()   { echo "[$(date '+%H:%M:%S')] ERROR $*" >&2; }
die()     { error "$*"; exit 1; }

require_cmd() {
    command -v "$1" &>/dev/null || die "Required command not found: $1"
}

is_root() {
    [[ $EUID -eq 0 ]]
}
```

Use in your script:
```bash
#!/usr/bin/env bash
source "$(dirname "$0")/lib/utils.sh"

require_cmd docker
is_root || die "This script requires root privileges"

log "Starting deployment..."
```

---

## Recursive Functions

```bash
# Factorial
factorial() {
    local n=$1
    if (( n <= 1 )); then
        echo 1
    else
        local prev=$(factorial $(( n - 1 )))
        echo $(( n * prev ))
    fi
}

echo "5! = $(factorial 5)"    # 5! = 120
echo "10! = $(factorial 10)"  # 10! = 3628800
```

---

## Practical Function Examples

### Check if command exists

```bash
command_exists() {
    command -v "$1" &>/dev/null
}

if command_exists git; then
    echo "git version: $(git --version)"
else
    echo "git not installed"
fi
```

### Retry with backoff

```bash
retry() {
    local max_attempts=$1
    local delay=$2
    shift 2
    local cmd=("$@")
    local attempt=1

    while (( attempt <= max_attempts )); do
        "${cmd[@]}" && return 0
        echo "Attempt $attempt/$max_attempts failed. Retrying in ${delay}s..."
        sleep "$delay"
        (( attempt++ ))
        (( delay *= 2 ))   # exponential backoff
    done

    echo "All $max_attempts attempts failed" >&2
    return 1
}

retry 3 2 curl -sf https://api.example.com/health
```

### Confirm (yes/no prompt)

```bash
confirm() {
    local prompt="${1:-Are you sure?}"
    read -p "$prompt [y/N]: " reply
    [[ "$reply" =~ ^[Yy]$ ]]
}

if confirm "Delete all logs?"; then
    rm -f /var/log/myapp/*.log
    echo "Logs deleted"
else
    echo "Aborted"
fi
```

### Create timestamped log

```bash
LOG_FILE="/var/log/myapp/app.log"

log() {
    local level="${1:-INFO}"
    shift
    printf '[%s] [%-5s] %s\n' "$(date '+%Y-%m-%d %H:%M:%S')" "$level" "$*" | tee -a "$LOG_FILE"
}

log "INFO"  "Application starting"
log "WARN"  "Disk usage above 80%"
log "ERROR" "Database connection failed"
```

---

## Key Takeaways

> - Define functions before calling them
> - Always use `local` for variables inside functions
> - Use `echo` + `$(...)` to return string values from functions
> - Use `return 0/1` for boolean true/false (exit codes only)
> - Source your library files with `source lib/utils.sh`
> - Functions share the same environment as the script unless you use `local`
> - Recursive functions work, but watch out for deep recursion (no tail-call optimization)
