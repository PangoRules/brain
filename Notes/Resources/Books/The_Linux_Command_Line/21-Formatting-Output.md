# Chapter 21 — Formatting Output

## Quick Reference

| Command  | Description                                         |
| -------- | --------------------------------------------------- |
| `nl`     | Number lines (more control than `cat -n`)           |
| `fold`   | Wrap long lines at a specified width                |
| `fmt`    | Reflow and reformat paragraph text                  |
| `pr`     | Paginate text for printing (adds headers/footers)   |
| `printf` | Format and print data (C-style format strings)      |
| `groff`  | GNU document formatter — renders man pages          |

---

## nl — Number Lines

Like `cat -n` but with more control over what gets numbered and how.

```bash
nl file.txt                     # number non-blank lines (default)
nl -b a file.txt                # number ALL lines (like cat -n)
nl -b p'^[[:upper:]]' file.txt  # number only lines matching regex
nl -n rz file.txt               # right-justified with leading zeros: 000001
nl -w 3 -s ' ' file.txt         # 3-char wide number, space separator
```

**`-b` style values:** `a` = all lines, `t` = non-blank only (default), `n` = none, `p`*regex* = matching lines
**`-n` format values:** `ln` = left-justified, `rn` = right-justified (default), `rz` = right + leading zeros

**Logical page sections** — `nl` can number header/body/footer independently using embedded markup:

```
\:\:\:    start of page header
\:\:      start of page body
\:        start of page footer
```

These markers are stripped from the output. Useful when building reports via pipelines with `sed`.

---

## fold — Wrap Long Lines

Hard-wraps lines at a given width. Useful for terminals, emails, or fixed-width output.

```bash
fold -w 72 file.txt             # wrap at 72 chars (hard break — cuts words)
fold -w 72 -s file.txt          # wrap at last space before limit (word-safe)
echo "a long string..." | fold -w 40 -s
```

> `-s` is almost always what you want — without it `fold` splits mid-word.

---

## fmt — Reflow Paragraph Text

Joins and wraps lines to form clean paragraphs. Respects blank lines and indentation as paragraph separators.

```bash
fmt file.txt                    # reflow to default 75-char width
fmt -w 50 file.txt              # reflow to 50 chars
fmt -w 50 -c file.txt           # crown margin: preserve first two lines' indentation
fmt -s file.txt                 # split only — never join short lines (safe for code)
fmt -u file.txt                 # uniform spacing: 1 space between words, 2 after sentences
fmt -w 72 -p '# ' file.txt      # reformat only lines starting with '# ' (comments)
```

**`-p` prefix trick** — format just the comment lines in source code, leaving code untouched:
```bash
fmt -w 72 -p '# ' script.sh
```

---

## pr — Paginate for Printing

Adds headers, footers, and page breaks. Mostly useful before sending to a printer or `lpr`.

```bash
pr file.txt                     # paginate with default settings (66 lines/page, 72 cols)
pr -l 60 -w 80 file.txt         # 60 lines per page, 80 cols wide
pr -h "My Report" file.txt      # custom header text
pr -2 file.txt                  # two-column output
pr -n file.txt                  # number lines
pr -d file.txt                  # double-space output
```

**Multi-file columns:**
```bash
pr -m file1.txt file2.txt       # merge files side-by-side (like paste but with headers)
```

---

## printf — Format and Print Data

C-style formatted output. A shell builtin in both bash and zsh. Not pipeline-friendly (no stdin) — mainly used in scripts.

```bash
printf "format string" arg1 arg2 ...
```

### Conversion Specifiers

| Specifier | Type                    | Example                     |
| --------- | ----------------------- | --------------------------- |
| `%s`      | String                  | `printf "%s\n" "hello"`     |
| `%d`      | Signed decimal integer  | `printf "%d\n" 42`          |
| `%f`      | Floating-point          | `printf "%f\n" 3.14`        |
| `%o`      | Octal                   | `printf "%o\n" 8`  → `10`   |
| `%x`      | Hex (lowercase)         | `printf "%x\n" 255` → `ff`  |
| `%X`      | Hex (uppercase)         | `printf "%X\n" 255` → `FF`  |
| `%%`      | Literal `%`             | `printf "100%%\n"`           |

### Escape sequences

`\n` newline, `\t` tab, `\\` backslash, `\r` carriage return

### Width, precision, flags

```
%[flags][width][.precision]specifier
```

| Format     | Input  | Output       | Notes                        |
| ---------- | ------ | ------------ | ---------------------------- |
| `%d`       | 380    | `380`        | Basic integer                |
| `%05d`     | 380    | `00380`      | Zero-padded to width 5       |
| `%-10s`    | "hi"   | `hi        ` | Left-aligned in 10-char field|
| `%10s`     | "hi"   | `        hi` | Right-aligned (default)      |
| `%.5s`     | "hello world" | `hello` | Truncate string to 5 chars  |
| `%+d`      | 380    | `+380`       | Always show sign             |
| `%#x`      | 380    | `0x17c`      | Alternate form (hex prefix)  |
| `%10.3f`   | 3.14159| `     3.142` | Width 10, 3 decimal places   |

### Practical printf examples

```bash
# Tab-separated columns
printf "%s\t%s\t%s\n" Name Age City

# Formatted table rows in a script
printf "%-15s %5d %8.2f\n" "Alice" 30 1234.5

# Pad a number for filenames
printf "file_%03d.txt\n" 7       # → file_007.txt

# Hex/octal inspection
printf "%d %x %o\n" 255 255 255  # → 255 ff 377

# Generate repeated output (printf reuses format if more args than specifiers)
printf "item %d\n" 1 2 3 4 5
```

---

## groff — Document Formatting System

GNU implementation of `troff`. Most relevant use: **man pages are groff documents** rendered on the fly.

### View a raw man page source

```bash
zcat /usr/share/man/man1/ls.1.gz | head          # raw groff markup
man ls | head                                     # rendered output
```

### Render man pages manually

```bash
# Render to terminal (ASCII)
zcat /usr/share/man/man1/ls.1.gz | groff -mandoc -T ascii | less

# Render to PostScript, convert to PDF
zcat /usr/share/man/man1/ls.1.gz | groff -mandoc > ls.ps
ps2pdf ls.ps ls.pdf                              # requires ghostscript
```

### Output formats (`-T`)

| Flag       | Output format              |
| ---------- | -------------------------- |
| `-T ascii` | Plain ASCII (terminal)     |
| `-T utf8`  | UTF-8 terminal output      |
| `-T ps`    | PostScript (default)       |
| `-T pdf`   | PDF (modern groff)         |

```bash
# Direct PDF output (groff 1.22.4+ supports this natively)
zcat /usr/share/man/man1/ls.1.gz | groff -mandoc -T pdf > ls.pdf
```

### Macro packages (`-m`)

| Flag        | Used for                      |
| ----------- | ----------------------------- |
| `-mandoc`   | Man pages (auto-detects mdoc/man) |
| `-man`      | Traditional man pages         |
| `-ms`       | General documents             |
| `-me`       | Older general documents       |

### With tbl (table preprocessor)

```bash
# -t enables the tbl preprocessor for table markup
sort -k 1,1 distros.txt | sed -f distros-tbl.sed | groff -t -T ascii
sort -k 1,1 distros.txt | sed -f distros-tbl.sed | groff -t -T pdf > report.pdf
```

### Format converters

Fedora includes many format-to-format converters — find them:
```bash
ls /usr/bin/*[[:alpha:]]2[[:alpha:]]*
```
Common ones: `ps2pdf`, `pdf2ps`, `fig2dev`, `a2ps`, `rst2html`, `pandoc`

---

## Practical Combos

```bash
# Pretty-print a script with line numbers and word wrap
fold -w 80 -s script.sh | nl

# Reformat a long README to 80 cols
fmt -w 80 README | pr -h "README" | less

# Generate a zero-padded file list
printf "backup_%04d.tar.gz\n" $(seq 1 10)

# View any man page as PDF (opens in Okular on your KDE setup)
man -t ls | ps2pdf - ls.pdf && xdg-open ls.pdf

# Render a man page straight to PDF with groff
zcat /usr/share/man/man1/grep.1.gz | groff -mandoc -T pdf > grep.pdf
xdg-open grep.pdf
```

---

## Honest Usefulness Ranking

| Command | How often you'll actually use it        |
| ------- | --------------------------------------- |
| `printf` | Very often — scripts, formatting output |
| `groff`  | Occasionally — understanding man pages  |
| `fold`   | Sometimes — wrapping pipeline output    |
| `fmt`    | Sometimes — reformatting plain text     |
| `nl`     | Rarely — `cat -n` covers most cases    |
| `pr`     | Rarely — only for actual printing jobs  |
