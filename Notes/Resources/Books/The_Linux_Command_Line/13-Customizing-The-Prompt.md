# Chapter 13 — Customizing the Prompt
THIS IS FOR BASH. CHECK WHAT IS YOUR SHELL
The shell prompt is controlled by the `PS1` environment variable ("prompt string 1"). The default value is `[\u@\h \W]\$`, which produces `[me@linuxbox ~]$`.

## Prompt Escape Codes

| Sequence | Displays |
| -------- | -------- |
| `\u` | Current username |
| `\h` | Hostname (without domain) |
| `\H` | Full hostname |
| `\w` | Full current working directory |
| `\W` | Last part of current working directory |
| `\d` | Date (e.g. "Mon May 26") |
| `\t` | Time in 24-hour HH:MM:SS |
| `\T` | Time in 12-hour HH:MM:SS |
| `\A` | Time in 24-hour HH:MM |
| `\@` | Time in 12-hour AM/PM |
| `\s` | Shell name |
| `\v` | Shell version |
| `\j` | Number of jobs in current session |
| `\l` | Name of current terminal device |
| `\!` | History number of current command |
| `\#` | Number of commands entered this session |
| `\$` | `$` for normal user, `#` for superuser |
| `\a` | ASCII bell (beep) |
| `\n` | Newline |
| `\[` | Begin non-printing character sequence |
| `\]` | End non-printing character sequence |

> Always wrap non-printing sequences (like color codes) in `\[` and `\]` so bash calculates the visible prompt length correctly.

## Changing the Prompt

```bash
# Back up current prompt
ps1_old="$PS1"

# Experiment freely
PS1="\A \h \$ "          # time + hostname
PS1="<\u@\h \W>\$ "      # angle-bracket style

# Restore
PS1="$ps1_old"
```

## Adding Color

Colors use ANSI escape codes in the format `\033[attr;colorcode m`.

### Text Colors

| Sequence | Color | Sequence | Color |
| -------- | ----- | -------- | ----- |
| `\033[0;30m` | Black | `\033[1;30m` | Dark gray |
| `\033[0;31m` | Red | `\033[1;31m` | Light red |
| `\033[0;32m` | Green | `\033[1;32m` | Light green |
| `\033[0;33m` | Brown | `\033[1;33m` | Yellow |
| `\033[0;34m` | Blue | `\033[1;34m` | Light blue |
| `\033[0;35m` | Purple | `\033[1;35m` | Light purple |
| `\033[0;36m` | Cyan | `\033[1;36m` | Light cyan |
| `\033[0;37m` | Light gray | `\033[1;37m` | White |
| `\033[0m` | **Reset** (always add at the end) | | |

### Background Colors

| Sequence | Color |
| -------- | ----- |
| `\033[0;40m` | Black |
| `\033[0;41m` | Red |
| `\033[0;42m` | Green |
| `\033[0;43m` | Brown |
| `\033[0;44m` | Blue |
| `\033[0;45m` | Purple |
| `\033[0;46m` | Cyan |
| `\033[0;47m` | Light gray |

### Text Attributes (replace `0` in `\033[0;...m`)

| Code | Attribute |
| ---- | --------- |
| 0 | Normal |
| 1 | Bold |
| 4 | Underscore |
| 5 | Blinking (often unsupported) |
| 7 | Inverse (swap fg/bg) |

### Example: Red Prompt

```bash
# Color bleeds into typed text without a reset at the end
PS1="\[\033[0;31m\]<\u@\h \W>\$ "

# Correct: reset color after the prompt
PS1="\[\033[0;31m\]<\u@\h \W>\$\[\033[0m\] "
```

## Cursor Movement

These escape codes let you position the cursor anywhere — useful for drawing elements like a status bar at the top of the terminal.

| Sequence | Action |
| -------- | ------ |
| `\033[l;cH` | Move to line `l`, column `c` |
| `\033[nA` | Move up `n` lines |
| `\033[nB` | Move down `n` lines |
| `\033[nC` | Move forward `n` characters |
| `\033[nD` | Move backward `n` characters |
| `\033[2J` | Clear screen, move to upper-left |
| `\033[K` | Clear from cursor to end of line |
| `\033[s` | Save current cursor position |
| `\033[u` | Restore saved cursor position |

### Example: Prompt with a Colored Status Bar

This draws a red bar with a yellow clock pinned to the top of the terminal:

```bash
PS1="\[\033[s\033[0;0H\033[0;41m\033[K\033[1;33m\t\033[0m\033[u\]<\u@\h \W>\$ "
```

| Part | Action |
| ---- | ------ |
| `\033[s` | Save cursor position |
| `\033[0;0H` | Jump to top-left corner |
| `\033[0;41m` | Set background to red |
| `\033[K` | Fill line with red (clear to end) |
| `\033[1;33m` | Set text to yellow |
| `\t` | Print current time |
| `\033[0m` | Reset colors |
| `\033[u` | Restore cursor to original position |
| `<\u@\h \W>\$` | The actual prompt |

## Saving the Prompt Permanently

Add to `~/.bashrc`:

```bash
PS1="\[\033[0;31m\]<\u@\h \W>\$\[\033[0m\] "
export PS1
```
