# Shell Scripting — File Operations

## Checking Files and Directories

```bash
# Does it exist?
if [ -e "$path" ]; then echo "exists"; fi
if [ -f "$path" ]; then echo "is a regular file"; fi
if [ -d "$path" ]; then echo "is a directory"; fi
if [ -L "$path" ]; then echo "is a symbolic link"; fi
if [ -s "$path" ]; then echo "file is non-empty"; fi
if [ -r "$path" ]; then echo "readable"; fi
if [ -w "$path" ]; then echo "writable"; fi
if [ -x "$path" ]; then echo "executable"; fi
```

---

## Reading Files

### Read entire file

```bash
# Print contents
cat file.txt

# Store in variable
content=$(cat file.txt)
echo "$content"

# Read with head/tail
head -n 5 file.txt      # first 5 lines
tail -n 5 file.txt      # last 5 lines
tail -f logfile.log     # follow (watch live updates)
```

### Read file line by line (best practice)

```bash
while IFS= read -r line; do
    echo "Line: $line"
done < "file.txt"

# Skip empty lines and comments
while IFS= read -r line; do
    [[ -z "$line" || "$line" == \#* ]] && continue
    echo "Processing: $line"
done < "config.txt"

# With line numbers
lineno=0
while IFS= read -r line; do
    (( lineno++ ))
    echo "$lineno: $line"
done < "file.txt"
```

### Read fields (CSV/TSV)

```bash
# Colon-separated (like /etc/passwd)
while IFS=: read -r user pass uid gid comment home shell; do
    echo "User: $user, Home: $home"
done < /etc/passwd

# CSV
while IFS=',' read -r name age city; do
    echo "Name: $name, Age: $age, City: $city"
done < data.csv
```

---

## Writing Files

```bash
# Overwrite (or create)
echo "Hello, World" > output.txt

# Append
echo "Another line" >> output.txt

# Multiple lines with heredoc
cat > config.txt << EOF
[database]
host = localhost
port = 5432
name = mydb
EOF

# Append with heredoc
cat >> config.txt << EOF
[server]
port = 8080
EOF

# Write with printf
printf "Name: %s\nAge: %d\n" "Alice" 30 > info.txt
```

---

## Copying, Moving, Renaming Files

```bash
cp source.txt dest.txt          # copy file
cp -r source_dir/ dest_dir/     # copy directory recursively
cp -p file.txt backup/          # preserve permissions & timestamps
cp -u *.sh scripts/             # copy only if newer

mv old.txt new.txt              # rename
mv file.txt /tmp/               # move to directory
mv *.log /var/log/myapp/        # move multiple files

# Safe rename using variable
for file in *.jpeg; do
    mv "$file" "${file%.jpeg}.jpg"
done
```

---

## Deleting Files

```bash
rm file.txt                     # delete file
rm -f file.txt                  # force delete (no error if missing)
rm -r directory/                # delete directory recursively
rm -rf directory/               # force-delete entire directory tree

# Safe deletion — check first
if [ -f "$file" ]; then
    rm "$file"
    echo "Deleted $file"
fi

# Use trash instead of rm (safer)
# alias rm='trash'  (if trash-cli is installed)
```

---

## Creating Directories

```bash
mkdir mydir                     # create directory
mkdir -p path/to/nested/dir     # create with all parents
mkdir -m 755 mydir              # with specific permissions
mkdir -p logs/{2024,2025}/{jan,feb,mar}  # create multiple
```

---

## Finding Files

```bash
# Basic find
find /path -name "*.log"                    # by name
find /path -name "*.sh" -type f             # files only
find /path -type d                          # directories only
find . -name "*.tmp" -newer reference.txt   # newer than a file

# Find and execute
find . -name "*.sh" -exec chmod +x {} \;
find /tmp -name "*.tmp" -delete             # delete found files
find . -type f -exec md5sum {} \;

# Find by size
find /var/log -size +100M                   # larger than 100MB
find /var/log -size +10k -size -100k        # between 10k and 100k

# Find by modification time
find /etc -mtime -1           # modified in last 1 day
find /var/log -mtime +30      # not modified in 30+ days
find . -newer file.txt        # newer than file.txt

# Find and process with xargs (faster than -exec for many files)
find . -name "*.log" | xargs grep "ERROR"
find . -name "*.sh" | xargs chmod +x
find /tmp -name "*.tmp" | xargs rm -f
```

---

## Temporary Files

```bash
# Create unique temp file
tmpfile=$(mktemp)
tmpdir=$(mktemp -d)

echo "Using tempfile: $tmpfile"
echo "Data to process" > "$tmpfile"

# Clean up temp files on exit (using trap)
cleanup() {
    rm -f "$tmpfile"
    rm -rf "$tmpdir"
}
trap cleanup EXIT    # runs on script exit (normal or error)

# Processing
process_file "$tmpfile"
```

---

## File Path Operations

```bash
filepath="/home/alice/documents/report.pdf"

dirname  "$filepath"     # /home/alice/documents
basename "$filepath"     # report.pdf
basename "$filepath" .pdf  # report  (strip extension)

# Pure bash parameter expansion
echo "${filepath%/*}"    # /home/alice/documents  (dirname)
echo "${filepath##*/}"   # report.pdf              (basename)
echo "${filepath##*.}"   # pdf                     (extension)
echo "${filepath%.*}"    # /home/alice/documents/report  (no ext)
```

---

## Archiving Files

```bash
# Create tar archive
tar -czf archive.tar.gz directory/         # gzip compressed
tar -cjf archive.tar.bz2 directory/        # bzip2 compressed
tar -caf archive.tar.xz directory/         # xz compressed

# Extract
tar -xzf archive.tar.gz                    # extract gzip
tar -xzf archive.tar.gz -C /target/dir    # extract to specific dir
tar -tzf archive.tar.gz                    # list contents

# Zip
zip -r output.zip directory/
unzip archive.zip
unzip archive.zip -d /target/dir
```

---

## Practical Examples

### Backup script

```bash
#!/usr/bin/env bash
set -euo pipefail

SOURCE="${1:-$HOME}"
BACKUP_DIR="/backup"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
OUTFILE="$BACKUP_DIR/backup_$TIMESTAMP.tar.gz"

mkdir -p "$BACKUP_DIR"

echo "Backing up $SOURCE → $OUTFILE"
tar -czf "$OUTFILE" "$SOURCE"
echo "Done. Size: $(du -sh "$OUTFILE" | cut -f1)"
```

### Log cleanup script

```bash
#!/usr/bin/env bash
LOG_DIR="/var/log/myapp"
DAYS_TO_KEEP=30

echo "Cleaning logs older than $DAYS_TO_KEEP days..."
find "$LOG_DIR" -name "*.log" -mtime +$DAYS_TO_KEEP -delete
echo "Files remaining: $(ls "$LOG_DIR" | wc -l)"
```

---

## Key Takeaways

> - Always quote file variables: `"$file"` — filenames can contain spaces
> - Use `while IFS= read -r line; do ... done < file` for reliable file reading
> - Use `mktemp` for temp files; always clean them up with `trap cleanup EXIT`
> - `find . -name "*.log" | xargs rm` is faster than `-exec rm {} \;` for many files
> - Use `mkdir -p` to create nested directories without error if they exist
> - `cp -r` for directories, `cp -p` to preserve timestamps/permissions
