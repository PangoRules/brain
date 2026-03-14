# Chapter 22 — Printing

## Quick Reference

| Command        | Description                                      |
| -------------- | ------------------------------------------------ |
| `lpr`          | Print files (Berkeley style)                     |
| `lp`           | Print files (System V style)                     |
| `lpstat`       | Show printer and queue status                    |
| `lpq`          | Show jobs in a printer queue                     |
| `lprm`         | Cancel print jobs (Berkeley)                     |
| `cancel`       | Cancel print jobs (System V)                     |
| `a2ps`         | Format anything as PostScript and print/save     |
| `pr`           | Paginate/format text before printing             |

---

## How Printing Works on Linux

**CUPS** (Common Unix Printing System) manages everything: drivers, queues, scheduling, and format conversion. **Ghostscript** acts as the PostScript/PDF rasterizer (RIP).

Every printer has a **print queue**. Jobs are spooled there and processed in order. CUPS also handles format conversion — it can accept plain text, PDF, PostScript, and image files.

**CUPS web interface** — manage printers, queues, and jobs in browser:
```
http://localhost:631
```

On Fedora you can also manage printers via **System Settings → Printers** in KDE.

---

## pr — Paginate Before Printing

Covered in Ch21, but used heavily before sending to `lpr`/`lp`.

```bash
ls /usr/bin | pr -3 -w 65         # 3-column, 65 chars wide
ls /usr/bin | pr -3 -w 65 | lpr   # paginate then print
```

Key options in a print context:

| Option          | Description                                     |
| --------------- | ----------------------------------------------- |
| `-l length`     | Page length in lines (default: 66 for US letter)|
| `-w width`      | Page width in chars (default: 72)               |
| `-h "text"`     | Custom header text                              |
| `-o offset`     | Left margin offset in chars                     |
| `-columns`      | Number of columns                               |
| `-d`            | Double-space output                             |
| `-t`            | Omit header/footer (useful when piping to a2ps) |
| `+first[:last]` | Print only a page range                         |

---

## lpr — Print Files (Berkeley Style)

```bash
lpr file.txt                    # print to default printer
lpr -P HPLaserJet file.txt      # print to named printer
lpr -# 3 file.txt               # 3 copies
lpr -p file.txt                 # pretty-print with shaded header

# Pipeline
ls /usr/bin | pr -3 | lpr
cat report.txt | lpr -P PDF     # "print" to cups-pdf virtual printer
```

---

## lp — Print Files (System V Style)

More options than `lpr`, especially for layout control.

```bash
lp file.txt                         # print to default printer
lp -d PrinterName file.txt          # named printer
lp -n 2 file.txt                    # 2 copies
lp -P 1,3,5-10 file.txt             # specific pages

# Layout options
lp -o landscape file.txt            # landscape orientation
lp -o fitplot image.jpg             # scale image to fit page
lp -o scaling=75 file.pdf           # scale to 75% of page
lp -o cpi=12 -o lpi=8 file.txt      # 12 chars/inch, 8 lines/inch
lp -o page-left=36 file.txt         # left margin (points; 72pt = 1 inch)

# Combine pr + lp for dense output
ls /usr/bin | pr -4 -w 90 -l 88 | lp -o page-left=36 -o cpi=12 -o lpi=8
```

---

## lpstat — Check Printer Status

```bash
lpstat -a                   # list all queues and whether they accept jobs
lpstat -d                   # show default printer name
lpstat -p                   # show status of all printers
lpstat -p PrinterName       # status of a specific printer
lpstat -s                   # summary: default, devices, queues
lpstat -t                   # full status report
lpstat -r                   # is the CUPS server running?
```

---

## lpq — View the Print Queue

```bash
lpq                         # queue for default printer
lpq -P PrinterName          # queue for a specific printer
lpq -a                      # all queues
```

Output shows job rank, owner, job ID, filename, and size. The job ID is what you need to cancel a job.

---

## lprm / cancel — Cancel Jobs

```bash
# Berkeley style
lprm 603                    # cancel job 603
lprm -                      # cancel all your jobs on default printer
lprm -P PrinterName 603     # cancel job on specific printer

# System V style
cancel 603                  # cancel job 603
cancel -a                   # cancel all your jobs
cancel -a PrinterName       # cancel all jobs on a printer
```

To get the job ID: `lpq` or `lpstat -t`

---

## a2ps — Anything to PostScript

Formats text (or other content) for printing with a polished layout — two pages per sheet, headers/footers, syntax highlighting for code. Useful for producing `.ps` files to inspect or convert.

```bash
# Install if missing
sudo dnf install a2ps

# Print to default printer
a2ps file.txt

# Save to PostScript file instead
a2ps -o output.ps file.txt
ls /usr/bin | pr -3 -t | a2ps -o listing.ps -L 66

# Key layout options
a2ps --columns=1 file.txt           # 1-up instead of default 2-up
a2ps -r file.txt                    # landscape
a2ps -R file.txt                    # portrait
a2ps -f 10 file.txt                 # 10pt font
a2ps -b "My Title" file.txt         # custom header
a2ps -u "DRAFT" file.txt            # watermark/underlay text
a2ps -n 2 file.txt                  # 2 copies
a2ps --guess file.txt               # show what type a2ps thinks the file is
a2ps --list=defaults                # show current default settings
```

**Convert PostScript to PDF:**
```bash
ps2pdf output.ps output.pdf         # ghostscript (usually pre-installed)
xdg-open output.pdf                 # opens in Okular on your KDE setup
```

---

## cups-pdf — Print to PDF

CUPS can create a virtual "PDF printer" that saves print jobs as PDFs instead of sending them to hardware.

```bash
# Install
sudo dnf install cups-pdf

# After install, a "PDF" printer appears in lpstat -a
# Jobs sent to it land in ~/PDF/ by default
lpr -P PDF file.txt
```

---

## Practical Combos

```bash
# Check what printers are configured
lpstat -a
lpstat -s

# Print a directory listing, 3 columns, to default printer
ls /usr/bin | pr -3 | lpr

# Print only pages 2-4 of a file
lp -P 2-4 document.txt

# Produce a syntax-highlighted PostScript of a script, then view as PDF
a2ps -o script.ps myscript.sh
ps2pdf script.ps script.pdf
xdg-open script.pdf

# Cancel all your stuck print jobs
cancel -a

# Watch the queue
watch -n 2 lpq
```

---

## Honest Usefulness Ranking

| Command        | Reality                                                 |
| -------------- | ------------------------------------------------------- |
| `lpstat`       | Useful — quick printer/queue check from terminal        |
| `lpq` / `cancel` | Useful — cancel stuck jobs without opening a GUI      |
| `lp` / `lpr`  | Occasionally useful for scripted printing               |
| `pr` (for print) | Mostly superseded by `lp -o` options or PDF workflows|
| `a2ps`         | Niche — nice for code printouts or quick `.ps` files    |
| History section | Context for why commands are the way they are          |

> Most day-to-day printing on a KDE desktop goes through **application → Print dialog → CUPS**. The command line tools shine for **scripted/automated printing**, **cancelling stuck jobs**, and **batch PDF generation**.
