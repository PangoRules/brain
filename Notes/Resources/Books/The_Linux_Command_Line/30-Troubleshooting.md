# Chapter 30 – Troubleshooting

## Quick Reference

| Technique | How |
|-----------|-----|
| Whole-script trace | `#!/bin/bash -x` or `bash -x script.sh` |
| Partial trace | `set -x` … `set +x` |
| Line numbers in trace | `export PS4='$LINENO + '` |
| Abort on error | `set -e` |
| Abort on unset var | `set -u` |
| Both + pipefail | `set -euo pipefail` |
| Static analysis | `shellcheck script.sh` |
| Safe wildcard | `rm ./*` not `rm *` |
| Check var before use | `[[ -n "$var" ]]` or quote: `"$var"` |

---

## Common Syntax Errors

### Missing quotes

```bash
echo "forgot to close    # bash keeps scanning for the closing "
                         # error reported much later than the actual line
```

Fix: editor with syntax highlighting catches this immediately. Kate (your KDE editor) and VSCode both highlight unclosed strings.

### Missing semicolons / tokens

```bash
if [ $number = 1 ] then   # missing ; before then
```

Error points to a later line (`else` or `fi`) — not the actual problem. The rule: always `]; then` or `]; do`.

### Unanticipated expansion (the most common silent killer)

```bash
number=
[ $number = 1 ]   # expands to [  = 1 ] — unary operator expected
```

**Fix: always double-quote variables in `[ ]`:**
```bash
[ "$number" = 1 ]   # expands to [ "" = 1 ] — correct
```

Rule: **quote every variable and command substitution** unless you explicitly need word splitting.

---

## Logical Errors

Common types:
- **Reversed logic** — `if` condition is backwards (`-eq` when you meant `-ne`)
- **Off-by-one** — loop starts at 1 when 0 is needed, or `<=` vs `<`
- **Unanticipated input** — empty string, filename with spaces, leading hyphen

---

## Defensive Programming

### The `cd && rm` disaster pattern

```bash
# DANGEROUS
cd $dir_name
rm *           # if cd fails, deletes files in current dir

# BETTER
cd "$dir_name" && rm ./*

# BEST
if [[ ! -d "$dir_name" ]]; then
    echo "No such directory: '$dir_name'" >&2; exit 1
fi
if ! cd "$dir_name"; then
    echo "Cannot cd to '$dir_name'" >&2; exit 1
fi
rm ./*
```

### Leading-hyphen filename protection

```bash
rm *      # dangerous: -rf~ is a valid filename, treated as flags
rm ./*    # safe: ./ prefix prevents flag interpretation
```

Use `./*` any time you expand wildcards with file-operating commands (`rm`, `chmod`, `chown`, etc.).

### Guard clauses for scripts

```bash
# Fail fast on bad input
[[ -n "$1" ]]   || { echo "Usage: $0 <dir>" >&2; exit 1; }
[[ -d "$1" ]]   || { echo "Not a directory: $1" >&2; exit 1; }
[[ -r "$1" ]]   || { echo "Cannot read: $1" >&2; exit 1; }
```

---

## `set` Options for Robustness

```bash
set -e          # exit immediately on any error
set -u          # treat unset variables as errors
set -o pipefail # pipeline fails if any command fails (not just last)
set -x          # trace mode (prints each command before executing)

# Combine at top of script:
set -euo pipefail
```

Or in shebang: `#!/bin/bash -euo pipefail` (but `-euo pipefail` needs `set -o pipefail` separately from the shebang — safer to use `set`).

> `set -e` has subtle gotchas: commands in `if` tests, `||`, `&&` chains are exempt. Use carefully and test edge cases.

---

## Tracing with `bash -x`

### Whole script

```bash
bash -x myscript.sh
# or
#!/bin/bash -x
```

Output prefixed with `+`:
```
+ number=1
+ '[' 1 = 1 ']'
+ echo 'Number is equal to 1.'
Number is equal to 1.
```

### Partial trace

```bash
set -x   # enable trace
# ... suspect code ...
set +x   # disable trace
```

### Add line numbers to trace output

```bash
export PS4='$LINENO + '
bash -x myscript.sh
# 5 + number=1
# 7 + '[' 1 = 1 ']'
```

More detailed PS4 (function name + line):
```bash
export PS4='+${BASH_SOURCE}:${LINENO}:${FUNCNAME[0]}: '
```

---

## Testing Techniques

### Stub dangerous commands with `echo`

```bash
echo rm ./*        # TESTING — prints what would be deleted
echo "would cd to: $dir_name"   # TESTING
exit               # TESTING — stop before rest of script runs
```

Mark with `# TESTING` comments so they're easy to find and remove.

### Comment out sections to isolate

```bash
# else
#     echo "no such directory" >&2
#     exit 1
```

Narrows down whether a removed section affects the bug.

### Test cases to cover

For any input-dependent code, test:
1. Valid expected input
2. Empty / missing input
3. Edge cases (zero, negative, max, path with spaces)
4. Malicious-ish input (hyphens, wildcards, slashes)

---

## `shellcheck` — Static Analysis (Best Tool Not in the Book)

```bash
dnf install ShellCheck   # Fedora
shellcheck myscript.sh
```

Catches most of the errors in this chapter before you even run the script:
- Unquoted variables
- `[ ]` vs `[[ ]]` issues
- Useless use of `cat`
- Word splitting problems
- Deprecated syntax

Also integrates with Kate (LSP via bash-language-server) and VSCode.

---

## Examining Variables During Execution

```bash
echo "DEBUG: var=$var" >&2         # always to stderr
echo "DEBUG: array=${arr[*]}" >&2
printf "DEBUG: %q\n" "$var" >&2    # %q shows invisible chars, spaces safely
```

`%q` format in `printf` is particularly useful — it quotes the value so you can see exactly what's in the variable including spaces, newlines, etc.

---

## Practical Debugging Workflow

1. **Run with `bash -x`** first — see what's actually executing
2. **Add `set -euo pipefail`** — make silent failures loud
3. **`shellcheck`** — catch static issues before running
4. **Echo guard clauses** — verify preconditions at entry points
5. **Comment out sections** — binary-search the problem area
6. **`printf '%q'` variables** — reveal hidden characters/spaces
7. **Test with `echo` stubs** before running destructive commands

---

## Setup Notes

- **Kate**: has built-in bash syntax highlighting — unclosed quotes show immediately; install `bash-language-server` via npm for shellcheck integration
- **Your scripts in `~/.local/bin`**: test dangerous ones with `bash -x ~/.local/bin/myscript` rather than running directly
- **Docker scripts**: always guard `cd` before any `rm` inside container paths — a missing mount means you're operating on the host
- **btrfs**: `rm ./*` safety matters especially when working in subvolume roots — check mount points with `findmnt` before cleanup scripts
