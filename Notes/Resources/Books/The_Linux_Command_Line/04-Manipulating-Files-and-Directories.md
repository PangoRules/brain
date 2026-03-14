
| Command | What it does |
|---------|-------------|
| `cp` | Copy |
| `mv` | Move / rename |
| `mkdir` | Create directory |
| `rm` | Remove |
| `ln` | Create links |

### Wildcards (globbing)

| Pattern | Matches |
|---------|---------|
| `*` | Any characters |
| `?` | Any single char |
| `[abc]` | Any char in set |
| `[!abc]` | Any char NOT in set |
| `[[:upper:]]` | Uppercase letter |
| `[[:lower:]]` | Lowercase letter |
| `[[:digit:]]` | Number |
| `[[:alnum:]]` | Alphanumeric |

> Tip: test wildcards with `ls` first, then swap `ls` for `rm`/`cp`/`mv`

### cp

| Example | Effect |
|---------|--------|
| `cp file1 file2` | Copy (overwrites silently!) |
| `cp -i file1 file2` | Interactive -- ask before overwrite |
| `cp -r dir1 dir2` | Recursive copy (required for dirs) |
| `cp -u *.html dest/` | Only copy newer files |
| `cp -a dir1 dir2` | Archive -- preserve all attributes |

### mv

| Example | Effect |
|---------|--------|
| `mv file1 file2` | Rename |
| `mv file1 dir1/` | Move into dir |
| `mv -i file1 file2` | Interactive |

### rm

| Example | Effect |
|---------|--------|
| `rm file` | Delete (no undo!) |
| `rm -i file` | Ask first |
| `rm -r dir` | Recursive delete |
| `rm -rf dir` | Force, no prompts |

> **No undelete in Linux.** `rm * .html` (accidental space) deletes everything.

### ln -- links

| Command | Creates |
|---------|---------|
| `ln file link` | Hard link (same inode, can't cross filesystems, no dirs) |
| `ln -s target link` | Symbolic link (can link anything) |