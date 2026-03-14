# Chapter 28 – Reading Keyboard Input

## Quick Reference
-z

| Command / Syntax | Purpose |
|-----------------|---------|
| `read var` | Read one line of stdin into `var` |
| `read` | Read into `$REPLY` |
| `read -p "prompt" var` | Display prompt inline |
| `read -s var` | Silent (no echo) — passwords |
| `read -t 10 var` | Timeout after 10 seconds |
| `read -n 1 var` | Read exactly 1 character |
| `read -r var` | Raw mode (no backslash interpretation) |
| `read -a arr` | Read into array |
| `IFS=":" read a b c <<< "$str"` | Split string on `:` into variables |
| `<<<` | Here string — feed single string to stdin |

---

## Basic `read`

```bash
read var              # reads a line, stores in var
read var1 var2 var3   # splits by IFS (whitespace by default)
read                  # stores entire line in $REPLY
```

**Splitting behavior:**
- Fewer words than vars → extras are empty
- More words than vars → last var gets all remaining words
- No vars listed → everything goes to `$REPLY`

```bash
echo -n "Enter a value: "
read answer
echo "You entered: $answer"
```

---

## `read` Options

| Option | Effect |
|--------|--------|
| `-p "prompt"` | Show prompt string (no separate `echo` needed) |
| `-s` | Silent — characters not echoed (passwords) |
| `-t N` | Timeout after N seconds; returns non-zero on timeout |
| `-n N` | Read exactly N characters (no Enter needed) |
| `-r` | Raw — backslash is literal, not an escape |
| `-e` | Use Readline for editing (arrow keys, history) |
| `-i string` | Default value (requires `-e`); pre-fills input |
| `-a array` | Store words into indexed array (Ch35) |
| `-d char` | Use char as delimiter instead of newline |
| `-u fd` | Read from file descriptor `fd` instead of stdin |

### Common patterns

```bash
# Inline prompt
read -p "Enter username: " user

# Password (hidden input)
read -sp "Password: " pass
echo    # newline after silent input

# Timeout with error handling
if read -t 15 -p "Continue? [y/N] " answer; then
    echo "Got: $answer"
else
    echo -e "\nTimed out." >&2
    exit 1
fi

# Default value (shows pre-filled, user can edit)
read -e -p "Username: " -i "$USER" name

# Read single keypress (no Enter)
read -n 1 -p "Press any key..."
```

---

## `$REPLY`

When `read` is called with no variable name, input goes to `$REPLY`:

```bash
read -p "Enter something: "
echo "You typed: $REPLY"
```

---

## IFS — Internal Field Separator

IFS controls how `read` splits input into variables. Default: space, tab, newline.

**Temporary IFS override** (only affects the one command):
```bash
IFS=":" read user pw uid gid name home shell <<< "$line"
```

**Example — parse `/etc/passwd`:**
```bash
read -p "Username: " user_name
file_info="$(grep "^${user_name}:" /etc/passwd)"
if [[ -n "$file_info" ]]; then
    IFS=":" read user pw uid gid name home shell <<< "$file_info"
    echo "UID: $uid  Shell: $shell  Home: $home"
else
    echo "User not found" >&2
    exit 1
fi
```

**Permanent change** (restore afterward):
```bash
OLD_IFS="$IFS"
IFS=","
# ... read commands ...
IFS="$OLD_IFS"
```

---

## Here Strings `<<<`

Feed a single string to a command's stdin without a pipe:

```bash
read var <<< "hello world"   # var = "hello world"

IFS=":" read a b c <<< "one:two:three"
# a=one  b=two  c=three
```

---

## You Can't Pipe to `read`

This does **not** work as expected:

```bash
echo "foo" | read var   # var is empty!
```

**Why**: pipes create subshells in bash. Variable assignments in a subshell don't propagate back to the parent. The `read` happens in the subshell, then the subshell exits and the variable is gone.

**Fix**: use a here string instead:
```bash
read var <<< "foo"   # works — no subshell
```

> **zsh note**: zsh by default runs the last pipeline segment in the current shell (not a subshell), so `echo "foo" | read var` actually works in interactive zsh. But in bash scripts (`#!/bin/bash`) it won't. Use `<<<` for portability.

---

## Input Validation

Combine `read` with `[[ =~ ]]` to validate before processing:

```bash
read -p "Enter an integer: " INT

# Validate integer
[[ "$INT" =~ ^-?[0-9]+$ ]] || { echo "Not an integer" >&2; exit 1; }

# Validate non-empty
[[ -z "$REPLY" ]] && { echo "Input required" >&2; exit 1; }

# Validate single word (no spaces)
(( "$(echo "$REPLY" | wc -w)" > 1 )) && { echo "Single item only" >&2; exit 1; }

# Validate filename-safe string
[[ "$REPLY" =~ ^[-[:alnum:]._]+$ ]] || { echo "Invalid characters" >&2; exit 1; }

# Validate floating point
[[ "$REPLY" =~ ^-?[[:digit:]]*\.[[:digit:]]+$ ]] && echo "float"

# Validate y/n
[[ "$REPLY" =~ ^[yYnN]$ ]] || { echo "Enter y or n" >&2; exit 1; }
```

---

## Menu-Driven Scripts

Pattern for interactive menus:

```bash
#!/bin/bash

show_menu() {
    clear
    echo "
Please Select:
  1. System info
  2. Disk space
  3. Home usage
  0. Quit
"
    read -p "Enter selection [0-3]: "
}

show_menu

if [[ ! "$REPLY" =~ ^[0-3]$ ]]; then
    echo "Invalid selection" >&2
    exit 1
fi

case "$REPLY" in
    0) echo "Bye."; exit ;;
    1) echo "Host: $HOSTNAME"; uptime ;;
    2) df -h ;;
    3) if (( $(id -u) == 0 )); then
           du -sh /home/* 2>/dev/null
       else
           du -sh "$HOME"
       fi ;;
esac
```

> Note: The book uses chained `if` statements for the menu dispatch; `case` (Ch29) is cleaner and is introduced next chapter.

---

## Practical Patterns

```bash
# Confirm before destructive action
read -rp "Delete all Docker volumes? [y/N] " confirm
[[ "$confirm" =~ ^[yY]$ ]] || { echo "Aborted."; exit 0; }

# Ask for sudo password once
read -sp "Enter sudo password: " sudo_pass
echo

# Read a file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/hosts

# Read config file with key=value pairs
while IFS="=" read -r key val; do
    echo "$key → $val"
done < myconfig.ini

# Countdown with keypress abort
echo "Starting in 5s... (press any key to abort)"
read -t 5 -n 1 && { echo "Aborted."; exit 0; }
```

---

## Setup Notes

- Your shell is **zsh** — `read` behaves the same for script use; the pipe-to-read subshell caveat is bash-specific but use `<<<` anyway for clarity
- `/etc/passwd` example works on Fedora but most real users are local; for LDAP/SSO users use `getent passwd username` instead
- `df -h` on your btrfs setup will show your 1.82 TiB `/` — add `-x tmpfs` to exclude tmpfs mounts
