
# Shell Scripting — Simple Introduction

## ✅ What is a Shell?
A **shell** is a program that lets you type commands to control Linux.
A **shell script** is a text file with commands that run one after another.

---

## ✅ Types of Shells
| Shell | Meaning | Notes |
|------|---------|-------|
| sh | Bourne Shell | Basic POSIX shell |
| bash | Bourne Again Shell | Most common on Linux |
| zsh | Z Shell | Default on macOS |
| ksh | Korn Shell | Used in enterprises |
| fish | Friendly Interactive Shell | User-friendly, not POSIX |
| dash | Debian Almquist Shell | Very fast, used for /bin/sh |

Check your shell:
```bash
echo $SHELL
echo $0
cat /etc/shells
```

---

## ✅ The Shebang (#!)
The first line of a script tells Linux which shell to use.

```bash
#!/usr/bin/env bash
```

Other examples:
```bash
#!/bin/bash
#!/bin/sh
#!/usr/bin/env python3
```

---

## ✅ Your First Script
Create a file **hello.sh**:

```bash
#!/usr/bin/env bash

echo "Hello, World!"
echo "Today is: $(date)"
echo "You are: $(whoami)"
```

---

## ✅ Running the Script
```bash
chmod +x hello.sh
./hello.sh
```

Other ways:
```bash
bash hello.sh
sh hello.sh
```

Source (runs in current shell):
```bash
source hello.sh
. hello.sh
```

---

## ✅ Comments
```bash
# This is a comment
echo "Hello"  # inline comment
# Multi-line comments → multiple # lines
```

---

## ✅ Exit Codes
- 0 → success
- non-zero → error

Show exit code:
```bash
echo $?
```

Custom exit:
```bash
exit 0
exit 1
exit 2
```

---

## ✅ Debugging Scripts
```bash
bash -x script.sh
bash -n script.sh
```

Inside script:
```bash
set -x
set +x
```

Recommended safety:
```bash
set -euo pipefail
```

---

## ✅ Simple Script Template
```bash
#!/usr/bin/env bash
set -euo pipefail

log() {
    echo "[INFO] $*"
}

main() {
    log "Script started"
    # your code here
    log "Script finished"
}

main "$@"
```

---

## ✅ Summary
- Use `#!/usr/bin/env bash`
- Run with `chmod +x`
- Debug using `bash -x`
- Use `set -euo pipefail` for safe scripts
- Exit code 0 = success
