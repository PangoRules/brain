
| Command | What it does |
|---------|-------------|
| `file name` | Determine file type |
| `less name` | View file with paging |

### ls options

| Flag | Effect |
|------|--------|
| `-a` | Show hidden files |
| `-l` | Long format |
| `-h` | Human-readable sizes |
| `-t` | Sort by modification time |
| `-S` | Sort by size |
| `-r` | Reverse sort |
| `-d` | Show directory itself, not contents |
| `-F` | Append type indicator (`/` for dirs) |

Combine freely: `ls -lth`, `ls -lt --reverse`

### less navigation

| Key | Action |
|-----|--------|
| `Space` / `PgDn` | Forward one page |
| `b` / `PgUp` | Back one page |
| `G` | End of file |
| `g` / `1G` | Beginning of file |
| `/text` | Search forward |
| `n` | Next search match |
| `q` | Quit |

### Long listing format
```
-rw-r--r-- 1 owner group 32059 2017-04-03 11:05 filename
```
First char: `-` = file, `d` = dir, `l` = symlink

### Key directories

| Dir | Purpose |
|-----|---------|
| `/bin` | Essential binaries |
| `/etc` | System config (text files) |
| `/home` | User home dirs |
| `/var/log` | Log files |
| `/tmp` | Temp files |
| `/usr/bin` | User programs |
| `/usr/local` | Locally compiled software |
| `/proc` | Virtual fs -- kernel peephole |

- **Symlink** = pointer to another file (`l` type, shown with `->`)