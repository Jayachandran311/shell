# Shell Scripting — Variables

## Declaring Variables

```bash
# No spaces around '='
name="Alice"
age=30
pi=3.14

echo $name       # Alice
echo "$name"     # Alice (always quote variables)
echo "${name}"   # Alice (curly braces for clarity)
```

> **Rule:** No spaces around `=`. `name = "Alice"` is wrong — shell treats it as a command.

---

## Quoting Rules

| Quote type | Behavior |
|-----------|---------|
| `"double"` | Expands variables and `$()` |
| `'single'` | No expansion — literal string |
| `` `backtick` `` | Command substitution (old style) |

```bash
name="Alice"
echo "Hello $name"    # Hello Alice
echo 'Hello $name'    # Hello $name  (literal)
echo "Today: $(date)" # Today: Mon Mar 30 ...
```

> **Best practice:** Always use **double quotes** around variables: `"$var"`. This prevents word splitting and glob expansion on values with spaces.

---

## Variable Types

### String Variables
```bash
greeting="Hello World"
echo $greeting
echo ${#greeting}     # length: 11
```

### Integer Variables
```bash
count=10
count=$((count + 1))   # arithmetic
echo $count             # 11

# Or declare as integer type
declare -i num=5
num+=3
echo $num    # 8
```

### Readonly Variables (Constants)
```bash
readonly MAX_SIZE=100
readonly DB_HOST="localhost"

MAX_SIZE=200   # Error: readonly variable
```

### Local Variables (inside functions)
```bash
greet() {
    local message="Hi there"  # only exists inside this function
    echo "$message"
}
```

---

## Special Variables

| Variable | Meaning |
|----------|---------|
| `$0` | Script name / path |
| `$1`, `$2`, `$3`… | Positional parameters (script arguments) |
| `$#` | Number of arguments passed |
| `$@` | All arguments as separate words |
| `$*` | All arguments as a single word |
| `$$` | PID (process ID) of current shell |
| `$!` | PID of last background process |
| `$?` | Exit code of last command |
| `$-` | Current shell options |
| `$_` | Last argument of previous command |

```bash
#!/usr/bin/env bash
echo "Script: $0"
echo "First arg: $1"
echo "Second arg: $2"
echo "All args: $@"
echo "Arg count: $#"
echo "My PID: $$"
```

Run: `./script.sh hello world`
```
Script: ./script.sh
First arg: hello
Second arg: world
All args: hello world
Arg count: 2
My PID: 12345
```

---

## Environment Variables

Variables available to all processes. Convention: **UPPERCASE**.

```bash
# View all environment variables
env
printenv

# View specific variable
echo $HOME
echo $PATH
echo $USER
echo $SHELL
echo $PWD
echo $OLDPWD

# Set environment variable for child processes
export MY_VAR="custom_value"
export PATH="$PATH:/opt/myapp/bin"   # add to PATH

# Remove (unset) a variable
unset MY_VAR
```

### Common Environment Variables

| Variable | Contains |
|----------|---------|
| `$HOME` | Home directory (`/home/alice`) |
| `$USER` | Current username |
| `$PATH` | Colon-separated list of directories for commands |
| `$PWD` | Current working directory |
| `$SHELL` | Path to current shell |
| `$EDITOR` | Default text editor |
| `$LANG` | System language/locale |
| `$TERM` | Terminal type |
| `$HOSTNAME` | System hostname |

---

## Variable Substitution (Parameter Expansion)

```bash
name="Alice"

# Default value — use default if variable is empty/unset
echo "${name:-Guest}"        # Alice (name is set)
unset name
echo "${name:-Guest}"        # Guest (name is unset)

# Assign default — set variable if unset
echo "${name:=DefaultUser}"  # DefaultUser (and sets $name)
echo $name                   # DefaultUser

# Error if unset
echo "${name:?Variable name is required}"

# Use alternate — use VALUE if variable IS set
echo "${name:+[logged in as $name]}"
```

---

## String Operations

```bash
str="Hello, World!"

echo ${#str}           # Length: 13

echo ${str:7}          # World!  (from index 7)
echo ${str:7:5}        # World   (5 chars from index 7)

echo ${str,,}          # hello, world!  (lowercase)
echo ${str^^}          # HELLO, WORLD!  (uppercase)
echo ${str^}           # Hello, world!  (capitalize first)

# Remove prefix
file="image.jpg"
echo ${file%.jpg}      # image  (remove .jpg suffix)
echo ${file#*.}        # jpg    (remove up to first dot)

# Replace
greeting="Hello World"
echo ${greeting/World/Bash}    # Hello Bash (first match)
echo ${greeting//l/L}          # HeLLo WorLd (all matches)
```

---

## Arrays (Basics)

> Covered in detail in Lesson 07. Quick intro here:

```bash
fruits=("apple" "banana" "cherry")

echo ${fruits[0]}        # apple
echo ${fruits[@]}        # all elements
echo ${#fruits[@]}       # count: 3
```

---

## Storing Command Output

```bash
today=$(date)              # preferred: $( )
today=`date`               # old style: backticks (avoid)

files=$(ls /etc | wc -l)
echo "Files in /etc: $files"

# Multi-line output
users=$(cat /etc/passwd | cut -d: -f1)
echo "$users"              # quote to preserve newlines
```

---

## Key Takeaways

> - No spaces around `=` when assigning variables
> - Always quote variables with `"$var"` to avoid issues with spaces
> - Use `readonly` for constants that shouldn't change
> - Use `local` inside functions to avoid polluting global scope
> - `export` makes a variable available to child processes
> - `$1`, `$2`, `$#`, `$@` are used to handle script arguments
