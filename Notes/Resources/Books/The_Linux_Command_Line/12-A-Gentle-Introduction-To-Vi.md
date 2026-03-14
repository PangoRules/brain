# Chapter 12 — A Gentle Introduction to vi

**vi** (pronounced "vee eye") is a modal text editor at the core of Unix tradition. Most Linux systems ship **vim** ("vi improved" by Bram Moolenaar), symlinked as `vi`. For my usecase I did neovim

**Why bother learning it:**
- Almost always available — POSIX requires it; nano is not universal
- Lightweight and fast; designed so hands never leave the keyboard
- Older version written in 1976 by Bill Joy (co-founder of Sun Microsystems)

## Starting and Stopping

```bash
vi filename      # open or create a file
```

| Command | Action |
| ------- | ------ |
| `:q`    | Quit |
| `:q!`   | Force quit without saving |
| `:w`    | Save |
| `:wq`   | Save and quit |
| `ZZ`    | Save and quit (command mode shortcut) |
| `:w filename` | Save as new filename (does not rename current session) |

> If you get lost, press `ESC` twice to return to command mode.

## Modes

vi is a **modal editor** — keys behave differently depending on the active mode.

| Mode | How to enter | Indicator |
| ---- | ------------ | --------- |
| Command | Default on start; press `ESC` to return | (none) |
| Insert  | Press `i` (before cursor) or `a` (after cursor) | `-- INSERT --` |

## Cursor Movement

All movement is done in **command mode**.

| Key | Moves cursor |
| --- | ------------ |
| `h` / `←` | Left one character |
| `l` / `→` | Right one character |
| `j` / `↓` | Down one line |
| `k` / `↑` | Up one line |
| `0` | Beginning of line |
| `^` | First non-whitespace character of line |
| `$` | End of line |
| `w` | Next word (punctuation counts) |
| `W` | Next word (ignores punctuation) |
| `b` | Previous word (punctuation counts) |
| `B` | Previous word (ignores punctuation) |
| `Ctrl-F` / `Page Down` | Down one page |
| `Ctrl-B` / `Page Up` | Up one page |
| `G` | Last line of file |
| `numberG` | Jump to line number (e.g. `1G` = first line) |

> Prefix any movement with a number to repeat it: `5j` moves down 5 lines.

## Editing

### Entering Insert Mode

| Key | Action |
| --- | ------ |
| `i` | Insert before cursor |
| `a` | Append after cursor |
| `A` | Append at end of line |
| `o` | Open new line below and enter insert mode |
| `O` | Open new line above and enter insert mode |

### Undo

```
u    # undo last change (vim supports multiple levels; real vi only one)
```

### Deleting

| Command | Deletes |
| ------- | ------- |
| `x`     | Current character |
| `dd`    | Current line |
| `dW`    | Cursor to beginning of next word |
| `d$`    | Cursor to end of line |
| `d0`    | Cursor to beginning of line |
| `dG`    | Current line to end of file |
| `d20G`  | Current line to line 20 |

> Prefix with a number to repeat: `5dd` deletes 5 lines, `3x` deletes 3 characters.

`d` **cuts** — text goes into a paste buffer.

### Copying (Yanking) and Pasting

| Command | Copies |
| ------- | ------ |
| `yy`    | Current line |
| `5yy`   | Current line and next 4 |
| `yW`    | Cursor to beginning of next word |
| `y$`    | Cursor to end of line |
| `yG`    | Current line to end of file |

| Key | Action |
| --- | ------ |
| `p` | Paste after cursor |
| `P` | Paste before cursor |

### Joining Lines

```
J    # join current line with the line below it
```

## Search and Replace

### Search Within a Line

```
f<char>    # jump to next occurrence of <char> on the current line
;          # repeat the search
```

### Search the Entire File

```
/pattern   # search forward; press ENTER
n          # jump to next match
```

### Global Substitution (ex command)

```
:%s/old/new/g     # replace all occurrences in the file
:%s/old/new/gc    # same, but ask for confirmation each time
```

| Part | Meaning |
| ---- | ------- |
| `:`  | Start an ex command |
| `%`  | Range: entire file (equivalent to `1,$`) |
| `s`  | Substitution |
| `g`  | Global — every match on each line |
| `c`  | Confirm each replacement |

**Confirmation keys (when using `gc`):**

| Key | Action |
| --- | ------ |
| `y` | Replace this instance |
| `n` | Skip this instance |
| `a` | Replace this and all remaining |
| `q` / `ESC` | Stop substituting |
| `l` | Replace this one, then quit ("last") |

## Editing Multiple Files

```bash
vi file1 file2 file3
```

| Command | Action |
| ------- | ------ |
| `:bn` | Switch to next file |
| `:bp` | Switch to previous file |
| `:buffers` | List all open buffers |
| `:buffer 2` | Switch to buffer 2 |
| `:e filename` | Open an additional file |
| `:r filename` | Insert contents of a file below the cursor |
