# Table of Contents
1. [[#👥 Owners, Group Members and Everybody Else]]]
2. [[#🔐 Reading, Writing, and Executing]]
3. [[#⚙️ chmod Change File Mode]]
4. [[#🫆 Changing Identities]]
## 👥 Owners, Group Members and Everybody Else
  
Linux is a **multiuser system**.  
More than one person can use a computer at the same time.  
  
Even if a machine has only one keyboard and mouse, multiple users can access it — especially over a network.  
  
🌐 If connected to a network or the Internet, remote users can log in using `ssh` (secure shell) and operate the system.  
🖥️ They can even run graphical applications remotely thanks to the X Window System.  
  
---
### 🛠 Useful Commands  
  
| Command | Description                       |
| ------- | --------------------------------- |
| id      | Display user identity             |
| chmod   | Change a file's mode              |
| umask   | Set default file permissions      |
| su      | Run a shell as another user       |
| sudo    | Execute a command as another user |
| chown   | Change a file's owner             |
| chgrp   | Change a file's group             |
| passwd  | Change a user's password          |

---  
### 👤 Ownership Model  
  
Users may **own** files and directories.  
  
👥 Users can belong to a **group**, allowing shared access.  
🌍 Access can also be granted to **everybody else**, referred to as the **world**.  
  
---
### 📂 System Account Files  
  
- `/etc/passwd` → User account definitions  
- `/etc/group` → Group definitions  
- `/etc/shadow` → Encrypted password information  
  
Each entry in `/etc/passwd` defines:  
- Login name  
- UID  
- GID  
- Real name  
- Home directory  
- Login shell  
  
---  
  
## 🔐 Reading, Writing, and Executing  
  
Access to files and directories is controlled using:  
  
- 📖 **Read (r)**  
- ✏️ **Write (w)**  
- ⚙️ **Execute (x)**  
  
Example:  
  
```bash  
[me@linuxbox ~]$ > foo.txt  
[me@linuxbox ~]$ ls -l foo.txt  
-rw-rw-r-- 1 me me 0 2018-03-06 14:52 foo.txt
```

### 🏷 File Type Indicator (First Character)

| Attribute | Type                                                        |
| --------- | ----------------------------------------------------------- |
| -         | Regular file                                                |
| d         | Directory                                                   |
| l         | Symbolic link (permissions shown are dummy values)          |
| c         | Character special file (byte-stream devices like terminals) |
| b         | Block special file (block devices like disks)               |
### 🔎 File Mode Structure

The remaining nine characters represent permissions:

|Owner|Group|World|
|---|---|---|
|rwx|rwx|rwx|

### 📘 Permission Meaning

| Attribute | Files                                  | Directories                                              |
| --------- | -------------------------------------- | -------------------------------------------------------- |
| r         | Allows file to be read                 | Allows listing contents (requires `x`)                   |
| w         | Allows file to be written or truncated | Allows creating, deleting, renaming files (requires `x`) |
| x         | Allows file to be executed             | Allows entering directory (`cd`)                         |
⚠️ Deleting a file depends on the **directory’s permissions**, not the file’s.

----
### 📊 Common File Attributes

| Attributes | Meaning                                 |
| ---------- | --------------------------------------- |
| -rwx------ | Owner: full access                      |
| -rw------- | Owner: read/write only                  |
| -rw-r--r-- | Owner: read/write; others: read         |
| -rwxr-xr-x | Owner: full; others: read/execute       |
| -rw-rw---- | Owner and group: read/write             |
| lrwxrwxrwx | Symbolic link (dummy permissions)       |
| drwxrwx--- | Directory; owner & group full access    |
| drwxr-x--- | Directory; owner full; group enter only |

## ⚙️ chmod: Change File Mode
Only the file's **owner** or the **superuser (root)** can change the mode of a file or directory.

`chmod` supports two ways to specify mode changes:

- 🔢 Octal (numeric) representation  
- 🔤 Symbolic representation  
---
### 🔢 Octal Representation
Each permission has a binary and octal value:

| Octal | Binary | File Mode |
|-------|--------|-----------|
| 0     | 000    | --- |
| 1     | 001    | --x |
| 2     | 010    | -w- |
| 3     | 011    | -wx |
| 4     | 100    | r-- |
| 5     | 101    | r-x |
| 6     | 110    | rw- |
| 7     | 111    | rwx |

Using **three octal digits**, we set permissions for:
- Owner  
- Group  
- World  

```bash
[me@linuxbox ~]$ > foo.txt
[me@linuxbox ~]$ ls -l foo.txt
-rw-rw-r-- 1 me me 0 2018-03-06 14:52 foo.txt

[me@linuxbox ~]$ chmod 600 foo.txt
[me@linuxbox ~]$ ls -l foo.txt
-rw------- 1 me me 0 2018-03-06 14:52 foo.txt
```

### 🔤 Symbolic representation
Symbolic notation does offer the advantage of allowing you to set a single attribute without disturbing any of the others.

| **Symbol ** | **Meaning**                     |
| ----------- | ------------------------------- |
| u           | File/Directory owner.           |
| g           | Group owner.                    |
| o           | World                           |
| a           | All (combination of u, g and o) |
>If no characters are specified, "all" will be assumed.

| **Notion** | **Meaning**                                                                                                                                                                   |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| u+x        | Add execute permission for the owner.                                                                                                                                         |
| u-x        | Remove execute permission from the owner.                                                                                                                                     |
| +x         | Add execute permission for the owner, group and world. Equivalent to a+x.                                                                                                     |
| o-rw       | Remove the read and write permissions from anyone besides the owner and group owner.                                                                                          |
| go=rw      | Set the group owner and anyone besides the owner to have read and write permissions. If either the group owner or the world previously had execute permission, it is removed. |
| u+x,go=rx  | Add execute permission for the owner and set the permissions for the group and others to read and execute. Multiple specifications may be separated by commas.                |

### 🎭 umask: Set Default Permissions
`umask` controls the **default permissions** applied when a new file or directory is created.  
  
It does **NOT** add permissions.  
It **removes** (masks) permissions from the system defaults.  

---  
#### 🧠 How It Works  
When a file is created:  
- Default file permission = `666` (rw-rw-rw-)  
- Default directory permission = `777` (rwxrwxrwx)  
`umask` subtracts permissions from those defaults.  
  
Formula:  
Default Permissions - umask = Final Permissions  
  
---  
#### 🔎 Example 1 — Default umask 0002

```bash
umask          # shows current mask, e.g. 0002
> foo.txt
ls -l foo.txt
-rw-rw-r--    # world loses write because of the mask
```

#### 🔎 Example 2 — umask 0000 (mask off)

```bash
umask 0000
> foo.txt
ls -l foo.txt
-rw-rw-rw-    # all write bits survive — world writable!
```

#### 🧮 How the Mask Works (Binary)

| | Octal | Binary |
|---|---|---|
| Default file perm | 666 | rw-rw-rw- |
| umask | 0002 | 000 000 010 |
| **Result** | **664** | **rw-rw-r--** |

Wherever a **1** appears in the mask's binary value, that permission bit is **removed**.
Common mask values: `0022` (removes group/world write) · `0002` (removes only world write).

> ⚠️ The umask change is **session-only**. To make it permanent, set it in your shell config (covered in Chapter 11).

---

### 🔒 Special Permissions

Beyond rwx, there are three extra permission bits (expressed as a **4th octal digit**):

| Bit | Octal | Name | Effect on Files | Effect on Directories |
|---|---|---|---|---|
| setuid | 4000 | Set User ID | Program runs as the **file owner** (not the caller) | N/A |
| setgid | 2000 | Set Group ID | Program runs with the **file's group** | New files inside inherit the **directory's group** |
| sticky | 1000 | Sticky bit | Ignored on Linux | Users can only delete/rename **their own** files |

### setuid — `chmod u+s program`
```bash
chmod u+s program
# ls output shows 's' in owner execute slot:
-rwsr-xr-x
```
> ⚠️ Allows ordinary users to run a program with elevated (often root) privileges. Keep setuid programs to an **absolute minimum**.

### setgid on a directory — `chmod g+s dir`
```bash
chmod g+s dir
# ls output shows 's' in group execute slot:
drwxrwsr-x
```
> Files created inside will inherit the **directory's group**, not the creator's primary group. Great for shared directories.

### sticky bit — `chmod +t dir`
```bash
chmod +t dir
# ls output shows 't' in world execute slot:
drwxrwxrwt
```
> Classic example: `/tmp` — anyone can write files there, but you can only delete **your own**.

---

## 🫆 Changing Identities

There are three ways to take on an alternate identity:
1. Log out and log back in as the alternate user.
2. Use **`su`** command.
3. Use **`sudo`** command.

---

### 🔄 su — Run a Shell as Another User

**Syntax:**
```bash
su [-[l]] [user]
```

| Flag | Meaning |
|---|---|
| `-l` or `-` | Start a **login shell** — loads the target user's environment and changes to their home dir |
| *(no user)* | Assumes **superuser (root)** |

**Become root:**
```bash
[me@linuxbox ~]$ su -
Password:
[root@linuxbox ~]#    # '#' prompt = superuser
```
Type `exit` to return to your original shell.

**Run a single command as root (without starting a new shell):**
```bash
su -c 'command'
```
> Wrap the command in **quotes** so expansion happens in the new shell, not yours.

```bash
[me@linuxbox ~]$ su -c 'ls -l /root/*'
Password:
-rw------- 1 root root 754 ... /root/anaconda-ks.cfg
[me@linuxbox ~]$
```

---

### ⚡ sudo — Execute a Command as Another User

`sudo` is the modern, preferred alternative. Key differences from `su`:

| | su | sudo |
|---|---|---|
| Password required | **Root's** password | **Your own** password |
| Starts a new shell | Yes | No (unless `-i`) |
| Loads other user's env | Yes | No |
| Per-command control | No | Yes (via `/etc/sudoers`) |
| Trust timer | No | Yes (~5 min after auth) |

**Run a privileged command:**
```bash
[me@linuxbox ~]$ sudo backup_script
Password:           # your password, not root's
System Backup Starting...
```

**Start an interactive superuser session (like `su -`):**
```bash
sudo -i
```

**List your sudo privileges:**
```bash
sudo -l
# User me may run the following commands:
#     (ALL) ALL
```

> 💡 **Ubuntu** disables the root account by default and uses `sudo` exclusively. The first user gets full sudo access.

---

### 👤 chown — Change File Owner and Group

Requires **superuser** privileges.

**Syntax:**
```bash
chown [owner][:[group]] file...
```

| Argument | Result |
|---|---|
| `bob` | Change owner to bob |
| `bob:users` | Change owner to bob **and** group to users |
| `:admins` | Change **group only** to admins (owner unchanged) |
| `bob:` | Change owner to bob **and** group to bob's login group |

**Real-world example — janet transfers a file to tony:**
```bash
[janet@linuxbox ~]$ sudo cp myfile.txt ~tony
[janet@linuxbox ~]$ sudo chown tony: ~tony/myfile.txt
# Now both owner and group show 'tony'
```
> After the first `sudo`, you're trusted for several minutes — no re-prompt.

---

### 👥 chgrp — Change Group Ownership

An older, more limited command. Changes **only** the group (not the owner).
`chown :group file` is the modern equivalent and preferred.

---

### 🎵 Practical Example — Setting Up a Shared Directory

**Goal:** Users `bill` and `karen` share `/usr/local/share/Music`.

```bash
# 1. Create a 'music' group with both users as members (via GUI or addgroup/usermod)

# 2. Create the directory
sudo mkdir /usr/local/share/Music

# 3. Assign group ownership and allow group write
sudo chown :music /usr/local/share/Music
sudo chmod 775 /usr/local/share/Music
# Result: drwxrwxr-x  root  music

# 4. Set the setgid bit so new files inherit 'music' group automatically
sudo chmod g+s /usr/local/share/Music
# Result: drwxrwsr-x  root  music

# 5. Each user sets umask to 0002 so group members can write each other's files
umask 0002
```

After step 5, files/dirs created inside inherit the `music` group and are group-writable — both users can collaborate freely.

> ⚠️ The `umask 0002` must be made **permanent** in your shell startup file (see Chapter 11), or it resets each session.

---

### 🔑 passwd — Change a User's Password

**Syntax:**
```bash
passwd [user]
```

- Run without arguments → change **your own** password (prompts for old password first).
- Run with a username (requires superuser) → change **another user's** password.

```bash
[me@linuxbox ~]$ passwd
(current) UNIX password:
New UNIX password:
```

`passwd` enforces **strong password** rules — rejects:
- Passwords too similar to the old one
- Passwords that are too short
- Dictionary words or easily guessed strings

Superuser options also allow: account locking, password expiration, etc. (see `man passwd`).
