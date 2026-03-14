# Chapter 27 – Flow Control: Branching with `if`

## Quick Reference

| Construct | Use |
|-----------|-----|
| `if [ expr ]; then ... fi` | POSIX-compatible test |
| `if [[ expr ]]; then ... fi` | Modern bash/zsh — preferred |
| `if (( arith )); then ... fi` | Integer arithmetic test |
| `elif` | Else-if chain |
| `$?` | Exit status of last command |
| `true` / `false` | Always succeed / always fail |
| `cmd1 && cmd2` | Run cmd2 only if cmd1 succeeds |
| `cmd1 \|\| cmd2` | Run cmd2 only if cmd1 fails |

---

## `if` Syntax

```bash
if commands; then
    commands
elif commands; then
    commands
else
    commands
fi
```

The condition is a **command** — `if` checks its exit status. Zero = true, non-zero = false.

---

## Exit Status

```bash
ls /etc
echo $?      # 0 = success

ls /nonexistent
echo $?      # 2 = failure
```

Every command returns an exit status. `if` evaluates this directly:

```bash
if ls /etc/hosts; then
    echo "file exists"
fi
```

`true` always exits 0; `false` always exits 1 — useful for stubs and loops.

---

## `test` / `[ ]`

```bash
[ expression ]   # spaces inside brackets are required
test expression  # equivalent
```

**`[ ]` is a command, not syntax** — spaces matter.

### File Expressions

| Expression | True if |
|------------|---------|
| `-e file` | file exists |
| `-f file` | regular file |
| `-d file` | directory |
| `-r file` | readable |
| `-w file` | writable |
| `-x file` | executable |
| `-L file` | symbolic link |
| `-s file` | size > 0 (non-empty) |
| `-z file` | file length is zero (empty file) — don't confuse with `-z string` |
| `file1 -nt file2` | file1 newer than file2 |
| `file1 -ot file2` | file1 older than file2 |

```bash
if [ -f ~/.zshrc ]; then
    echo "zshrc exists"
fi

if [ -d /var/lib/docker ]; then
    echo "Docker data dir present"
fi
```

### String Expressions

| Expression | True if |
|------------|---------|
| `-z string` | string is empty (zero length) |
| `-n string` | string is non-empty |
| `str1 = str2` | strings equal (note: `=` not `==` in `[ ]`) |
| `str1 != str2` | strings not equal |
| `str1 > str2` | str1 sorts after str2 |
| `str1 < str2` | str1 sorts before str2 |

> **Warning**: `>` and `<` must be quoted or escaped in `[ ]` to avoid shell redirection: `[ "$a" \< "$b" ]`

```bash
if [ -z "$ANSWER" ]; then
    echo "No answer given" >&2
    exit 1
fi

if [ "$ANSWER" = "yes" ]; then
    echo "YES"
elif [ "$ANSWER" = "no" ]; then
    echo "NO"
else
    echo "Unknown"
fi
```

### Integer Expressions

| Expression | Meaning |
|------------|---------|
| `int1 -eq int2` | equal |
| `int1 -ne int2` | not equal |
| `int1 -lt int2` | less than |
| `int1 -le int2` | less than or equal |
| `int1 -gt int2` | greater than |
| `int1 -ge int2` | greater than or equal |

```bash
if [ "$INT" -eq 0 ]; then
    echo "zero"
elif [ "$INT" -lt 0 ]; then
    echo "negative"
else
    echo "positive"
fi
```

---

## `[[ ]]` — Modern Replacement (Preferred)

`[[ ]]` is a shell keyword (not a command), so it handles `<`, `>`, `&&`, `||` without quoting issues. Supported in bash and zsh.

**Key additions over `[ ]`:**

| Feature              | Example                           |
| -------------------- | --------------------------------- |
| Regex match          | `[[ "$str" =~ ^-?[0-9]+$ ]]`      |
| Glob pattern match   | `[[ $FILE == *.tar.gz ]]`         |
| No word splitting    | safe without quotes in most cases |
| `&&` / `\|\|` inside | `[[ -f a && -f b ]]`              |

```bash
# Validate integer
if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
    echo "valid integer"
else
    echo "not an integer" >&2
    exit 1
fi

# Range check
if [[ "$INT" -ge 1 && "$INT" -le 100 ]]; then
    echo "in range"
fi

# Negate with grouping
if [[ ! ("$INT" -ge 1 && "$INT" -le 100) ]]; then
    echo "out of range"
fi

# Pattern match (glob, not regex)
FILE=report.tar.gz
if [[ $FILE == *.tar.gz ]]; then
    echo "it's a tarball"
fi
```

---

## `(( ))` — Integer Arithmetic Tests

Uses C-style operators. Variables don't need `$`. Non-zero result = true.

```bash
if (( INT == 0 )); then echo "zero"; fi
if (( INT < 0 ));  then echo "negative"; fi
if (( (INT % 2) == 0 )); then echo "even"; fi

# Arithmetic truth test
if (( 1 )); then echo "true"; fi   # always true
if (( 0 )); then echo "true"; fi   # never executes
```

Combined with `[[ ]]` for integer validation:
```bash
if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
    if (( INT < 0 )); then
        echo "negative"
    fi
fi
```

---

## Logical Operators

| Operation | `[ ]` / `test` | `[[ ]]` / `(( ))` |
|-----------|---------------|-------------------|
| AND | `-a` | `&&` |
| OR | `-o` | `\|\|` |
| NOT | `!` | `!` |

In `test`/`[ ]`, grouping requires escaped parens: `\( ... \)`

```bash
# test style (avoid)
if [ "$INT" -ge 1 -a "$INT" -le 100 ]; then ...

# [[ ]] style (preferred)
if [[ "$INT" -ge 1 && "$INT" -le 100 ]]; then ...
```

---

## Control Operators: `&&` and `||`

Short-circuit evaluation outside of `if`:

```bash
# Run cd only if mkdir succeeds
mkdir /tmp/work && cd /tmp/work

# Create dir only if it doesn't exist
[[ -d /tmp/work ]] || mkdir /tmp/work

# Exit script if required dir missing
[[ -d /var/lib/docker ]] || { echo "Docker dir missing" >&2; exit 1; }
```

This pattern is common in scripts for lightweight error handling without full `if` blocks.

---

## Practical Patterns

```bash
# Check if running as root
if [[ "$(id -u)" -eq 0 ]]; then
    echo "Running as root"
fi

# Check command exists
if [[ -x "$(command -v ffmpeg)" ]]; then
    echo "ffmpeg available"
fi

# Safe script guard
[[ -f "$1" ]] || { echo "Usage: $0 <file>" >&2; exit 1; }

# Multi-condition file check
if [[ -f "$CONFIG" && -r "$CONFIG" ]]; then
    source "$CONFIG"
fi

# Validate non-empty string
if [[ -z "$INPUT" ]]; then
    echo "Error: INPUT is empty" >&2
    exit 1
fi
```

---

## `[ ]` vs `[[ ]]` — Which to Use

| Scenario | Use |
|----------|-----|
| Portability (POSIX sh, `/bin/sh` scripts) | `[ ]` / `test` |
| bash/zsh scripts (most of the time) | `[[ ]]` |
| Integer arithmetic | `(( ))` |
| Regex matching | `[[ =~ ]]` |

**Verdict**: Use `[[ ]]` for all bash/zsh scripts. Use `(( ))` for integer math. Only fall back to `[ ]` if writing POSIX-portable sh scripts.

> **zsh note**: Both `[[ ]]` and `(( ))` work in zsh. zsh also supports `[[ str =~ regex ]]`. Your `.zshrc` functions and interactive use follow the same rules.

---

## Setup-Specific Examples

```bash
# Check if ProtonVPN interface is up
if [[ -d /proc/sys/net/ipv4/conf/proton0 ]]; then
    echo "ProtonVPN active"
fi

# Check Docker daemon running
if systemctl is-active --quiet docker; then
    echo "Docker running"
fi

# Btrfs check
if [[ "$(stat -f --format=%T /)" == "btrfs" ]]; then
    echo "Root is btrfs"
fi

# In a script: run with elevated disk usage if root
if [[ "$(id -u)" -eq 0 ]]; then
    du -sh /home/*
else
    du -sh "$HOME"
fi
```
