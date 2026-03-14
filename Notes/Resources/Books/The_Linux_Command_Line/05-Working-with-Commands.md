
| Command | What it does |
|---------|-------------|
| `type cmd` | What kind of command is it |
| `which cmd` | Where is the executable |
| `help cmd` | Help for builtins |
| `cmd --help` | Usage info |
| `man cmd` | Manual page |
| `man 5 passwd` | Specific man section |
| `apropos term` | Search man pages (= `man -k`) |
| `whatis cmd` | One-line description |
| `info cmd` | GNU info docs (hyperlinked) |
| `alias name='cmd'` | Create alias |
| `unalias name` | Remove alias |
| `alias` | List all aliases |

### Four types of commands
1. **Executable** -- binary/script in PATH
2. **Builtin** -- part of bash itself (`cd`, `type`)
3. **Function** -- shell script in environment
4. **Alias** -- user-defined shortcut

### Man sections
1=user cmds, 5=file formats, 8=admin cmds

### Alias tips
- `alias ll='ls -lh'`
- Semicolons chain commands: `alias backup='cd /backup; ls -l'`
- Session-only unless added to `~/.bashrc`

---
