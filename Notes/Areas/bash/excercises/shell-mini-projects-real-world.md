# 🚀 Real-World Automation Mini Projects (Zsh)

## 1. Automated Backup System
Backup + rotate old backups.

**Hints:**
- `tar`
- `find -mtime`
- date formatting

---

## 2. System Health Dashboard
Display CPU, memory, disk.

**Hints:**
- `uptime`
- `free`
- `df -h`

---

## 3. Log Rotation Utility
Archive large log files.

**Hints:**
- `gzip`
- `mv`
- `stat`

---

## 4. Developer Project Bootstrapper
Create folder structure + README.

**Hints:**
- `mkdir -p`
- `cat <<EOF`

---

## 5. Git Status Reporter
Scan multiple repos and show branch + status.

**Hints:**
- `for dir in */`
- `git status`
- `git branch`

---

## 6. SSH Host Scanner
Ping multiple hosts and report availability.

**Hints:**
- `for`
- `ping -c 1`
- `$?`

---

## 7. File Integrity Checker
Compare checksums over time.

**Hints:**
- `sha256sum`
- `diff`

---

## 8. Disk Usage Email Alert
Alert if disk > 80%.

**Hints:**
- `df`
- `awk`
- comparison with `(( ))`

---

## 9. Simple Deployment Script
Pull repo, build, restart service.

**Hints:**
- `git pull`
- `systemctl`
- `set -e`

---

## 10. Local Service Watchdog
Restart service if it crashes.

**Hints:**
- `while true`
- `pgrep`
- `sleep`