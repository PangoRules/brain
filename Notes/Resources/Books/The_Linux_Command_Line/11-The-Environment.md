# Chapter 11 — The Environment

The shell maintains a body of information during a session called the **environment**. Programs read it to determine system configuration and adjust their behavior.

## Commands

| Command  | Description                                          |
| -------- | ---------------------------------------------------- |
| printenv | Print part or all of the environment                 |
| set      | Set shell options (also displays variables)          |
| export   | Export a variable to subsequently executed programs  |
| alias    | Create an alias for a command                        |

## What's Stored in the Environment

Two types of data live in the environment:

- **Environment variables** — available to the shell and all child processes
- **Shell variables** — internal to the current shell session (set by bash/zsh itself)

The shell also stores **aliases** and **shell functions**.

### Inspecting the Environment

```bash
printenv | less        # show environment variables only
set | less             # show both shell and environment variables
printenv USER          # print a single variable's value
echo $HOME             # also works for any variable
alias                  # show defined aliases (not shown by printenv or set)
```

### Notable Variables

| Variable | Meaning |
| -------- | ------- |
| USER     | Current username |
| HOME     | Path to home directory |
| SHELL    | Path to the current shell |
| PWD      | Current working directory |
| OLDPWD   | Previous working directory |
| PATH     | Colon-separated list of directories searched for executables |
| PS1      | Prompt string (defines your shell prompt appearance) |
| LANG     | Locale/language settings |
| TERM     | Terminal emulator type |
| EDITOR   | Default text editor |
| PAGER    | Default pager program (e.g. less) |
| DISPLAY  | Display name for the X Window System |

## How the Environment Is Established

Bash reads **startup files** when a session begins. There are two session types:

### Login Shell
Started when you log in (TTY, SSH, etc.). Reads in order:
1. `/etc/profile` — system-wide defaults
2. `~/.bash_profile` → `~/.bash_login` → `~/.profile` (first one found wins)

### Non-Login Shell
Started when you open a terminal in a GUI. Reads:
1. `/etc/bash.bashrc`
2. `~/.bashrc`

> `~/.bashrc` is the most important file for personal customization. A typical `.bash_profile` simply sources it so both session types share the same config.

A typical `.bash_profile`:
```bash
PATH=$PATH:$HOME/bin
export PATH

# Source .bashrc if it exists
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```

## Modifying the Environment

**Rule of thumb:**
- Shell and program settings → `~/.bashrc`
- PATH additions and exported variables → `~/.bash_profile` (or `.profile` on Ubuntu)

Example additions to `~/.bashrc`:
```bash
# Make directory sharing easier
umask 0002

# Cleaner command history
export HISTCONTROL=ignoredups
export HISTSIZE=1000

# Useful aliases
alias ll='ls -l --color=auto'
alias l.='ls -d .* --color=auto'
```

Use `#` for comments. Lines starting with `#` are ignored by the shell.

## Activating Changes

After editing a startup file, apply it to the current session without restarting:

```bash
source ~/.bashrc
```
