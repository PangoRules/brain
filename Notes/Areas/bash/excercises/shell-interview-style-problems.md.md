# 💼 Zsh Interview-Style Problems

## 1. Reverse File Lines
Print file contents reversed.

**Hints:**
- `tac`
- `awk`

---

## 2. Detect Duplicate Lines
Print duplicate lines only.

**Hints:**
- `sort`
- `uniq -d`

---

## 3. Top 3 Most Common Words
In a text file.

**Hints:**
- `tr`
- `sort`
- `uniq -c`

---

## 4. Count Files by Extension
Print summary like:
.txt: 4
.log: 10

**Hints:**
- `for`
- parameter expansion
- associative arrays (zsh)

---

## 5. Find Broken Symlinks
List all broken symbolic links.

**Hints:**
- `find`
- `-type l`
- `-exec`

---

## 6. Validate IP Address
Check if string is valid IPv4.

**Hints:**
- regex
- `[[ =~ ]]`

---

## 7. Check Port Availability
Check if port is in use.

**Hints:**
- `ss -tuln`
- `grep`

---

## 8. Detect Zombie Processes
Print zombie processes only.

**Hints:**
- `ps aux`
- state column

---

## 9. Compare Two Directories
List files present in one but not the other.

**Hints:**
- `diff`
- `comm`

---

## 10. Simulate Tail -f
Continuously monitor file for changes.

**Hints:**
- `while true`
- `sleep`
- `tail`