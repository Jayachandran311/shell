
# Shell Scripting — File Operations (Simple & Student‑Friendly)

## ✅ Checking Files & Directories
```bash
[ -e "$path" ] && echo "exists"
[ -f "$path" ] && echo "regular file"
[ -d "$path" ] && echo "directory"
[ -L "$path" ] && echo "symlink"
[ -s "$path" ] && echo "non‑empty"
[ -r "$path" ] && echo "readable"
[ -w "$path" ] && echo "writable"
[ -x "$path" ] && echo "executable"
```

---
## ✅ Reading Files
### Print whole file
```bash
cat file.txt
content=$(cat file.txt)
echo "$content"
```

### Head/tail
```bash
head -n 5 file.txt
tail -n 5 file.txt
tail -f logfile.log
```

### Read line‑by‑line
```bash
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt
```

Skip blanks/comments:
```bash
while IFS= read -r line; do
    [[ -z "$line" || "$line" == \#* ]] && continue
    echo "$line"
done < config.txt
```

CSV/TSV:
```bash
while IFS=',' read -r name age city; do
    echo "$name — $city"
done < data.csv
```

---
## ✅ Writing Files
```bash
echo "Hello" > out.txt      # overwrite
echo "More" >> out.txt     # append
```

Heredoc:
```bash
cat > config.ini << EOF
[server]
port=8080
EOF
```

printf:
```bash
printf "Name: %s
Age: %d
" "Alice" 30 > info.txt
```

---
## ✅ Copying, Moving, Renaming
```bash
cp a.txt b.txt
cp -r src/ backup/
cp -p file.txt saved/
cp -u *.sh scripts/

mv old new
mv file.txt /tmp/

for f in *.jpeg; do mv "$f" "${f%.jpeg}.jpg"; done
```

---
## ✅ Deleting Files
```bash
rm file.txt
rm -f file.txt
rm -r dir/
rm -rf dir/
```

Safe delete:
```bash
[ -f "$file" ] && rm "$file"
```

---
## ✅ Creating Directories
```bash
mkdir mydir
mkdir -p a/b/c
mkdir -m 755 secure_dir
mkdir -p logs/{2024,2025}/{jan,feb,mar}
```

---
## ✅ Finding Files
```bash
find . -name "*.log"
find . -name "*.sh" -type f
find /var/log -size +100M
find . -mtime -1          # modified last 1 day
```

Find + exec:
```bash
find . -name "*.sh" -exec chmod +x {} \;
find . -name "*.tmp" -delete
```

xargs (faster):
```bash
find . -name "*.log" | xargs grep ERROR
```

---
## ✅ Temporary Files
```bash
tmpfile=$(mktemp)
tmpdir=$(mktemp -d)

echo "Data" > "$tmpfile"

cleanup() {
  rm -f "$tmpfile"
  rm -rf "$tmpdir"
}
trap cleanup EXIT
```

---
## ✅ File Path Operations
```bash
filepath="/home/alice/report.pdf"

dirname "$filepath"      # /home/alice
basename "$filepath"     # report.pdf
basename "$filepath" .pdf   # report
```

Using bash only:
```bash
echo "${filepath%/*}"     # dirname
echo "${filepath##*/}"    # basename
echo "${filepath##*.}"    # ext
echo "${filepath%.*}"     # remove ext
```

---
## ✅ Archiving Files
```bash
tar -czf archive.tar.gz folder/
tar -xzf archive.tar.gz
zip -r out.zip folder/
unzip out.zip
```

---
## ✅ Practical Example — Backup
```bash
SOURCE="$HOME"
BACKUP="backup_$(date +%F).tar.gz"

tar -czf "$BACKUP" "$SOURCE"
echo "Created: $BACKUP"
```

---
## ✅ Practical Example — Log Cleanup
```bash
find /var/log/myapp -name "*.log" -mtime +30 -delete
```

---
## ✅ Key Takeaways
- Always quote filenames: `"$file"`
- Use `while IFS= read -r` to read files safely
- Use `mktemp` for temporary files + `trap` for cleanup
- `find` + `xargs` is fast for bulk operations
- `mkdir -p` avoids errors when directories already exist
