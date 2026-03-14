# ⚙️ Zsh Intermediate Shell Exercises

## 1. Argument Parser
Support flags like:
./script.sh -f file -v

**Hints:**
- `getopts`
- `$OPTARG`

---

## 2. Log Analyzer
Count ERROR/WARNING/INFO occurrences.

**Hints:**
- `grep -c`
- pipes

---

## 3. Disk Usage Reporter
Find largest 5 directories in a path.

**Hints:**
- `du -sh`
- `sort -h`
- `head`

---

## 4. User Home Scanner
Loop through /home and report sizes.

**Hints:**
- `for dir in /home/*`
- `du -sh`

---

## 5. Process Monitor
List top 5 memory-consuming processes.

**Hints:**
- `ps aux`
- `sort -k`
- `head`

---

## 6. Temporary File Cleaner
Delete files older than X days.

**Hints:**
- `find`
- `-mtime`
- `-exec`

---

## 7. Permission Fixer
Find files not owned by current user.

**Hints:**
- `find`
- `-user`
- `whoami`

---

## 8. Password Strength Checker
Validate input length + characters.

**Hints:**
- regex `[[ =~ ]]`
- `${#var}`

---

## 9. Batch Renamer
Rename files to lowercase.

**Hints:**
- `${var:l}` (zsh lowercase)
- `mv`

---

## 10. Cron Log Parser
Extract entries from today only.

**Hints:**
- `date`
- `grep`
- `awk`