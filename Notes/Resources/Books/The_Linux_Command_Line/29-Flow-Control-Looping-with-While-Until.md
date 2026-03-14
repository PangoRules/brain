# Chapter 29 – Flow Control: Looping with `while` / `until`

## Quick Reference

| Construct | Behavior |
|-----------|----------|
| `while cmd; do ... done` | Loop while cmd exits 0 |
| `until cmd; do ... done` | Loop until cmd exits 0 |
| `while true; do ... done` | Infinite loop |
| `break` | Exit loop immediately |
| `continue` | Skip to next iteration |
| `done < file` | Feed file into loop via stdin |
| `cmd \| while read ...; done` | Pipe into loop (vars lost after — subshell) |

---

## `while`

```bash
while commands; do
    commands
done
```

Evaluates exit status of `commands`. Loops as long as exit status is **zero**.

```bash
count=1
while [[ "$count" -le 5 ]]; do
    echo "$count"
    (( count++ ))
done
```

---

## `until`

```bash
until commands; do
    commands
done
```

Opposite of `while` — loops until exit status is **zero** (stops when condition becomes true).

```bash
count=1
until [[ "$count" -gt 5 ]]; do
    echo "$count"
    (( count++ ))
done
```

Use whichever reads more naturally for the condition you're testing.

---

## Infinite Loop with `break`

```bash
while true; do
    read -p "Enter selection [0-3]: "
    if [[ "$REPLY" == 0 ]]; then
        break      # exit loop
    elif [[ "$REPLY" =~ ^[1-3]$ ]]; then
        # handle selection
        continue   # skip to next iteration (skip remaining code below)
    else
        echo "Invalid entry."
    fi
done
```

- `while true` — `true` always exits 0, so the loop never exits on its own
- `break` — immediately exits the loop; execution resumes after `done`
- `continue` — skips the rest of the current iteration; jumps back to the test

---

## Menu Pattern (practical template)

```bash
#!/bin/bash

DELAY=2

while true; do
    clear
    cat <<- _EOF_
        Please Select:
          1. System info
          2. Disk space
          3. Home usage
          0. Quit
    _EOF_

    read -p "Enter selection [0-3]: "

    if [[ ! "$REPLY" =~ ^[0-3]$ ]]; then
        echo "Invalid entry."; sleep "$DELAY"; continue
    fi

    case "$REPLY" in
        0) break ;;
        1) echo "Host: $HOSTNAME"; uptime; sleep "$DELAY" ;;
        2) df -h; sleep "$DELAY" ;;
        3) if (( $(id -u) == 0 )); then
               du -sh /home/* 2>/dev/null
           else
               du -sh "$HOME"
           fi
           sleep "$DELAY" ;;
    esac
done

echo "Program terminated."
```

---

## Reading Files with Loops

### Redirect file into loop (canonical pattern)

```bash
while IFS= read -r line; do
    echo "$line"
done < /etc/hosts
```

- `IFS=` — prevents leading/trailing whitespace stripping
- `-r` — raw mode, backslashes treated literally
- Redirection goes **after `done`**, not at the `while` line

### Multi-field parsing

```bash
while IFS=":" read user pw uid gid name home shell; do
    echo "$user → $shell"
done < /etc/passwd
```

### Piping into a loop

```bash
sort /etc/passwd | while IFS=":" read user pw uid gid name home shell; do
    echo "$user"
done
```

> **Subshell warning**: In bash, the loop body in a pipeline runs in a subshell. Any variables set inside the loop **are lost** after `done`. Use `done < <(cmd)` (process substitution) instead to avoid the subshell:
> ```bash
> while IFS=":" read user pw uid gid name home shell; do
>     echo "$user"
> done < <(sort /etc/passwd)
> ```
> zsh doesn't have this problem (last pipe segment runs in current shell), but the process substitution form works everywhere.

---

## Practical Patterns

```bash
# Wait for a process to finish
while pgrep -x "rsync" > /dev/null; do
    echo "rsync still running..."
    sleep 5
done
echo "rsync done"

# Retry until success (with limit)
attempts=0
until ping -c1 -W1 8.8.8.8 &>/dev/null; do
    (( attempts++ ))
    (( attempts >= 5 )) && { echo "Network unreachable" >&2; exit 1; }
    sleep 2
done
echo "Network up"

# Process all .log files
while IFS= read -r -d '' file; do
    echo "Processing: $file"
done < <(find /var/log -name "*.log" -print0)

# Simple spinner while background job runs
some_long_command &
pid=$!
while kill -0 "$pid" 2>/dev/null; do
    printf '.'
    sleep 1
done
echo " done"

# Countdown with abort
for (( i=5; i>0; i-- )); do
    printf "\rStarting in %d... (Ctrl-C to abort)" "$i"
    sleep 1
done
echo
```

---

## Setup Notes

- **Docker**: `while` loops are useful for waiting on container readiness: `until docker exec mycontainer pg_isready; do sleep 1; done`
- **btrfs snapshots**: loop over `btrfs subvolume list / | while read ...` to process subvolumes
- **ProtonVPN**: check `until ip link show proton0 &>/dev/null; do sleep 1; done` before starting VPN-dependent tasks
- **zsh**: `while`/`until`/`break`/`continue` all work identically in zsh scripts and interactive use
