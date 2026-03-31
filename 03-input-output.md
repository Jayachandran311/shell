# Shell Scripting — Input & Output

## Output with `echo`

```bash
echo "Hello, World!"
echo -n "No newline at end"    # -n suppresses trailing newline
echo -e "Line1\nLine2\tTabbed" # -e enables escape sequences

# Escape sequences with -e
echo -e "\n"      # newline
echo -e "\t"      # tab
echo -e "\033[31mRed Text\033[0m"    # colored output
```

### Colors (ANSI escape codes)

```bash
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
BOLD='\033[1m'
RESET='\033[0m'

echo -e "${GREEN}Success!${RESET}"
echo -e "${RED}Error!${RESET}"
echo -e "${YELLOW}Warning!${RESET}"
```

---

## Output with `printf`

More precise formatting than `echo` — similar to C's `printf`.

```bash
printf "Hello, %s!\n" "Alice"        # Hello, Alice!
printf "Number: %d\n" 42             # Number: 42
printf "Float: %.2f\n" 3.14159       # Float: 3.14
printf "Padded: %10s\n" "right"      #      right
printf "Left:   %-10s|\n" "left"     # left      |

# Multiple values
printf "Name: %-15s Age: %d\n" "Alice" 30
printf "Name: %-15s Age: %d\n" "Bob" 25
```

---

## Reading Input with `read`

```bash
# Basic read
read name
echo "Hello, $name"

# With prompt
read -p "Enter your name: " name
echo "Hello, $name"

# Silent input (for passwords)
read -sp "Enter password: " pass
echo ""   # print newline after silent input
echo "Password length: ${#pass}"

# Read with timeout (seconds)
read -t 5 -p "Quick! Enter something (5 sec timeout): " answer

# Read into multiple variables
read -p "Enter first and last name: " first last
echo "First: $first, Last: $last"

# Read array
read -a fruits <<< "apple banana cherry"
echo ${fruits[0]}   # apple

# Read a file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/hosts
```

---

## stdin, stdout, stderr

Every process has 3 standard data streams:

| Stream | FD | Default | Symbol |
|--------|-----|---------|--------|
| stdin  | 0  | keyboard | `<` |
| stdout | 1  | terminal | `>` or `1>` |
| stderr | 2  | terminal | `2>` |

```bash
echo "Normal output"    # goes to stdout
echo "Error!" >&2       # redirect to stderr

ls /fakedir             # ls sends error to stderr
ls /etc                 # ls sends listing to stdout
```

---

## Redirection

```bash
# Redirect stdout to file (overwrites)
echo "Hello" > output.txt
ls -la > filelist.txt

# Append stdout to file
echo "More text" >> output.txt

# Redirect stdin from file
sort < names.txt

# Redirect stderr to file
ls /fakedir 2> errors.txt

# Redirect both stdout and stderr to file
command > output.txt 2>&1
command &> output.txt      # shorthand (bash only)

# Discard output (sends to /dev/null)
command > /dev/null         # discard stdout
command 2> /dev/null        # discard stderr
command &> /dev/null        # discard everything

# Redirect stdout to stderr
echo "This is an error" >&2
```

### Redirection Examples

```bash
# Log all output to a file
./backup.sh > backup.log 2>&1

# Show output AND save to file
./script.sh 2>&1 | tee output.log

# Send input from a string
sort <<< "banana apple cherry"

# Separate stdout and stderr
command > success.log 2> error.log
```

---

## Here Documents (heredoc)

Multi-line input to a command:

```bash
cat << EOF
This is line 1
This is line 2
Variable: $HOME
EOF

# Indent-friendly (strip leading tabs with <<-)
cat <<- EOF
    This is line 1
    This is line 2
EOF

# No variable expansion (single-quote the delimiter)
cat << 'EOF'
Variable: $HOME     ← this prints literally, no expansion
EOF
```

Practical use:
```bash
# Write a config file
cat > /tmp/config.conf << EOF
[server]
host = localhost
port = 8080
user = $USER
EOF
```

---

## Pipes (`|`)

Connect stdout of one command to stdin of the next:

```bash
command1 | command2 | command3

# Examples
ls -la | grep ".sh"                  # filter .sh files
cat /etc/passwd | cut -d: -f1        # extract usernames
ps aux | grep nginx | grep -v grep   # find nginx processes
cat access.log | sort | uniq -c | sort -rn | head -10   # top IPs

# Count lines
ls | wc -l
cat file.txt | wc -l

# With error handling (pipefail)
set -o pipefail    # fail if any pipe stage fails
```

---

## tee — Split Output

Writes to both stdout and a file:

```bash
ls -la | tee filelist.txt          # display and save
command | tee -a logfile.txt       # append mode
command 2>&1 | tee output.log      # capture all
```

---

## Process Substitution

Treat command output as a file:

```bash
# Compare two commands' output
diff <(ls dir1) <(ls dir2)

# Use command output as input file
while read line; do
    echo "Got: $line"
done < <(ls /etc)
```

---

## Key Takeaways

> - Use `echo -e` for escape sequences (newlines, colors)
> - Use `printf` for precise formatted output
> - Use `read -p` for user input, `read -sp` for passwords
> - `>` overwrites, `>>` appends, `2>` redirects stderr
> - `&>` redirects both stdout and stderr (bash shorthand)
> - `/dev/null` is a black hole — use to suppress unwanted output
> - Pipes `|` chain commands; use `set -o pipefail` to catch pipe errors
