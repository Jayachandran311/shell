# Shell Scripting — Arrays

## Indexed Arrays

```bash
# Declare and initialize
fruits=("apple" "banana" "cherry" "date")

# Or declare first, then assign
declare -a colors
colors[0]="red"
colors[1]="green"
colors[2]="blue"

# Or from command output
files=($(ls *.sh))      # be careful with spaces in filenames
files=($(find . -name "*.sh"))
```

---

## Accessing Array Elements

```bash
fruits=("apple" "banana" "cherry" "date")

echo ${fruits[0]}        # apple     (first element)
echo ${fruits[1]}        # banana
echo ${fruits[-1]}       # date      (last element, bash 4.2+)
echo ${fruits[@]}        # apple banana cherry date (ALL elements)
echo ${fruits[*]}        # apple banana cherry date (ALL, as one word)
echo ${#fruits[@]}       # 4         (length/count)
echo ${!fruits[@]}       # 0 1 2 3   (indices)
```

---

## Difference: `$@` vs `$*`

```bash
arr=("hello world" "foo" "bar")

for item in "${arr[@]}"; do    # correct — preserves elements with spaces
    echo "Item: $item"
done
# Item: hello world
# Item: foo
# Item: bar

for item in "${arr[*]}"; do    # wrong — merges all into one string
    echo "Item: $item"
done
# Item: hello world foo bar
```

> **Rule:** Always use `"${array[@]}"` (not `*`) when iterating.

---

## Modifying Arrays

```bash
fruits=("apple" "banana" "cherry")

# Append element
fruits+=("mango")
fruits+=("grape" "kiwi")    # append multiple

# Update element
fruits[1]="blueberry"

# Remove element (creates a gap — index still exists as empty)
unset fruits[2]

# Remove entire array
unset fruits

# Re-index after removal (no gaps)
fruits=("${fruits[@]}")     # re-create without gaps
```

---

## Array Slicing

```bash
arr=("a" "b" "c" "d" "e" "f")

echo ${arr[@]:2}        # c d e f       (from index 2 to end)
echo ${arr[@]:1:3}      # b c d         (3 elements starting at index 1)
echo ${arr[@]: -2}      # e f           (last 2 elements)

# Copy a portion
subset=("${arr[@]:1:3}")
echo ${subset[@]}       # b c d
```

---

## Array Operations

```bash
# Join array into string
fruits=("apple" "banana" "cherry")
joined=$(IFS=','; echo "${fruits[*]}")
echo $joined    # apple,banana,cherry

# Split string into array
csv="one,two,three,four"
IFS=',' read -ra parts <<< "$csv"
echo ${parts[0]}    # one
echo ${parts[2]}    # three

# Merge two arrays
arr1=("a" "b" "c")
arr2=("d" "e" "f")
merged=("${arr1[@]}" "${arr2[@]}")
echo ${merged[@]}   # a b c d e f

# Sort (need a workaround — bash has no built-in sort)
sorted=($(for item in "${fruits[@]}"; do echo "$item"; done | sort))
```

---

## Associative Arrays (Dictionaries)

Require `declare -A` (bash 4+):

```bash
declare -A person
person["name"]="Alice"
person["age"]=30
person["city"]="Chennai"

echo ${person["name"]}    # Alice
echo ${person[@]}         # all values
echo ${!person[@]}        # all keys

# Initialize inline
declare -A config=(
    ["host"]="localhost"
    ["port"]="5432"
    ["db"]="mydb"
)

# Iterate
for key in "${!config[@]}"; do
    echo "$key = ${config[$key]}"
done
```

---

## Practical Array Examples

### Process a list of servers

```bash
#!/usr/bin/env bash
declare -a SERVERS=("web01" "web02" "db01" "cache01")

check_server() {
    local server=$1
    if ping -c 1 -W 2 "$server" &>/dev/null; then
        echo "✓ $server"
        return 0
    else
        echo "✗ $server — unreachable"
        return 1
    fi
}

failed=()
for server in "${SERVERS[@]}"; do
    check_server "$server" || failed+=("$server")
done

if (( ${#failed[@]} > 0 )); then
    echo ""
    echo "Failed servers: ${failed[*]}"
    exit 1
fi
```

### Parse a CSV file into array

```bash
#!/usr/bin/env bash
declare -a names
declare -a emails

while IFS=',' read -r name email; do
    names+=("$name")
    emails+=("$email")
done < users.csv

for i in "${!names[@]}"; do
    echo "Name: ${names[$i]}, Email: ${emails[$i]}"
done
```

### Deduplication

```bash
items=("apple" "banana" "apple" "cherry" "banana")
declare -A seen
unique=()

for item in "${items[@]}"; do
    if [[ -z "${seen[$item]+set}" ]]; then
        seen[$item]=1
        unique+=("$item")
    fi
done

echo "Unique: ${unique[@]}"   # apple banana cherry
```

---

## Key Takeaways

> - Declare indexed arrays: `arr=(val1 val2 val3)`
> - Declare associative arrays: `declare -A map`
> - Always quote: `"${arr[@]}"` for iteration, not bare `$arr`
> - `${#arr[@]}` = count, `${!arr[@]}` = list of indices/keys
> - `arr+=("new")` appends an element
> - Use `IFS=',' read -ra arr <<< "$str"` to split a string into an array
> - `unset arr[i]` removes an element but leaves a gap in indices
