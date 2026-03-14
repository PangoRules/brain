# Chapter 18 — Archiving and Backup

## Quick Reference

| Command  | Description                                           |
| -------- | ----------------------------------------------------- |
| `gzip`   | Compress/decompress files (.gz)                       |
| `bzip2`  | Higher compression, slower (.bz2)                     |
| `xz`     | Best compression ratio, slower (.xz) — common on Fedora |
| `zstd`   | Fast compression, great ratio (.zst) — used by btrfs/RPM |
| `tar`    | Archive files (often combined with compression)       |
| `zip`    | Archive + compress, Windows-compatible (.zip)         |
| `rsync`  | Efficient local/remote file sync                      |

---

## Compression

### gzip

Replaces the original file with a `.gz` version in-place.

```bash
gzip file.txt               # → file.txt.gz (original gone)
gunzip file.txt.gz          # → file.txt restored
gzip -k file.txt            # keep original
gzip -9 file.txt            # max compression (slower)
gzip -l file.txt.gz         # list compression stats
gzip -t file.txt.gz         # test integrity
```

**Without touching files (pipe-friendly):**
```bash
ls -l /etc | gzip > listing.gz      # compress stdout
zcat listing.gz | less              # view without decompressing
zless listing.gz                    # same thing
```

### bzip2

Same interface as gzip, higher compression, slower. Uses `.bz2`.

```bash
bzip2 file.txt              # → file.txt.bz2
bunzip2 file.txt.bz2
bzip2 -k file.txt           # keep original
bzcat file.txt.bz2 | less
```

### xz

Best compression ratio of the three. Common on Fedora (kernel tarballs, man pages). Uses `.xz`.

```bash
xz file.txt                 # → file.txt.xz
unxz file.txt.xz
xz -k file.txt              # keep original
xz -l file.txt.xz           # stats
xzcat file.txt.xz | less
```

### zstd — Modern Fast Compression

Not in the book, but relevant: `zstd` is used by btrfs (your `/` filesystem), Fedora RPM packages, and Docker. Best speed-to-ratio tradeoff.

```bash
zstd file.txt               # → file.txt.zst
zstd -d file.txt.zst        # decompress
zstd -19 file.txt           # max compression
zstdcat file.txt.zst | less
```

> Don't double-compress. Applying gzip to a `.jpg`, `.mp3`, or already-compressed file wastes time and often makes it *larger*.

---

## Archiving

### tar

The standard Unix archive tool. Mode goes first, no leading dash needed.

| Mode | Action                        |
| ---- | ----------------------------- |
| `c`  | Create archive                |
| `x`  | Extract archive               |
| `t`  | List contents                 |
| `r`  | Append files to archive       |

**Create:**
```bash
tar cf archive.tar dir/             # create (uncompressed)
tar czf archive.tar.gz dir/         # create + gzip
tar cjf archive.tar.bz2 dir/        # create + bzip2
tar cJf archive.tar.xz dir/         # create + xz
tar --zstd -cf archive.tar.zst dir/ # create + zstd
```

**List:**
```bash
tar tf archive.tar.gz               # list contents
tar tvf archive.tar.gz              # verbose (like ls -l)
```

**Extract:**
```bash
tar xf archive.tar.gz               # extract here
tar xf archive.tar.gz -C /tmp/      # extract to specific dir
tar xf archive.tar.gz path/to/file  # extract single file
tar xf archive.tar.gz --wildcards 'dir-*/file-A'
```

> tar strips leading `/` from absolute paths by default — extracted files land relative to CWD, not root. This is a safety feature.

**Append with find (incremental-style):**
```bash
find ~/projects -name "*.py" -exec tar rf backup.tar '{}' '+'
find ~/projects -newer checkpoint -exec tar rf backup.tar '{}' '+'
```

**Pipe across network via SSH (no temp file needed):**
```bash
ssh remote-sys 'tar cf - Documents' | tar xf -
# ← creates archive on remote, streams it, extracts locally
```

**Use stdin for file list:**
```bash
find . -name "*.log" | tar czf logs.tar.gz -T -
# -T - reads filenames from stdin
```

### zip / unzip

Use for Windows interop. On Linux, prefer tar+gzip.

```bash
zip -r archive.zip dir/             # recursive
unzip archive.zip                   # extract
unzip -l archive.zip                # list contents
unzip archive.zip path/to/file      # extract one file
find . -name "*.txt" | zip -@ docs.zip  # pipe filenames in
unzip -p archive.zip | less         # pipe output to stdout
```

> Unlike tar, `zip` updates an existing archive rather than replacing it.

---

## rsync — Sync Files Efficiently

Only transfers what changed. Much faster than `cp` for repeated syncs.

```bash
rsync -av source/ destination/      # sync contents of source into dest
rsync -av source destination/       # sync source dir itself into dest
```

**`source/` vs `source` — trailing slash matters:**
```bash
rsync -av ~/docs/ /backup/docs/     # copies contents of docs/
rsync -av ~/docs  /backup/          # copies the docs/ dir itself into /backup/
```

### Key Flags

| Flag            | Effect                                              |
| --------------- | --------------------------------------------------- |
| `-a`            | Archive: recursive + preserve perms/times/symlinks  |
| `-v`            | Verbose                                             |
| `-n`/`--dry-run`| Show what would happen, don't do it                |
| `--delete`      | Remove files in dest that no longer exist in source |
| `--exclude`     | Skip matching paths                                 |
| `--progress`    | Show per-file progress                              |
| `-z`            | Compress during transfer                            |

### Local Backup

```bash
# Dry run first — always
rsync -avn --delete ~/projects/ /mnt/backup/projects/

# Real run
rsync -av --delete ~/projects/ /mnt/backup/projects/
```

### Remote Backup over SSH

```bash
rsync -av --delete --rsh=ssh ~/projects/ pango@remote-sys:/backup/projects/

# Or shorthand (ssh is default in modern rsync):
rsync -av --delete ~/projects/ pango@remote-sys:/backup/projects/
```

### Practical Patterns for Your Setup

```bash
# Back up home, skip Docker volumes and cache
rsync -av --delete \
  --exclude='.cache/' \
  --exclude='docker/' \
  /home/pango/ /mnt/backup/pango/

# Sync a local project to a remote dev server
rsync -av --delete --exclude='.git/' ~/dev/myapp/ pango@devbox:~/myapp/

# Mirror from an rsync server
rsync -av rsync://mirror.example.com/fedora/ ~/fedora-mirror/

# Archive + transfer without a temp file
tar czf - ~/important/ | ssh remote-sys 'cat > ~/backup.tar.gz'
```

---

## Compression Cheat Sheet

| Format   | Ratio  | Speed  | Use case                          |
| -------- | ------ | ------ | --------------------------------- |
| `.gz`    | Good   | Fast   | General purpose, universal compat |
| `.bz2`   | Better | Slower | When size matters more than speed |
| `.xz`    | Best   | Slow   | Distro tarballs, archiving        |
| `.zst`   | Good+  | Fastest| Real-time, btrfs, RPMs, Docker    |
| `.zip`   | Good   | Fast   | Windows interop                   |
