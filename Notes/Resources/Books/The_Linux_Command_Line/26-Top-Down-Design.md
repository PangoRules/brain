# Chapter 26 — Top-Down Design (Shell Functions)

**Top-down design** — break a large task into high-level steps, then refine each step into smaller sub-steps. In shell scripts this maps directly to **functions**.

---

## Shell Functions

### Syntax — two equivalent forms

```bash
# Form 1 — formal (bash keyword)
function name {
    commands
    return
}

# Form 2 — preferred (POSIX-compatible, works in bash and zsh)
name () {
    commands
    return
}
```

- `return` is optional but good practice — makes intent clear
- Functions **must be defined before they are called** — shell reads top to bottom
- Function names follow the same rules as variables

### Minimal example

```bash
#!/usr/bin/env bash

greet () {
    echo "Hello, $1"
}

greet "pango"    # → Hello, pango
greet "world"    # → Hello, world
```

---

## Arguments

Functions receive arguments the same way scripts do — positional parameters `$1`, `$2`, etc. `$0` is still the script name, not the function name.

```bash
report_uptime () {
    local host="${1:-$HOSTNAME}"    # use arg or default to $HOSTNAME
    echo "Uptime for $host:"
    uptime
}

report_uptime                    # uses $HOSTNAME
report_uptime "remotehost"       # uses "remotehost"
```

| Parameter | Value                            |
| --------- | -------------------------------- |
| `$1`–`$9` | Positional arguments             |
| `$@`      | All arguments as separate words  |
| `$*`      | All arguments as a single string |
| `$#`      | Number of arguments              |
| `$0`      | Script name (not function name)  |

---

## Local Variables

Without `local`, all variables inside a function are **global** — they pollute the surrounding script and collide with other functions.

```bash
foo=0    # global

funct_a () {
    local foo=1          # scoped to funct_a only
    echo "inside: $foo"  # → 1
}

funct_a
echo "outside: $foo"     # → 0  (global unchanged)
```

**Always use `local` for variables inside functions.** It's the single most important function hygiene rule.

```bash
process_file () {
    local input="$1"
    local output="${2:-/tmp/output.txt}"
    local line_count

    line_count=$(wc -l < "$input")
    echo "Processing $line_count lines from $input"
}
```

---

## Return Values and Exit Codes

`return` sets the function's **exit code** (0 = success, 1–255 = error). It does NOT return a value like in other languages.

```bash
is_root () {
    [[ $EUID -eq 0 ]]   # returns 0 if root, 1 if not (no explicit return needed)
}

if is_root; then
    echo "Running as root"
fi
```

**To "return" a value, use stdout and capture with `$(...)`:**

```bash
get_date () {
    date +"%Y-%m-%d"         # print to stdout
}

today="$(get_date)"          # capture it
echo "Today is $today"
```

```bash
# Return code check
my_func () {
    some_command || return 1   # propagate failure
    return 0
}

if ! my_func; then
    echo "my_func failed" >&2
fi
```

---

## Stubs — Keep Scripts Runnable During Development

Start with empty stub functions so the script runs (and is testable) while you flesh out the logic:

```bash
report_uptime () {
    echo "Function report_uptime executed." >&2   # debug feedback to stderr
    return
}

report_disk_space () {
    echo "Function report_disk_space executed." >&2
    return
}
```

Then replace stub bodies one at a time:

```bash
report_uptime () {
    cat << _EOF_
<h2>System Uptime</h2>
<pre>$(uptime)</pre>
_EOF_
}

report_disk_space () {
    cat << _EOF_
<h2>Disk Space</h2>
<pre>$(df -h)</pre>
_EOF_
}

report_home_space () {
    cat << _EOF_
<h2>Home Space Utilization</h2>
<pre>$(du -sh /home/* 2>/dev/null)</pre>
_EOF_
}
```

---

## Functions in `.zshrc` (Your Setup)

Functions beat aliases for anything more than a trivial command — they support arguments, conditionals, loops, local variables.

```bash
# In ~/.zshrc

# Quick disk usage summary
ds () {
    echo "Disk Space Utilization for $HOSTNAME"
    df -h
}

# Show top N processes by CPU (default 10)
topcpu () {
    local n="${1:-10}"
    ps aux --sort=-%cpu | head -n $(( n + 1 ))
}

# Make a dir and cd into it
mkcd () {
    mkdir -p "$1" && cd "$1"
}

# Extract any archive format
extract () {
    case "$1" in
        *.tar.gz|*.tgz)  tar xzf "$1" ;;
        *.tar.bz2)       tar xjf "$1" ;;
        *.tar.xz)        tar xJf "$1" ;;
        *.tar.zst)       tar --zstd -xf "$1" ;;
        *.zip)           unzip "$1" ;;
        *.gz)            gunzip "$1" ;;
        *.bz2)           bunzip2 "$1" ;;
        *.xz)            unxz "$1" ;;
        *)               echo "Unknown format: $1" >&2; return 1 ;;
    esac
}
```

Reload after editing:
```bash
source ~/.zshrc
```

---

## The Project — sys_info_page (Chapter 26 State)

```bash
#!/usr/bin/env bash
# sys_info_page — HTML system report

set -euo pipefail

TITLE="System Information Report For $HOSTNAME"
CURRENT_TIME="$(date +"%x %r %Z")"
TIMESTAMP="Generated $CURRENT_TIME, by $USER"

report_uptime () {
    cat << _EOF_
<h2>System Uptime</h2>
<pre>$(uptime)</pre>
_EOF_
}

report_disk_space () {
    cat << _EOF_
<h2>Disk Space</h2>
<pre>$(df -h)</pre>
_EOF_
}

report_home_space () {
    cat << _EOF_
<h2>Home Space</h2>
<pre>$(du -sh /home/* 2>/dev/null)</pre>
_EOF_
}

cat << _EOF_
<html>
  <head><title>$TITLE</title></head>
  <body>
    <h1>$TITLE</h1>
    <p>$TIMESTAMP</p>
    $(report_uptime)
    $(report_disk_space)
    $(report_home_space)
  </body>
</html>
_EOF_
```

---

## Quick Reference

```bash
# Define
my_func () {
    local var="$1"
    echo "result"       # "return" a value via stdout
    return 0            # exit code
}

# Call
my_func arg1

# Capture output
result="$(my_func arg1)"

# Check exit code
if my_func arg1; then
    echo "success"
fi

# List defined functions (zsh)
functions

# List defined functions (bash)
declare -F

# Show function body
declare -f my_func      # bash
functions my_func       # zsh
```
