
# Shell Scripting — Variables (Simple & Student‑Friendly)

## ✅ Declaring Variables
```bash
name="Alice"
age=30
pi=3.14

echo "$name"
echo "${name}"
```
✅ **No spaces** around `=` (wrong: `name = "Alice"`).

---

## ✅ Quoting Rules
| Type | What it does |
|------|--------------|
| "double" | Expands variables (`$var`) and commands (`$( )`) |
| 'single' | Literal text, no expansion |
| backticks | Old command substitution |

Examples:
```bash
name="Alice"
echo "Hello $name"       # Hello Alice
echo 'Hello $name'       # literal
```

✅ Always quote variables: `"$var"`

---

## ✅ Variable Types
### ✅ Strings
```bash
greeting="Hello World"
echo "$greeting"
echo ${#greeting}   # length
```

### ✅ Integers
```bash
count=10
count=$((count + 1))

declare -i num=5
num+=3
```

### ✅ Readonly
```bash
readonly MAX=100
```

### ✅ Local (inside functions)
```bash
greet() {
    local msg="Hi"
    echo "$msg"
}
```

---

## ✅ Special Variables
| Var | Meaning |
|-----|---------|
| $0 | Script name |
| $1 $2 | First/second argument |
| $# | Total arguments |
| $@ | All arguments |
| $$ | PID of script |
| $? | Exit code of last command |

Example:
```bash
echo "$0 $1 $2 $# $@ $$"
```

---

## ✅ Environment Variables
```bash
echo $HOME
echo $PATH
export MY_VAR="value"
unset MY_VAR
```

Common ones:
- `$USER`
- `$SHELL`
- `$PWD`
- `$EDITOR`

---

## ✅ Parameter Expansion
```bash
name="Alice"
echo ${name:-Guest}     # default value
echo ${name:=Default}   # set if empty
echo ${name:?Required}  # error if empty
```

---

## ✅ String Operations
```bash
str="Hello World"

echo ${#str}        # length

echo ${str:6}       # substring
echo ${str,,}       # lowercase
echo ${str^^}       # uppercase

echo ${str/World/Bash}  # replace
```

---

## ✅ Arrays (Basics)
```bash
fruits=("apple" "banana" "cherry")
echo ${fruits[0]}
echo ${fruits[@]}
echo ${#fruits[@]}
```

---

## ✅ Capture Command Output
```bash
today=$(date)
files=$(ls /etc | wc -l)
```

---

## ✅ Key Takeaways
- No spaces around `=`
- Always quote variables: `"$var"`
- Use `local` inside functions
- Use `export` to pass variables to child processes
- `$1`, `$2`, `$@`, `$#` help handle script arguments
