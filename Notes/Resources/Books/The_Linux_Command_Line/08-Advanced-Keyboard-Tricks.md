
### Cursor movement

| Key | Action |
|-----|--------|
| `Ctrl-A` | Beginning of line |
| `Ctrl-E` | End of line |
| `Alt-F` | Forward one word |
| `Alt-B` | Backward one word |
| `Ctrl-L` | Clear screen |

### Editing

| Key | Action |
|-----|--------|
| `Ctrl-D` | Delete char at cursor |
| `Ctrl-T` | Transpose chars |
| `Alt-T` | Transpose words |
| `Alt-U` | Uppercase word |
| `Alt-L` | Lowercase word |

### Kill & yank (cut & paste)

| Key | Action |
|-----|--------|
| `Ctrl-K` | Kill to end of line |
| `Ctrl-U` | Kill to start of line |
| `Alt-D` | Kill to end of word |
| `Alt-Bksp` | Kill to start of word |
| `Ctrl-Y` | Yank (paste) from kill-ring |

### Tab completion
- `Tab` -- auto-complete (commands, paths, variables, usernames)
- `Tab Tab` -- show all possible completions
- `Alt-*` -- insert all completions
- Works on: commands, filenames, `$`variables, `~`users, `@`hosts

### History

| Key | Action |
|-----|--------|
| `Ctrl-R` | Reverse incremental search |
| `Ctrl-J` | Accept found command |
| `Ctrl-G` | Cancel search |
| `Ctrl-O` | Execute and advance to next |
| `Ctrl-P` / `Up` | Previous |
| `Ctrl-N` / `Down` | Next |

### History expansion

| Sequence | Repeats |
|----------|---------|
| `!!` | Last command |
| `!N` | Command #N |
| `!string` | Last command starting with string |
| `!?string` | Last command containing string |

```bash
history | grep command   # search history
```

- History file: `~/.bash_history`
- `script file` -- record entire terminal session to file
