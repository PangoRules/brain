# Chapter 10 — Processes

**Multi-tasking** is the illusion of doing more than one thing at once by rapidly switching between programs. The kernel manages this through **processes**.

At boot, the kernel launches `init` (PID 1), which runs init scripts that start all system services. Background services with no UI are called **daemon programs**.

Each process gets a **PID** assigned in ascending order. A process that spawns another is a **parent**; the new one is a **child**.

##  🔠 Commands

| Command  | Description                        |
| -------- | ---------------------------------- |
| ps       | Snapshot of current processes      |
| top      | Live process viewer                |
| jobs     | List active jobs                   |
| bg       | Resume a job in the background     |
| fg       | Bring a job to the foreground      |
| kill     | Send a signal to a process         |
| killall  | Send signals to processes by name  |
| shutdown | Shut down or reboot the system     |

## 👁 Viewing Processes — `ps`

```bash
ps        # processes in the current terminal session only
ps x      # all your processes (? in TTY = no controlling terminal)
ps aux    # all processes from all users with full details
```

### Process States (STAT column)

| State | Meaning |
| ----- | ------- |
| R | Running or ready to run |
| S | Sleeping (waiting for an event) |
| D | Uninterruptible sleep (waiting for I/O) |
| T | Stopped |
| Z | Zombie — child terminated but not yet cleaned up by parent |
| < | High-priority |
| N | Low-priority (nice) |

### `ps aux` Column Headers

| Header | Meaning |
| ------ | ------- |
| USER   | Owner of the process |
| %CPU   | CPU usage |
| %MEM   | Memory usage |
| VSZ    | Virtual memory size |
| RSS    | Physical RAM used (KB) |
| START  | Time the process started |

## 👓 Live View — `top`

```bash
top    # updates every 3 seconds, sorted by CPU activity
```

Press `h` for help, `q` to quit.

### Header Breakdown

| Row | Field        | Meaning |
| --- | ------------ | ------- |
| 1   | uptime       | Time since last boot |
| 1   | load average | Waiting-to-run processes (1/5/15 min avg). Under 1.0 = not busy |
| 2   | Tasks        | Process counts by state |
| 3   | %Cpu         | `us`=user, `sy`=system, `ni`=nice, `id`=idle, `wa`=I/O wait |
| 4   | Mem          | Physical RAM usage |
| 5   | Swap         | Virtual memory (swap) usage |

## 🎛 Controlling Processes

### Background & Foreground

```bash
some_command &    # start in background; shell prints [job#] PID
jobs              # list background jobs
fg %1             # bring job 1 to the foreground
bg %1             # resume stopped job 1 in the background
```

### Keyboard Shortcuts

| Keys   | Signal | Effect |
| ------ | ------ | ------ |
| Ctrl-C | INT    | Interrupt (terminate) the foreground process |
| Ctrl-Z | TSTP   | Stop (pause) the foreground process |

## 📶 Signals & `kill`

```bash
kill -signal PID    # TERM is the default signal if none specified
kill -l             # list all available signals
```

### Common Signals

| #  | Name | Meaning |
| -- | ---- | ------- |
| 1  | HUP  | Hang up; also used to reload/reinitialize daemons |
| 2  | INT  | Interrupt — same as Ctrl-C |
| 9  | KILL | Force-terminate via kernel; **cannot be caught or ignored** |
| 15 | TERM | Graceful terminate (default) |
| 18 | CONT | Continue a stopped process |
| 19 | STOP | Pause — cannot be ignored |
| 20 | TSTP | Terminal stop — same as Ctrl-Z; can be ignored by programs |

> You must own the process (or be superuser) to send signals to it.

## ⏬ `killall` — Kill by Name

```bash
killall [-u user] [-signal] name
```

##  ⚫ Shutdown

```bash
sudo shutdown -h now   # halt
sudo shutdown -r now   # reboot
```

Alternatives: `halt`, `poweroff`, `reboot`

## 🗣️ Other Useful Commands

| Command | Description |
| ------- | ----------- |
| pstree  | Process list as a tree showing parent–child relationships |
| vmstat  | Snapshot of memory, swap, and disk I/O (`vmstat 5` = continuous) |
| xload   | Graphical system load graph over time |
| tload   | Terminal-based system load graph |
