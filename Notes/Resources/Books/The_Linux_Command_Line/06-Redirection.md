
### Redirection operators

| Operator | Effect |
|----------|--------|
| `>` | Stdout to file (overwrites!) |
| `>>` | Stdout append to file |
| `2>` | Stderr to file |
| `&>` | Both stdout+stderr to file |
| `&>>` | Both append |
| `> file 2>&1` | Traditional both-to-file |
| `<` | Stdin from file |
| `\|` | Pipe stdout to next command |

- File descriptors: 0=stdin, 1=stdout, 2=stderr
- `> file` with no command = truncate/create empty file
- `/dev/null` = discard output: `cmd 2>/dev/null`
- `Ctrl-D` = signal EOF

### Filter commands

| Command | What it does |
|---------|-------------|
| `cat` | Concatenate / display files |
| `sort` | Sort lines |
| `uniq` | Remove adjacent duplicates (`-d` = show only dupes) |
| `grep pattern file` | Find matching lines (`-i` case-insensitive, `-v` invert) |
| `wc` | Count lines/words/bytes (`-l` = lines only) |
| `head -n N` | First N lines |
| `tail -n N` | Last N lines |
| `tail -f file` | Follow file in real time (`Ctrl-C` to stop) |
| `tee file` | Split output to file AND stdout |

### Pipeline examples
```bash
ls /usr/bin | sort | uniq | wc -l
ls /usr/bin | tee listing.txt | grep zip
cat movie.mpeg.0* > movie.mpeg        # join split files
```

> **Never** confuse `>` (to file) with `|` (to command)
