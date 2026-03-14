# Chapter 19 — Regular Expressions

Regular expressions (regex) are symbolic notations for identifying patterns in text. More powerful than shell globs — they work inside file *contents*, not just filenames.

## Quick Reference

| Command  | Description                                         |
| -------- | --------------------------------------------------- |
| `grep`   | Search lines in files matching a regex              |
| `grep -E`| Extended regex (ERE) — use this over `egrep`        |
| `grep -P`| Perl-compatible regex (PCRE) — most powerful        |

---

## grep

```bash
grep [options] regex [file...]

grep 'pattern' file.txt
ls /usr/bin | grep zip          # pipe into grep
grep -r 'pattern' ~/projects/   # recursive search
```

### Key Options

| Option | Long option            | Description                                    |
| ------ | ---------------------- | ---------------------------------------------- |
| `-i`   | `--ignore-case`        | Case-insensitive match                         |
| `-v`   | `--invert-match`       | Print lines that do *not* match                |
| `-c`   | `--count`              | Print match count, not the lines               |
| `-l`   | `--files-with-matches` | Print only filenames that contain a match      |
| `-L`   | `--files-without-match`| Print only filenames with no match             |
| `-n`   | `--line-number`        | Prefix matched lines with line number          |
| `-h`   | `--no-filename`        | Suppress filename prefix in multi-file search  |
| `-E`   | `--extended-regexp`    | Use ERE (replaces `egrep`)                     |
| `-r`   | `--recursive`          | Search all files under a directory             |
| `-o`   | `--only-matching`      | Print only the matched part, not the whole line|

> Always quote patterns in single quotes to prevent the shell from interpreting regex metacharacters.

---

## Regex Metacharacters

```
^ $ . [ ] { } - ? * + ( ) | \
```

Everything else is a **literal** (matches itself). The backslash `\` escapes a metacharacter to treat it as a literal.

---

## Anchors

```bash
grep '^zip'     # matches lines that START with "zip"
grep 'zip$'     # matches lines that END with "zip"
grep '^zip$'    # matches lines that contain ONLY "zip"
grep '^$'       # matches blank lines
```

---

## The Dot — Any Character

`.` matches exactly one character, any character.

```bash
grep '.zip'     # 4-char match: anything + "zip" (won't match bare "zip")
grep 'z.p'      # z, any char, p  →  "zip", "zap", "z p", etc.
```

---

## Bracket Expressions — Character Sets

Match one character from a defined set.

```bash
grep '[bg]zip'      # matches "bzip" or "gzip"
grep '[^bg]zip'     # matches anything + "zip" except "b" or "g" before it
grep '^[A-Z]'       # lines starting with an uppercase letter
grep '[0-9]'        # any digit
```

- `[^...]` — negation: character must *not* be in the set
- `-` inside brackets is a range, e.g. `[a-z]`, `[0-9]`
- To include a literal `-`, put it first: `[-AZ]`

### POSIX Character Classes (prefer these over ranges)

Ranges like `[A-Z]` depend on locale collation order (`en_US.UTF-8` uses dictionary order, not ASCII order), which produces unexpected results. Use POSIX classes instead:

| Class        | Equivalent (ASCII)  | Matches                        |
| ------------ | ------------------- | ------------------------------ |
| `[:alnum:]`  | `[A-Za-z0-9]`       | Letters and digits             |
| `[:alpha:]`  | `[A-Za-z]`          | Letters only                   |
| `[:digit:]`  | `[0-9]`             | Digits only                    |
| `[:lower:]`  | `[a-z]`             | Lowercase letters              |
| `[:upper:]`  | `[A-Z]`             | Uppercase letters              |
| `[:space:]`  | `[ \t\r\n\v\f]`     | Any whitespace                 |
| `[:blank:]`  | `[ \t]`             | Space and tab only             |
| `[:punct:]`  | punctuation chars   | Punctuation                    |
| `[:word:]`   | `[A-Za-z0-9_]`      | Word chars (letters, digits, _)|
| `[:xdigit:]` | `[0-9A-Fa-f]`       | Hex digits                     |

```bash
grep '^[[:upper:]]'         # lines starting with uppercase — locale-safe
grep '[[:digit:]]'          # any line with a digit
grep '[[:alpha:]][[:digit:]]'  # letter followed immediately by digit
```

> Note double brackets: `[[:upper:]]` — the outer `[]` is the bracket expression, inner `[:upper:]` is the class name.

---

## BRE vs ERE

| Feature        | BRE (basic)     | ERE (`-E`)      |
| -------------- | --------------- | --------------- |
| `.` `^` `$` `*` `[]` | ✓         | ✓               |
| `+` `?` `\|` `()` `{}` | need `\` | no `\` needed |

**Always use `grep -E`** for anything involving `+`, `?`, `|`, `()`, or `{}`. It's cleaner and `egrep` is deprecated.

```bash
grep -E 'pattern'       # extended regex
grep 'patt\+ern'        # BRE: escape + to use it as quantifier (awkward)
```

---

## Alternation (ERE)

`|` means OR. Group with `()` to scope it.

```bash
grep -E 'AAA|BBB|CCC'               # match any of the three
grep -E '^(bz|gz|zip)'              # lines starting with bz, gz, or zip
grep -E '^bz|gz|zip'                # different: ^bz OR gz anywhere OR zip anywhere
```

---

## Quantifiers (ERE)

Apply to the preceding element (character, class, or group).

| Quantifier | Meaning                       |
| ---------- | ----------------------------- |
| `?`        | 0 or 1 (makes it optional)    |
| `*`        | 0 or more                     |
| `+`        | 1 or more                     |
| `{n}`      | Exactly n times               |
| `{n,m}`    | Between n and m times         |
| `{n,}`     | n or more times               |
| `{,m}`     | Up to m times                 |

```bash
# Optional parentheses around area code
grep -E '^\(?[0-9]{3}\)? [0-9]{3}-[0-9]{4}$'

# Sentence: starts uppercase, ends with period
grep -E '^[[:upper:]][[:upper:][:lower:] ]*\.'

# Words separated by single spaces
grep -E '^([[:alpha:]]+ ?)+$'
```

---

## Grouping with `()`

Groups elements for quantifiers or alternation.

```bash
grep -E '(abc)+'        # "abc", "abcabc", etc.
grep -E '^(foo|bar)baz' # "foobaz" or "barbaz"
```

---

## Practical Patterns

### Validate / filter a list
```bash
# Find invalid phone numbers (invert match)
grep -Ev '^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$' phonelist.txt

# Find lines with only whitespace
grep -E '^\s*$' file.txt

# Find lines that are NOT blank
grep -E '\S' file.txt
```

### find with regex
`find -regex` requires the pattern to match the **entire path**, not just a substring — use `.*` on both ends.

```bash
# Find files with "weird" characters in name (not alphanumeric/dash/dot/slash)
find . -regex '.*[^-_./0-9a-zA-Z].*'

# find + ERE
find . -regextype egrep -regex '.*/[[:digit:]]{4}-[[:digit:]]{2}.*'
```

### locate with regex
```bash
locate --regex 'bin/(bz|gz|zip)'    # BRE
locate --regex 'bin/(bz|gz|zip)'    # ERE
```

### Search in less and vim
```bash
# less: press / then type pattern (ERE supported)
/^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$

# vim: uses BRE by default — escape { } ( )
/\([0-9]\{3\}\) [0-9]\{3\}-[0-9]\{4\}

# vim: enable ERE with \v (very magic mode)
/\v\([0-9]{3}\) [0-9]{3}-[0-9]{4}$

# Enable search highlighting in vim
:set hlsearch
```

### Search compressed files
```bash
zgrep -El 'regex|regular expression' /usr/share/man/man1/*.gz
```

---

## Cheat Sheet

```
.         any single character
^         start of line
$         end of line
[abc]     one of: a, b, or c
[^abc]    not any of: a, b, c
[a-z]     range (locale-dependent — prefer POSIX classes)
[[:alpha:]] POSIX class

# ERE only (grep -E):
?         0 or 1
*         0 or more
+         1 or more
{n,m}     n to m times
|         alternation (OR)
()        grouping
```

**Quote all patterns in single quotes.** Regex metacharacters like `*`, `?`, `|`, `(`, `)` are also shell special characters — unquoted, the shell mangles them before grep sees them.
