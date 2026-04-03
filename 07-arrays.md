
# Shell Scripting — Arrays (Simple & Student‑Friendly)

## ✅ Indexed Arrays
```bash
fruits=("apple" "banana" "cherry" "date")

# Declare then assign
declare -a colors
colors[0]="red"
colors[1]="green"
colors[2]="blue"

# From command output (be careful with spaces)
files=($(ls *.sh))
```

---
## ✅ Accessing Elements
```bash
echo ${fruits[0]}      # apple
echo ${fruits[-1]}     # last element (bash 4.2+)
echo ${fruits[@]}      # all elements
echo ${#fruits[@]}     # number of items
echo ${!fruits[@]}     # indices
```

---
## ✅ "$@" vs "$*"
```bash
arr=("hello world" "foo" "bar")

for x in "${arr[@]}"; do echo "$x"; done  # correct
```
✅ Preserves spaces.

```bash
for x in "${arr[*]}"; do echo "$x"; done  # wrong
```
⚠️ Merges all into a single string.

---
## ✅ Modifying Arrays
```bash
fruits+=("mango")        # append
fruits+=("grape" "kiwi") # append multiple

fruits[1]="blueberry"    # update

unset fruits[2]           # remove element (leaves gap)

fruits=("${fruits[@]}")  # reindex (remove gaps)
```

---
## ✅ Array Slicing
```bash
arr=(a b c d e f)

echo ${arr[@]:2}       # c d e f
echo ${arr[@]:1:3}     # b c d
echo ${arr[@]: -2}     # e f

subset=("${arr[@]:1:3}")
```

---
## ✅ Array Operations
### Join array
```bash
fruits=(apple banana cherry)
joined=$(IFS=','; echo "${fruits[*]}")
```

### Split string → array
```bash
csv="one,two,three"
IFS=',' read -ra parts <<< "$csv"
```

### Merge arrays
```bash
merged=("${arr1[@]}" "${arr2[@]}")
```

### Sort array
```bash
sorted=($(printf "%s
" "${fruits[@]}" | sort))
```

---
## ✅ Associative Arrays (Dictionaries)
```bash
declare -A person
person[name]="Alice"
person[city]="Chennai"
person[age]=30

echo ${person[name]}
echo ${!person[@]}   # keys
echo ${person[@]}    # values
```

Loop through:
```bash
for k in "${!person[@]}"; do
    echo "$k = ${person[$k]}"
done
```

---
## ✅ Practical Examples
### Process server list
```bash
SERVERS=(web01 web02 db01)
failed=()

for s in "${SERVERS[@]}"; do
    if ping -c1 "$s" &>/dev/null; then
        echo "✓ $s"
    else
        failed+=("$s")
    fi
done
```

### Read CSV file
```bash
while IFS=',' read -r name email; do
    names+=("$name")
    emails+=("$email")
done < users.csv
```

### Deduplicate array
```bash
items=(apple banana apple cherry banana)
declare -A seen
unique=()

for item in "${items[@]}"; do
    if [[ -z ${seen[$item]+x} ]]; then
        seen[$item]=1
        unique+=("$item")
    fi
done
```

---
## ✅ Key Takeaways
- Indexed arrays: `arr=(a b c)`
- Associative arrays: `declare -A map`
- Iterate safely using: `"${arr[@]}"`
- Count: `${#arr[@]}`, indices: `${!arr[@]}`
- Append: `arr+=("value")`
- Split using: `IFS=',' read -ra arr <<< "$str"`
