# Chapter 36 – Exotica

## Quick Reference

| Feature | Syntax |
|---------|--------|
| Group command (current shell) | `{ cmd1; cmd2; }` |
| Subshell (child shell) | `(cmd1; cmd2)` |
| Process substitution (output) | `<(list)` |
| Process substitution (input) | `>(list)` |
| Set trap | `trap "cmd" SIGNAL` |
| Remove trap | `trap - SIGNAL` |
| Background job | `cmd &` |
| Get last background PID | `$!` |
| Wait for PID | `wait $pid` |
| Create temp file | `mktemp /tmp/name.XXXXXXXXXX` |
| Create named pipe | `mkfifo pipename` |

---

## Group Commands `{ }` vs Subshells `( )`

### Group command — runs in **current shell**

```bash
{ ls -l; echo "done"; cat file.txt; } > output.txt
{ ls -l; echo "done"; cat file.txt; } | sort
```

- Braces require: **space after `{`**, **semicolon or newline before `}`**
- Variable assignments **persist** after the block
- Faster, uses less memory than subshells

### Subshell — runs in **child shell**

```bash
(ls -l; echo "done"; cat file.txt) > output.txt
(cd /tmp && do_something)          # cd doesn't affect parent
```

- Variable assignments and `cd` are **lost** when subshell exits
- Use when you want isolation (e.g., temporary `cd`)

### Redirect group output

```bash
# Capture multi-command output into a variable
result=$(
    echo "Header"
    date
    uptime
)

# Append multiple commands to log
{
    echo "=== $(date) ==="
    systemctl status docker
    df -h
} >> /var/log/daily-check.log
```

---

## Process Substitution `<( )` and `>( )`

Treats command output as a file (actually `/dev/fd/N`). Solves the pipe-to-`read` subshell problem.

```bash
# BROKEN — read runs in subshell, REPLY is lost
echo "foo" | read
echo "$REPLY"            # empty

# FIXED — process substitution, no subshell for read
read < <(echo "foo")
echo "$REPLY"            # foo
```

### Canonical file-reading loop (with pipeline source)

```bash
# Read sorted output without losing variables
while IFS= read -r line; do
    echo "$line"
    (( count++ ))
done < <(sort /etc/passwd)
echo "Total: $count"     # count is accessible here
```

### Compare two command outputs

```bash
diff <(ls /dir1) <(ls /dir2)
diff <(sort file1) <(sort file2)

# Compare running config vs saved config
diff <(nginx -T 2>/dev/null) nginx-backup.conf
```

### Other uses

```bash
# Read into array from command
mapfile -t lines < <(find /var/log -name "*.log" -newer /tmp/marker)

# tee to multiple destinations
tee >(gzip > output.gz) >(wc -l) > /dev/null < input.txt

# Parse docker inspect
while IFS= read -r line; do
    echo "Container: $line"
done < <(docker ps --format '{{.Names}}')
```

---

## Traps — Signal Handling

```bash
trap "commands" SIGNAL [SIGNAL...]
trap - SIGNAL        # restore default handler
trap "" SIGNAL       # ignore signal
trap                 # list current traps
```

### Common signals

| Signal | Number | Trigger |
|--------|--------|---------|
| `SIGINT` | 2 | Ctrl-C |
| `SIGTERM` | 15 | `kill` default |
| `SIGHUP` | 1 | terminal close |
| `EXIT` | — | script exits (any reason) |
| `ERR` | — | any command returns non-zero |
| `DEBUG` | — | before every command |

### Clean up temp files on exit

```bash
#!/bin/bash
tmpfile=$(mktemp /tmp/myscript.XXXXXXXXXX)

cleanup() {
    rm -f "$tmpfile"
    echo "Cleaned up." >&2
}
trap cleanup EXIT          # runs on any exit, including errors

# ... rest of script uses $tmpfile ...
```

### Graceful interrupt handling

```bash
interrupted=0

handle_sigint() {
    echo -e "\nInterrupted — cleaning up..." >&2
    interrupted=1
}
trap handle_sigint SIGINT

for f in *.log; do
    (( interrupted )) && break
    process "$f"
done
```

### `trap` on `ERR` (complement to `set -e`)

```bash
trap 'echo "Error at line $LINENO" >&2' ERR
```

---

## Temporary Files

```bash
# Basic (not safe — predictable name)
tmpfile=/tmp/$(basename "$0").$$.$RANDOM

# Correct — mktemp (unpredictable, atomic creation)
tmpfile=$(mktemp /tmp/myscript.XXXXXXXXXX)

# In $HOME (avoids /tmp world-writable security issues)
[[ -d "$HOME/tmp" ]] || mkdir "$HOME/tmp"
tmpfile=$(mktemp "$HOME/tmp/myscript.XXXXXXXXXX")

# Temp directory
tmpdir=$(mktemp -d /tmp/myscript.XXXXXXXXXX)
trap 'rm -rf "$tmpdir"' EXIT
```

- More `X`s = more randomness in the suffix
- `$$` adds PID but is not sufficient alone — combine with `mktemp`
- Always pair with a `trap ... EXIT` to ensure cleanup

---

## Asynchronous Execution with `wait`

```bash
# Launch background job, capture PID
some_command &
pid=$!

# Do other work...

# Wait for specific PID
wait "$pid"
echo "Exit status: $?"

# Wait for all background jobs
wait
```

### Parallel processing pattern

```bash
#!/bin/bash
# Process files in parallel, limit concurrency

max_jobs=8    # or: nproc (gives 24 on your i9)
pids=()

for f in *.mkv; do
    ffmpeg -i "$f" "${f%.mkv}.mp4" &
    pids+=($!)

    # Throttle: wait when at max
    while (( ${#pids[@]} >= max_jobs )); do
        for i in "${!pids[@]}"; do
            if ! kill -0 "${pids[i]}" 2>/dev/null; then
                unset 'pids[i]'
            fi
        done
        pids=("${pids[@]}")   # repack array
        sleep 0.5
    done
done

# Wait for remaining
for pid in "${pids[@]}"; do wait "$pid"; done
echo "All done."
```

---

## Named Pipes (FIFOs)

```bash
mkfifo mypipe
ls -l mypipe     # prw-r--r-- (p = pipe)

# Terminal 1: write to pipe (blocks until reader attaches)
ls -l > mypipe

# Terminal 2: read from pipe
cat < mypipe
```

Behaves like `ls -l | cat` but decoupled in time and across processes/scripts.

```bash
# Practical: log multiplexer
mkfifo /tmp/logpipe
tee /tmp/logpipe < /dev/stdin | logger -t myscript &
cat /tmp/logpipe | grep ERROR > errors.log
```

Named pipes are rarely used directly — process substitution `<()` is usually cleaner for the same job.

---

## Practical Combinations

```bash
# Capture + display simultaneously (tee trick)
output=$(some_command | tee /dev/stderr)

# Run multiple checks, redirect all to one log
{
    echo "=== System Check $(date) ==="
    uptime
    df -h
    free -h
    docker ps 2>/dev/null || echo "Docker not running"
    ip -br addr show proton0 2>/dev/null || echo "VPN down"
} | tee -a ~/system-check.log

# Safe background script with cleanup
run_with_cleanup() {
    local tmpdir; tmpdir=$(mktemp -d)
    trap "rm -rf '$tmpdir'" EXIT SIGINT SIGTERM

    (
        cd "$tmpdir"
        do_work_here
    )
}

# Process substitution + associative array (no subshell variable loss)
declare -A owner_count
while read -r owner; do
    (( ++owner_count["$owner"] ))
done < <(stat -c %U /usr/bin/*)
```

---

## Setup Notes

- **`$!` and parallel jobs**: on your 24-thread i9, batching with `wait` is worth it for CPU-bound tasks (ffmpeg, image processing, compilation)
- **Traps and Docker**: use `trap 'docker rm -f $container_id' EXIT` when scripts spin up temporary containers
- **ProtonVPN**: use `trap cleanup EXIT` in any script that modifies routing tables or firewall rules
- **Named pipes**: occasionally useful for KDE Plasma scripting — `mkfifo` can connect a long-running process to a status indicator
- **zsh**: group commands, subshells, process substitution, `trap`, `wait`, `mktemp`, `mkfifo` all work identically; zsh additionally supports `=(cmd)` as a process substitution variant that provides a real temp file path
