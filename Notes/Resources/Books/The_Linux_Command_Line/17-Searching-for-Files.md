# Chapter 17 — Searching for Files

## Quick Reference

| Command    | Description                                          |
| ---------- | ---------------------------------------------------- |
| `locate`   | Fast filename search via pre-built database          |
| `find`     | Deep search by name, type, size, time, perms, etc.   |
| `xargs`    | Build argument lists from stdin and run a command    |
| `touch`    | Create empty files or update timestamps              |
| `stat`     | Show detailed file/filesystem metadata               |

---

## locate — Fast Name Search

Searches a pre-built database of all paths on the system. Extremely fast but only as fresh as the last database update.

```bash
locate bin/zip                  # find paths containing "bin/zip"
locate zip | grep bin           # pipe into grep for tighter filtering
locate -i readme                # case-insensitive match
```

**On your Fedora setup:** `locate` is symlinked to `plocate` (a faster, modern replacement for `mlocate`). The database is rebuilt daily via a systemd timer, not cron:

```bash
systemctl list-timers | grep plocate   # next scheduled run
sudo updatedb                           # force an immediate rebuild
```

> Very recent files won't show up until after the next `updatedb` run.

---

## find — Search by Anything

Walks a directory tree live. Slower than `locate` but searches by name, type, size, modification time, permissions, owner, and more.

```bash
find ~                          # list everything under home
find ~ | wc -l                  # count all files and dirs
find /etc -type f               # only regular files
find /etc -type d               # only directories
```

### Tests

**By type:**

| Flag      | Matches              |
| --------- | -------------------- |
| `-type f` | Regular files        |
| `-type d` | Directories          |
| `-type l` | Symbolic links       |
| `-type b` | Block devices        |
| `-type c` | Character devices    |

**By name:**
```bash
find ~ -name "*.jpg"            # exact case
find ~ -iname "*.jpg"           # case-insensitive
```

> Always quote the pattern — otherwise the shell expands the glob before `find` sees it.

**By size:**
```bash
find ~ -type f -size +1G        # larger than 1 GB
find ~ -type f -size -100k      # smaller than 100 KB
find ~ -type f -size 0          # exactly empty
```

Size units: `b` (512-byte blocks), `c` (bytes), `k` (KiB), `M` (MiB), `G` (GiB). `+` = greater than, `-` = less than, no prefix = exact.

**By modification time:**
```bash
find ~ -mmin -30                # modified in the last 30 minutes
find ~ -mtime -7                # modified in the last 7 days
find ~ -newer reference.txt     # modified more recently than a file
```

**By permissions:**
```bash
find ~ -perm 0644               # exact permissions match
find /etc -perm -0004           # world-readable (at least)
```

**By owner / group:**
```bash
find /home -user pango
find /var -nouser               # files with no valid owner (orphaned)
```

**Other useful tests:**
```bash
find ~ -empty                   # empty files and directories
find ~ -type f -name "*.bak"    # specific extension
```

### Operators

Tests combine with logical operators (implicit `-and` between adjacent tests):

```bash
find ~ -type f -name "*.log"                    # implicit -and
find ~ \( -name "*.jpg" -o -name "*.png" \)     # -or
find ~ -type f -not -perm 0600                  # -not / !
```

Find files with bad permissions (not 0600) **or** directories with bad permissions (not 0700):

```bash
find ~ \( -type f -not -perm 0600 \) -or \( -type d -not -perm 0700 \)
```

> `\( \)` must be escaped — parens have special meaning to the shell.

Short-circuit evaluation: in `expr1 -and expr2`, if expr1 is false, expr2 is never evaluated (and vice versa for `-or`). Order of tests matters for performance.

### Actions

**Predefined:**
```bash
find ~ -name "*.bak" -print     # print path (default if no action given)
find ~ -name "*.bak" -ls        # like ls -dils on each match
find ~ -name "*.bak" -delete    # delete matches
find ~ -name "*.bak" -quit      # stop after first match
```

> **Always** test with `-print` before using `-delete`. `-depth` is automatically applied with `-delete`.

**Execute a command on each match:**
```bash
# ';' → spawn a new process per file
find ~ -type f -name "foo*" -exec ls -l '{}' ';'

# '+' → batch all results into one command call (faster)
find ~ -type f -name "foo*" -exec ls -l '{}' +
```

**Interactive execution (prompt before each):**
```bash
find ~ -type f -name "foo*" -ok rm '{}' ';'
```

### Options (scope control)

```bash
find ~ -maxdepth 2              # don't recurse deeper than 2 levels
find ~ -mindepth 1              # skip the top directory itself
find / -mount                   # don't cross into other filesystems
```

> `-mount` is handy on your setup: avoids descending into Docker overlay mounts (`/var/lib/docker`), btrfs subvolumes, or the ProtonVPN tunnel interface's pseudo-paths.

---

## xargs — Build Argument Lists from stdin

Converts piped stdin into arguments for a command. More efficient than `-exec ... ';'` for large result sets.

```bash
find ~ -type f -name "*.log" | xargs ls -lh
find ~ -type f -name "*.txt" | xargs grep "TODO"
```

**Handle filenames with spaces (always safe habit):**
```bash
find ~ -iname "*.jpg" -print0 | xargs --null ls -lh
# -print0 uses null as separator; --null/-0 reads null-separated input
```

**Check the max argument limit:**
```bash
xargs --show-limits
```

---

## touch — Create Files / Update Timestamps

```bash
touch newfile.txt               # create empty file, or update timestamps if it exists
touch -t 202501011200 file.txt  # set specific timestamp (YYYYMMDDhhmm)
touch -r reference.txt target   # copy timestamps from one file to another
```

**Practical use with find:**
```bash
touch checkpoint                        # create reference file now
# ... do some work ...
find ~ -newer checkpoint                # see what changed since then
```

**Bulk create files (brace expansion):**
```bash
mkdir -p playground/dir-{001..100}
touch playground/dir-{001..100}/file-{A..Z}    # 2600 files instantly
```

---

## stat — Detailed File Metadata

Shows everything the filesystem knows about a file: size, inode, permissions, all three timestamps (Access, Modify, Change).

```bash
stat file.txt
stat /home/pango                # works on directories too
stat -f /                       # filesystem stats (btrfs on your /)
```

```
  File: file.txt
  Size: 1024       Blocks: 8         IO Block: 4096   regular file
Device: fd00h/...  Inode: 123456     Links: 1
Access: (0644/-rw-r--r--)  Uid: (1000/pango)  Gid: (1000/pango)
Access: 2026-02-22 10:00:00
Modify: 2026-02-22 09:55:00
Change: 2026-02-22 09:55:00
```

- **Access** — last read
- **Modify** — last content change (what `-mtime`/`-mmin` test against)
- **Change** — last metadata change (perms, owner, etc.)

---

## Practical Combos

```bash
# Find large files eating space on your 1.8 TB btrfs drive
find / -mount -type f -size +1G 2>/dev/null

# Find files modified in the last 24 hours (exclude Docker/VPN noise)
find /home /etc /opt -mount -mtime -1

# Find and remove stale .bak files (dry-run first, then delete)
find ~ -type f -name "*.bak" -print
find ~ -type f -name "*.bak" -delete

# Fix permissions on a directory tree
find ~/projects -type d -exec chmod 755 '{}' +
find ~/projects -type f -exec chmod 644 '{}' +

# Search file contents across all .md notes
find . -name "*.md" -print0 | xargs --null grep -l "networking"
```
