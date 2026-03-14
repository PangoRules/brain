# Chapter 20 — Text Processing

## Quick Reference

| Command  | Description                                        |
| -------- | -------------------------------------------------- |
| `cat`    | Concatenate / view files, reveal hidden characters |
| `sort`   | Sort lines of text                                 |
| `uniq`   | Remove or count adjacent duplicate lines           |
| `cut`    | Extract columns by character position or field     |
| `paste`  | Merge files side-by-side (column-wise)             |
| `join`   | Join two files on a shared key field               |
| `comm`   | Compare two sorted files line by line              |
| `diff`   | Show differences between files                     |
| `patch`  | Apply a diff to a file                             |
| `tr`     | Transliterate or delete characters                 |
| `sed`    | Stream editor — filter and transform text          |
| `aspell` | Interactive spell checker                          |

---

## cat (revisited)

```bash
cat -A file.txt     # show non-printing chars: ^I = tab, $ = end of line
cat -n file.txt     # number all lines
cat -s file.txt     # squeeze multiple blank lines into one
cat -ns file.txt    # combine: number + squeeze
```

`-A` is useful for spotting invisible problems: trailing spaces, Windows `\r\n` line endings (`^M$`), or tabs vs. spaces.

**DOS → Unix line endings:**
```bash
tr -d '\r' < dos_file.txt > unix_file.txt
# or
sed -i 's/\r//' file.txt
```

---

## sort

```bash
sort file.txt                   # alphabetical
sort -r file.txt                # reverse
sort -n file.txt                # numeric (treats values as numbers, not strings)
sort -nr file.txt               # reverse numeric
sort -f file.txt                # case-insensitive
sort -u file.txt                # sort + remove duplicates
sort file1.txt file2.txt > merged.txt   # merge and sort multiple files
```

### Sorting by field (`-k`)

Fields are whitespace-delimited by default. `-k` specifies which field(s) to sort on.

```bash
sort -k 5 file.txt              # sort by field 5 (to end of line)
sort -k 1,1 file.txt            # sort by field 1 ONLY (not to end of line)
sort -k 2n file.txt             # field 2, numeric
sort -k 1,1 -k 2n file.txt     # primary: field 1 alpha; secondary: field 2 numeric
```

**Sort `ls -l` output by file size (field 5):**
```bash
ls -l /usr/bin | sort -nrk 5 | head
```

**Find top disk users:**
```bash
du -s /usr/share/* | sort -nr | head
```

### Custom delimiter (`-t`)

```bash
sort -t ':' -k 7 /etc/passwd    # sort passwd by shell (field 7)
sort -t ',' -k 2 data.csv       # sort CSV by second column
```

### Field offsets (for awkward date formats)

`-k field.char` lets you start sorting mid-field. For dates in MM/DD/YYYY format:

```bash
sort -k 3.7nbr -k 3.1nbr -k 3.4nbr file.txt   # year, then month, then day
```

---

## uniq

**Requires sorted input** — only removes adjacent duplicates.

```bash
sort file.txt | uniq             # remove duplicates
sort file.txt | uniq -c          # count occurrences (most useful)
sort file.txt | uniq -d          # show only duplicated lines
sort file.txt | uniq -u          # show only unique lines (no duplicates)
sort file.txt | uniq -i          # case-insensitive
```

**Find most common words in a file:**
```bash
tr -s ' ' '\n' < file.txt | sort | uniq -c | sort -nr | head
```

> `sort -u` is a shortcut for `sort | uniq` but you lose the `-c` count option.

---

## cut — Extract Columns

```bash
cut -c 1-5 file.txt             # characters 1 through 5
cut -c 7-10 file.txt            # characters 7 through 10
cut -c 23- file.txt             # character 23 to end of line
```

**By field (tab-delimited by default):**
```bash
cut -f 1 file.txt               # first tab-delimited field
cut -f 1,3 file.txt             # fields 1 and 3
cut -f 2-4 file.txt             # fields 2 through 4
```

**Custom delimiter:**
```bash
cut -d ':' -f 1 /etc/passwd     # usernames from passwd
cut -d ':' -f 1,7 /etc/passwd   # username and shell
cut -d ',' -f 2 data.csv        # second CSV column
```

**`--complement`** — extract everything *except* the specified fields:
```bash
cut -f 2 --complement file.txt  # all fields except field 2
```

> `cut` requires consistent delimiters. Use `expand` to convert tabs to spaces first if needed, then use `-c`.

---

## paste — Merge Files Side by Side

Combines files column-by-column (opposite of `cut`).

```bash
paste file1.txt file2.txt           # merge side-by-side, tab-separated
paste -d ',' file1.txt file2.txt    # use comma as delimiter
paste -s file.txt                   # paste all lines of one file into one line
```

**Practical example — reorder columns:**
```bash
sort -k 3 data.txt > by-date.txt
cut -f 1,2 by-date.txt > names.txt
cut -f 3   by-date.txt > dates.txt
paste dates.txt names.txt           # date first, then name + version
```

---

## join — Join Files on a Shared Key

Like a SQL JOIN — combines fields from two files that share a key column. **Both files must be sorted on the key field.**

```bash
join file1.txt file2.txt            # join on first field (default)
join -1 2 -2 1 file1.txt file2.txt  # join on field 2 of file1 and field 1 of file2
```

---

## comm — Compare Sorted Files

Produces three columns: unique to file1 | unique to file2 | common to both.

```bash
comm file1.txt file2.txt        # all three columns
comm -12 file1.txt file2.txt    # suppress col 1 & 2 → show only common lines
comm -23 file1.txt file2.txt    # suppress col 2 & 3 → show only lines unique to file1
comm -13 file1.txt file2.txt    # suppress col 1 & 3 → show only lines unique to file2
```

---

## diff — Show File Differences

```bash
diff file1.txt file2.txt        # default (terse) format
diff -c file1.txt file2.txt     # context format (3 lines of context)
diff -u file1.txt file2.txt     # unified format (most common, used by git)
diff -r dir1/ dir2/             # recursive directory diff
```

**Unified format markers:**
- ` ` (space) — unchanged context line
- `-` — line removed from file1
- `+` — line added in file2

**Create a patch file (GNU recommended format):**
```bash
diff -Naur old_file new_file > changes.patch
diff -Naur old_dir/ new_dir/ > changes.patch    # recursive
```

---

## patch — Apply a diff

```bash
patch < changes.patch           # applies patch (filenames come from the diff header)
patch -p1 < changes.patch       # strip 1 leading path component (common for source trees)
patch --dry-run < changes.patch # test without applying
patch -R < changes.patch        # reverse / undo a patch
```

---

## tr — Transliterate or Delete Characters

Operates on stdin only — no filenames.

```bash
echo "hello" | tr a-z A-Z           # lowercase to uppercase
echo "hello" | tr [:lower:] [:upper:]  # same, POSIX-safe
echo "hello" | tr -d 'aeiou'        # delete vowels
echo "  too   many   spaces  " | tr -s ' '  # squeeze repeated spaces to one
tr -d '\r' < dos.txt > unix.txt     # strip Windows carriage returns
```

**Char set formats:** enumerated list `ABC`, range `A-Z` (locale-sensitive), POSIX class `[:upper:]`.

**Useful tricks:**
```bash
# Count words (one per line)
tr -s ' ' '\n' < file.txt | wc -l

# ROT13 encoding/decoding (same command both ways)
echo "secret" | tr a-zA-Z n-za-mN-ZA-M
```

---

## sed — Stream Editor

Processes text line by line. Applies commands to matching lines.

```bash
sed 's/old/new/' file.txt           # substitute first match per line
sed 's/old/new/g' file.txt          # substitute all matches per line (global)
sed 's/old/new/gi' file.txt         # global + case-insensitive
sed -i 's/old/new/g' file.txt       # edit file in-place (GNU sed)
sed -i.bak 's/old/new/g' file.txt   # in-place with backup
```

> Always test without `-i` first. `-i` modifies the file directly.

### Addresses — target specific lines

```bash
sed '3s/old/new/' file.txt          # only line 3
sed '1,5s/old/new/' file.txt        # lines 1 through 5
sed '$s/old/new/' file.txt          # last line only
sed '/pattern/s/old/new/' file.txt  # lines matching a regex
sed '/pattern/!s/old/new/' file.txt # lines NOT matching
sed -n '1,5p' file.txt              # print lines 1-5 (-n suppresses default output)
sed -n '/SUSE/p' file.txt           # print only matching lines (like grep)
```

### Common sed commands

| Command | Description                               |
| ------- | ----------------------------------------- |
| `s`     | Substitute (most used)                    |
| `d`     | Delete line                               |
| `p`     | Print line (use with `-n`)                |
| `q`     | Quit after this line                      |
| `a`     | Append text after line                    |
| `i`     | Insert text before line                   |
| `=`     | Print current line number                 |
| `y/x/y/`| Transliterate (like `tr`, same length sets)|

```bash
sed '/^#/d' file.txt                # delete comment lines
sed '/^$/d' file.txt                # delete blank lines
sed -n '/start/,/end/p' file.txt    # print between two patterns
sed '1i\Header line' file.txt       # insert a line before line 1
```

### Back references in substitution

Capture groups with `\(` `\)` in BRE, reference with `\1`, `\2`, etc.

```bash
# Reformat date from MM/DD/YYYY to YYYY-MM-DD
sed 's/\([0-9]\{2\}\)\/\([0-9]\{2\}\)\/\([0-9]\{4\}\)$/\3-\1-\2/' file.txt

# Swap two words
echo "foo bar" | sed 's/\(\w\+\) \(\w\+\)/\2 \1/'
```

### Multiple commands

```bash
sed -e 's/foo/bar/' -e 's/baz/qux/' file.txt   # -e chains commands
sed 's/foo/bar/; s/baz/qux/' file.txt           # semicolon separator
sed -f script.sed file.txt                       # run commands from a script file
```

### sed script file example

```bash
# distros.sed
# Comment lines start with #
1 i\
Linux Distributions Report\

s/\([0-9]\{2\}\)\/\([0-9]\{2\}\)\/\([0-9]\{4\}\)$/\3-\1-\2/
y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/
```
```bash
sed -f distros.sed distros.txt
```

---

## aspell — Spell Checker

```bash
aspell check file.txt           # interactive spell check
aspell -H check file.txt        # HTML mode (skips tags, checks content)
aspell list < file.txt          # print misspelled words to stdout (non-interactive)
aspell dump master | grep foo   # search the word list
```

In interactive mode: number keys select replacements, `i` ignores, `a` adds to personal dictionary, `r` lets you type a replacement, `x` exits.

> Install on Fedora if missing: `sudo dnf install aspell aspell-en`

---

## Practical Combos

```bash
# Top 10 largest files in /usr/share
du -s /usr/share/* | sort -nr | head

# List unique shells in use on the system
cut -d ':' -f 7 /etc/passwd | sort | uniq -c | sort -nr

# Find duplicate lines across two log files
cat log1.txt log2.txt | sort | uniq -d

# Strip comments and blank lines from a config
sed '/^#/d; /^$/d' /etc/some.conf

# Rename a variable across all source files (dry run first)
grep -rl 'old_name' ~/projects/
sed -i 's/old_name/new_name/g' $(grep -rl 'old_name' ~/projects/)

# Convert a CSV column to a sorted unique list
cut -d ',' -f 2 data.csv | sort | uniq

# Check what changed between two config file versions
diff -u /etc/old.conf /etc/new.conf | less
```

---

## sed vs. tr vs. awk

| Need                        | Use       |
| --------------------------- | --------- |
| Char-by-char substitution   | `tr`      |
| Line-based search/replace   | `sed`     |
| Column/field manipulation   | `awk`     |
| Complex logic + text        | `awk` / `perl` |
