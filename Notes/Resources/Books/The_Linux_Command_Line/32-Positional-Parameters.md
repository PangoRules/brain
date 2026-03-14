# Chapter 32 – Positional Parameters

## Quick Reference

| Variable | Value |
|----------|-------|
| `$0` | Script name / full path |
| `$1` … `$9` | Arguments 1–9 |
| `${10}`, `${11}` … | Arguments 10+ (braces required) |
| `$#` | Number of arguments |
| `$*` | All args as one string (unquoted: word-split) |
| `$@` | All args as separate words (unquoted: word-split) |
| `"$*"` | All args as one quoted string (IFS-joined) |
| `"$@"` | All args as separate quoted words — **use this** |
| `shift` | Drop `$1`, shift all args down by one, `$#` decrements |
| `$FUNCNAME` | Name of current function (use inside functions, not `$0`) |
| `$(basename "$0")` | Script name without path (for usage messages) |

---

## Positional Parameters

```bash
#!/bin/bash
echo "Script:  $0"
echo "Arg 1:   $1"
echo "Arg 2:   $2"
echo "Count:   $#"
```

- `$0` is always the script path, even with no arguments
- Arguments beyond `$9` need braces: `${10}`, `${55}`
- Unset parameters expand to empty string (quote them: `"$1"`)

---

## `shift` — Iterate Over Many Arguments

Each `shift` drops `$1` and moves everything down: `$2→$1`, `$3→$2`, etc. `$#` decrements.

```bash
while [[ $# -gt 0 ]]; do
    echo "Arg: $1"
    shift
done
```

---

## `$*` vs `$@` — The Critical Difference

Given: `script "word" "words with spaces"`

| Form | Expands to | Words seen by next command |
|------|-----------|---------------------------|
| `$*` | `word words with spaces` | 4 words (splits on spaces) |
| `$@` | `word words with spaces` | 4 words (splits on spaces) |
| `"$*"` | `"word words with spaces"` | 1 word (IFS-joined) |
| `"$@"` | `"word" "words with spaces"` | 2 words ✓ |

**Always use `"$@"`** when passing arguments on to another command. It's the only form that preserves arguments with spaces intact.

```bash
# Wrapper that preserves all arguments correctly
my_wrapper () {
    some_command --option "$@"
}

# Wrong — breaks args with spaces
some_command $*
some_command $@
some_command "$*"

# Right
some_command "$@"
```

---

## `basename` and `$FUNCNAME`

```bash
PROGNAME="$(basename "$0")"   # "myscript" not "/home/pango/.local/bin/myscript"

usage () {
    echo "$PROGNAME: usage: $PROGNAME [-f file | -i]" >&2
}

# Inside functions, use $FUNCNAME instead of $0
file_info () {
    if [[ ! -e "$1" ]]; then
        echo "$FUNCNAME: usage: $FUNCNAME <file>" >&2
        return 1
    fi
    file "$1"
    stat "$1"
}
```

---

## Option Parsing with `while` + `case` + `shift`

The standard manual pattern (no `getopts`):

```bash
#!/bin/bash

PROGNAME="$(basename "$0")"

usage () {
    echo "$PROGNAME: usage: $PROGNAME [-f file | -i] [-h]" >&2
}

interactive=
filename=

while [[ -n "$1" ]]; do
    case "$1" in
        -f | --file)
            shift
            filename="$1"          # $1 is now the argument after -f
            ;;
        -i | --interactive)
            interactive=1
            ;;
        -h | --help)
            usage; exit 0
            ;;
        --)
            shift; break           # end of options, rest are positional
            ;;
        -*)
            echo "$PROGNAME: unknown option: $1" >&2
            usage; exit 1
            ;;
        *)
            break                  # first non-option arg, stop parsing
            ;;
    esac
    shift
done

# Remaining args (non-option) are still in $@
```

Key mechanics:
- Outer `shift` at end of loop advances past current option
- Extra `shift` inside `-f` branch consumes the option's argument
- `--` convention signals end of options
- `-*)` catches any unknown `-x` flags before `*)` catch-all

---

## `getopts` — The Built-in Alternative

For simple short options, `getopts` is cleaner than manual `while`+`case`:

```bash
while getopts ":f:ih" opt; do
    case "$opt" in
        f)  filename="$OPTARG" ;;
        i)  interactive=1 ;;
        h)  usage; exit 0 ;;
        :)  echo "Option -$OPTARG requires an argument" >&2; exit 1 ;;
        ?)  echo "Unknown option: -$OPTARG" >&2; exit 1 ;;
    esac
done
shift $((OPTIND - 1))   # remove parsed options, leaving positional args in $@
```

- `getopts` handles `-f value`, `-fvalue`, and combined `-fi` forms automatically
- `:` before option string suppresses default error messages (silent mode)
- `$OPTARG` holds the argument for options that take one (marked with `:` after the letter)
- After the loop, `shift $((OPTIND - 1))` leaves only non-option args in `$@`
- **Limitation**: no long options (`--file`) — use manual `while`+`case` for those

---

## Interactive Mode Pattern

```bash
if [[ -n "$interactive" ]]; then
    while true; do
        read -p "Output filename: " filename
        [[ -z "$filename" ]] && continue          # empty — ask again
        if [[ -e "$filename" ]]; then
            read -p "'$filename' exists. Overwrite? [y/n/q] > "
            case "$REPLY" in
                y|Y) break ;;
                q|Q) echo "Aborted."; exit 0 ;;
                *)   continue ;;
            esac
        else
            break
        fi
    done
fi
```

---

## Validating and Using Output File

```bash
if [[ -n "$filename" ]]; then
    if touch "$filename" && [[ -f "$filename" ]]; then
        generate_output > "$filename"
    else
        echo "$PROGNAME: Cannot write to '$filename'" >&2
        exit 1
    fi
else
    generate_output      # stdout
fi
```

`touch` + `[[ -f ]]` validates both that the path is writable and that it's a regular file (not a directory or device).

---

## Practical Patterns

```bash
# Require exactly one argument
(( $# == 1 )) || { echo "Usage: $0 <file>" >&2; exit 1; }

# Require at least one argument
(( $# >= 1 )) || { echo "Usage: $0 <file>..." >&2; exit 1; }

# Process all files passed as arguments
for file in "$@"; do
    [[ -f "$file" ]] || { echo "Not a file: $file" >&2; continue; }
    process "$file"
done

# Pass all args through to another command (wrapper pattern)
ffmpeg_wrapper () {
    ffmpeg -loglevel warning "$@"
}

# Collect remaining non-option args after getopts
while getopts "v" opt; do
    case $opt in v) verbose=1 ;; esac
done
shift $((OPTIND - 1))
files=("$@")   # array of remaining args
```

---

## Setup Notes

- Your scripts live in `~/.local/bin` — `$0` will show the full path; `basename "$0"` gives just the script name for cleaner usage messages
- **zsh**: positional parameters, `$#`, `$@`, `"$@"`, `shift`, `$FUNCNAME` all work identically; `getopts` works the same way
- For complex CLIs (many long options, subcommands), consider `getopts` for short opts + manual `case` for long opts, or use a dedicated argument parsing library
