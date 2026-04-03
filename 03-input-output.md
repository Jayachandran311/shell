
# Shell Scripting — Input & Output (Simple & Student‑Friendly)

## ✅ Output with `echo`
```bash
echo "Hello, World!"
echo -n "No newline"      # no newline
echo -e "Line1
Line2"   # enable escapes
```

Common escapes:
- `
` → newline
- `	` → tab
- Colors using ANSI codes:
```bash
RED='[0;31m'
GREEN='[0;32m'
RESET='[0m'

echo -e "${GREEN}Success${RESET}"
```

---

## ✅ Output with `printf`
```bash
printf "Hello %s!
" "Alice"
printf "Number: %d
" 42
printf "Float: %.2f
" 3.14159
printf "%-10s | %d
" "Alice" 30
```

✅ More predictable than `echo`.

---

## ✅ Reading Input (`read`)
```bash
read name
read -p "Enter name: " name
read -sp "Password: " pass
read -t 5 -p "Timeout 5s: " ans
```

Multiple variables:
```bash
read -p "First Last: " first last
```

Read an array:
```bash
read -a fruits <<< "apple banana"
```

Read file line-by-line:
```bash
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/hosts
```

---

## ✅ stdin, stdout, stderr
| Stream | FD | Description |
|--------|----|-------------|
| stdin  | 0  | input (keyboard) |
| stdout | 1  | normal output |
| stderr | 2  | error output |

```bash
echo "OK"        # stdout
echo "ERR" >&2   # stderr
```

---

## ✅ Redirection
```bash
echo "Hello" > file.txt      # overwrite
echo "More" >> file.txt       # append
sort < names.txt              # stdin
ls /fake 2> errors.txt        # stderr only
command > out.txt 2>&1        # both
command &> out.txt            # shorthand
command > /dev/null           # discard output
```

---

## ✅ Redirection Examples
```bash
./backup.sh > backup.log 2>&1
tool | tee output.log
sort <<< "banana apple"
command > ok.log 2> err.log
```

---

## ✅ Here Documents (Heredocs)
```bash
cat << EOF
Line1
Line2
EOF
```

No variable expansion:
```bash
cat << 'EOF'
$HOME
EOF
```

Write config file:
```bash
cat > config.conf << EOF
host=localhost
user=$USER
EOF
```

---

## ✅ Pipes (`|`)
```bash
ls -la | grep ".sh"
cat /etc/passwd | cut -d: -f1
ps aux | grep nginx | grep -v grep
```

Count:
```bash
ls | wc -l
```

Make pipes fail properly:
```bash
set -o pipefail
```

---

## ✅ `tee` — Split Output
```bash
ls -la | tee list.txt
command | tee -a log.txt
command 2>&1 | tee all.log
```

---

## ✅ Process Substitution
```bash
diff <(ls dir1) <(ls dir2)

while read line; do
    echo "Got: $line"
done < <(ls /etc)
```

---

## ✅ Key Takeaways
- Use `echo -e` for escapes and colors
- Use `printf` for formatted output
- Use `read -p` for prompts, `read -sp` for passwords
- `>` overwrites, `>>` appends, `2>` redirects errors
- `/dev/null` discards output
- Pipes (`|`) chain commands
- Use `set -o pipefail` to catch pipe errors
