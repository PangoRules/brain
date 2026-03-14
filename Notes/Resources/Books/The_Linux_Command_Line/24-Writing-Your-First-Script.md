# Chapter 24 — Writing Your First Script

## The Anatomy of a Shell Script

```bash
#!/bin/bash
# This is a comment — everything after # is ignored

echo 'Hello World!'
```

Three required things to run a script:
1. **Write it** — plain text file with commands
2. **Make it executable** — `chmod 755 script.sh`
3. **Put it where the shell can find it** — in a directory on `$PATH`

---

## Shebang (`#!`)

The first line tells the kernel which interpreter to use. Without it, the shell tries to run the file as commands in the current shell — which works but is fragile.

```bash
#!/bin/bash          # explicit bash
#!/bin/zsh           # explicit zsh (your shell)
#!/usr/bin/env bash  # portable: finds bash wherever it lives
#!/usr/bin/env zsh   # portable: finds zsh wherever it lives
#!/usr/bin/env python3
#!/usr/bin/env node
```

> **You use zsh**, not bash. The book uses `#!/bin/bash` throughout. For your own scripts, use `#!/bin/zsh` or `#!/usr/bin/env bash` (bash is more portable across systems).
>
> Most POSIX-compatible bash scripts run fine under zsh. If you're writing scripts meant to run on other systems, stick with `#!/bin/bash`.

---

## Permissions

```bash
chmod 755 script.sh     # rwxr-xr-x — owner + everyone can execute
chmod 700 script.sh     # rwx------ — only owner can execute (private scripts)
chmod +x script.sh      # just add execute bit (preserves other perms)
```

Scripts must also be **readable** to be executed — the interpreter reads the file.

---

## Where to Put Scripts

Your `~/.local/bin` is already in your `$PATH` — drop scripts there and they're immediately available by name.

```bash
# Your current setup:
# ~/.local/bin is in PATH (confirmed in ~/.zshrc)
# That's where aider, claude, uv etc. already live

# For personal scripts:
cp myscript.sh ~/.local/bin/myscript
chmod +x ~/.local/bin/myscript
myscript       # works from anywhere

# For scripts shared with all users on the system:
sudo cp myscript.sh /usr/local/bin/myscript
sudo chmod 755 /usr/local/bin/myscript
```

**Location conventions:**

| Location           | Use case                                  |
| ------------------ | ----------------------------------------- |
| `~/.local/bin`     | Personal scripts (your setup — use this)  |
| `~/bin`            | Alternative personal location             |
| `/usr/local/bin`   | System-wide, manually installed scripts   |
| `/usr/local/sbin`  | System admin scripts (need root)          |

> Never put your scripts in `/bin` or `/usr/bin` — those belong to the distro package manager (Fedora/dnf).

**Check what's on your PATH:**
```bash
echo $PATH
echo $PATH | tr ':' '\n'    # one directory per line — easier to read
```

**Run a script not on PATH:**
```bash
./script.sh          # explicit relative path
/full/path/script.sh # explicit absolute path
bash script.sh       # run with explicit interpreter (ignores shebang, no +x needed)
```

---

## Sourcing vs. Executing

```bash
./script.sh          # executes in a subshell — variables/cd don't affect current shell
source script.sh     # runs in current shell — changes DO affect current session
. script.sh          # same as source (POSIX shorthand)
```

Use `source` / `.` when a script needs to modify the current environment (e.g., activate a venv, set env vars, update PATH).

---

## Reloading Shell Config

After editing `~/.zshrc`:
```bash
source ~/.zshrc
# or
. ~/.zshrc
```

---

## Formatting for Readability

### Long options in scripts

Short options are fine on the command line. In scripts, prefer long options — they're self-documenting:

```bash
# Command line style
ls -adl

# Script style — clearer what each flag does
ls --all --directory -l
rm --recursive --force --verbose
```

### Line continuation

Break long commands across lines with `\` (backslash at end of line, no trailing space):

```bash
find ~/projects \
    -type f \
    -name "*.log" \
    -mtime +30 \
    -exec rm '{}' '+'

# Flags can be grouped for clarity too
curl \
    --silent \
    --location \
    --output file.tar.gz \
    "https://example.com/file.tar.gz"
```

### Comments

```bash
#!/bin/bash
# Script: backup.sh
# Purpose: Backs up home directory to external drive
# Usage: backup.sh [destination]

# --- Configuration ---
SOURCE="$HOME"
DEST="${1:-/mnt/backup}"   # use first arg, or default

rsync -av --delete "$SOURCE" "$DEST"   # trailing comment OK too
```

---

## Setting Up Your Editor

### vim (for scripting)

Add to `~/.vimrc` (without leading `:`):

```vim
syntax on           " syntax highlighting
set hlsearch        " highlight search matches
set tabstop=4       " tab = 4 spaces wide
set shiftwidth=4    " indent size
set expandtab       " use spaces instead of tabs
set autoindent      " auto-indent new lines
set number          " show line numbers
```

Enable per-session:
```
:syntax on
:set number
:set tabstop=4
```

### Kate (your KDE editor — already configured for syntax highlighting)

Kate auto-detects shell scripts and highlights them. For scripting:
- **Settings → Configure Kate → Editing** — set indent width, use spaces
- The `.kate-swp` file in your Calibre notes dir means Kate is already your go-to

### shellcheck — lint your scripts

Essential tool. Catches bugs, bad practices, and portability issues before you run anything:

```bash
sudo dnf install ShellCheck

shellcheck script.sh
shellcheck -s bash script.sh    # lint as bash specifically
shellcheck -s sh script.sh      # lint as POSIX sh (most portable)
```

---

## A Minimal Useful Script Template

```bash
#!/usr/bin/env bash
# script-name — one-line description
# Usage: script-name [options] args

set -euo pipefail   # exit on error, undefined vars, pipe failures — always add this

# Constants
readonly SCRIPT_NAME="$(basename "$0")"

# Functions
usage() {
    echo "Usage: $SCRIPT_NAME [options]"
    exit 0
}

main() {
    echo "Hello from $SCRIPT_NAME"
}

main "$@"
```

**`set -euo pipefail` explained:**
- `-e` — exit immediately if any command fails
- `-u` — error on undefined variables (catches typos in var names)
- `-o pipefail` — a pipe fails if any command in it fails (not just the last)

---

## Quick Workflow

```bash
# 1. Create script
vim ~/.local/bin/myscript

# 2. Add shebang + content, save

# 3. Make executable
chmod +x ~/.local/bin/myscript

# 4. Lint it
shellcheck ~/.local/bin/myscript

# 5. Run it
myscript
```
