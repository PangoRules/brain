# Chapter 31 – Flow Control: Branching with `case`

## Quick Reference

| Syntax | Meaning |
|--------|---------|
| `case word in` | Start case block, match `word` |
| `pattern)` | Pattern to match (glob-style) |
| `;;` | End branch, exit case (first match only) |
| `;;&` | End branch, continue testing next patterns (bash 4.0+) |
| `p1\|p2)` | OR — match either pattern |
| `*)` | Default / catch-all (put last) |
| `esac` | End case block |

---

## Syntax

```bash
case word in
    pattern1)
        commands
        ;;
    pattern2 | pattern3)
        commands
        ;;
    *)
        commands   # default / catch-all
        ;;
esac
```

- `word` is expanded once, then matched against each pattern in order
- First match wins — no further patterns are tested (unless `;;&`)
- `*)` catches anything not matched above; always put it last

---

## Patterns (Glob-Style, Not Regex)

| Pattern | Matches |
|---------|---------|
| `a)` | exactly the string `a` |
| `[[:alpha:]])` | single alphabetic character |
| `[[:digit:]])` | single digit |
| `[ABC][0-9])` | A, B, or C followed by a digit |
| `???)` | exactly three characters |
| `*.txt)` | anything ending in `.txt` |
| `q\|Q)` | literal `q` or `Q` |
| `*)` | anything (catch-all) |

Same glob syntax as pathname expansion and `[[ == ]]` pattern matching.

---

## Basic Example — Menu

```bash
read -p "Enter selection [0-3]: "

case "$REPLY" in
    0)  echo "Quit"; exit ;;
    1)  echo "Host: $HOSTNAME"; uptime ;;
    2)  df -h ;;
    3)  if (( $(id -u) == 0 )); then
            du -sh /home/* 2>/dev/null
        else
            du -sh "$HOME"
        fi ;;
    *)  echo "Invalid entry" >&2; exit 1 ;;
esac
```

---

## OR Patterns — Case-Insensitive Input

```bash
case "$REPLY" in
    q|Q)    exit ;;
    y|Y)    echo "Yes" ;;
    n|N)    echo "No" ;;
    *)      echo "Enter y or n" >&2 ;;
esac
```

---

## Fall-Through with `;;&` (bash 4.0+)

`;;` stops after the first match. `;;&` continues testing subsequent patterns:

```bash
read -n 1 -p "Type a character: "
echo
case "$REPLY" in
    [[:upper:]])  echo "upper case" ;;&
    [[:lower:]])  echo "lower case" ;;&
    [[:alpha:]])  echo "alphabetic" ;;&
    [[:digit:]])  echo "digit"      ;;&
    [[:xdigit:]]) echo "hex digit"  ;;&
    [[:graph:]])  echo "visible"    ;;&
esac
# For 'a': prints lower case, alphabetic, hex digit, visible
```

Use `;;&` when a value can legitimately match multiple categories.

> **zsh note**: zsh uses `;&` (fall through to next branch, skip its pattern test) and `;|` (test next pattern). The bash `;;&` behaviour maps to zsh's `;|`. In practice, `;;&` is rare — use it only when you genuinely need multi-category classification.

---

## Practical Patterns

### File type dispatcher

```bash
case "$file" in
    *.tar.gz|*.tgz)  tar xzf "$file" ;;
    *.tar.bz2)       tar xjf "$file" ;;
    *.tar.xz)        tar xJf "$file" ;;
    *.tar.zst)       tar --zstd -xf "$file" ;;
    *.zip)           unzip "$file" ;;
    *.7z)            7z x "$file" ;;
    *.gz)            gunzip "$file" ;;
    *.bz2)           bunzip2 "$file" ;;
    *.xz)            unxz "$file" ;;
    *)               echo "Unknown archive format: $file" >&2; return 1 ;;
esac
```

### Script argument / subcommand dispatch

```bash
case "$1" in
    start)   systemctl start myservice ;;
    stop)    systemctl stop myservice ;;
    restart) systemctl restart myservice ;;
    status)  systemctl status myservice ;;
    -h|--help) echo "Usage: $0 {start|stop|restart|status}" ;;
    *)       echo "Unknown command: $1" >&2; exit 1 ;;
esac
```

### OS/distro detection

```bash
case "$(uname -s)" in
    Linux)   echo "Linux" ;;
    Darwin)  echo "macOS" ;;
    *)       echo "Unknown OS" ;;
esac

# Distro-specific (your system)
case "$(grep -oP '(?<=^ID=).+' /etc/os-release)" in
    fedora)  PKG_MGR="dnf" ;;
    ubuntu|debian) PKG_MGR="apt" ;;
    arch)    PKG_MGR="pacman" ;;
    *)       PKG_MGR="unknown" ;;
esac
```

### Y/N confirmation (reusable pattern)

```bash
read -rp "Continue? [y/N] " confirm
case "$confirm" in
    [yY]|[yY][eE][sS]) : ;;   # proceed
    *) echo "Aborted."; exit 0 ;;
esac
```

---

## `case` vs Chained `if/elif`

| Scenario | Prefer |
|----------|--------|
| Matching one variable against many fixed values | `case` |
| Glob/pattern matching on a string | `case` |
| Comparing two different variables | `if/elif` |
| Numeric range checks (`-gt`, `-le`) | `if/elif` |
| Regex matching (`=~`) | `if [[ ]]` |

`case` is cleaner and faster for multi-branch string dispatch. For anything numeric or involving two-variable comparisons, stick with `if`.
