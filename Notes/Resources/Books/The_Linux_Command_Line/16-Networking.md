# Chapter 16 — Networking

## Quick Reference

| Command      | Description                                              |
| ------------ | -------------------------------------------------------- |
| `ping`       | Send ICMP ECHO_REQUEST to a host                         |
| `traceroute` | Print the hop-by-hop route to a host                     |
| `ip`         | Show/manipulate interfaces, routes, and tunnels          |
| `ss`         | Socket statistics (modern replacement for `netstat`)     |
| `wget`       | Non-interactive downloader (HTTP/FTP)                    |
| `ssh`        | Remote login over encrypted tunnel                       |
| `scp`        | Secure copy over SSH                                     |
| `sftp`       | Interactive secure file transfer over SSH                |

---

## Examining and Monitoring a Network

### ping

Sends ICMP ECHO_REQUEST packets to verify connectivity. Runs until interrupted with `Ctrl-C`.

```bash
pango@fedora ➜ ping linuxcommand.org
PING linuxcommand.org (216.105.38.11) 56(84) bytes of data.
64 bytes from secureprojects.sourceforge.net (216.105.38.11): icmp_seq=1 ttl=43 time=176 ms
^C
--- linuxcommand.org ping statistics ---
3 packets transmitted, 2 received, 33.3% packet loss, time 2001ms
rtt min/avg/max/mdev = 175.709/187.125/198.541/11.416 ms
```

> Note: Firewalls and some routers silently drop ICMP — no reply doesn't always mean the host is down.

### traceroute

Lists every router hop between you and the destination. Gaps (`* * *`) mean the router didn't reply to the TTL-expired probe — firewall, rate limiting, or ISP filtering — **not** a missing hop.

```bash
pango@fedora ➜ traceroute slashdot.org
traceroute to slashdot.org (104.18.4.215), 30 hops max, 60 byte packets
 1  10.2.0.1 (10.2.0.1)  55.387 ms ...        ← ProtonVPN gateway
 2  103.88.234.148       55.247 ms ...        ← VPN exit node
 3  * * *
 7  * * 38.194.249.112   56.368 ms
11  104.18.4.215         87.219 ms ...
```

> On your setup hop 1 is the ProtonVPN gateway (10.2.0.1) because all traffic routes through `proton0`. Use `-T` or `-I` flags to bypass firewall filtering on hops.

### ip

Replaced the deprecated `ifconfig`. Shows interfaces, addresses, and the routing table.

**Show all interfaces:**
```bash
ip a
# or shorter:
ip addr show
```

Your interfaces:
```
lo          127.0.0.1/8         loopback
enp6s0      (DOWN)              ethernet — no cable
wlo1        192.168.1.225/24    WiFi — primary (DHCP)
docker0     172.17.0.1/16       Docker bridge
proton0     10.2.0.2/32         ProtonVPN WireGuard tunnel
ipv6leakintrf0                  ProtonVPN IPv6 leak prevention
```

**Show routing table:**
```bash
ip route
# default via 192.168.1.1 dev wlo1 (your WiFi gateway)
# 172.17.0.0/16 dev docker0
# 192.168.1.0/24 dev wlo1
```

**Check a single interface:**
```bash
ip addr show wlo1
ip addr show proton0
```

**Look for UP and a valid inet address** — those are the two things that confirm an interface is healthy.

### ss (Socket Statistics)

Modern replacement for `netstat` — faster, more detail, same idea. `netstat` is deprecated on Fedora.

```bash
ss -tuln          # TCP+UDP listening sockets, numeric (no DNS lookup)
ss -tp            # TCP sockets + process names
ss -s             # summary statistics
```

Flags: `-t` TCP, `-u` UDP, `-l` listening only, `-n` numeric, `-p` process info.

Equivalent of the old `netstat -r` (routing table) is now just `ip route`.

---

## Transferring Files

### wget

Downloads files from HTTP or FTP non-interactively. Keeps going even if you log out.

```bash
wget https://example.com/file.tar.gz           # single file
wget -r -np https://example.com/docs/          # recursive site download
wget -c https://example.com/bigfile.iso        # resume partial download
wget -b https://example.com/file.iso           # background download (logs to wget-log)
```

---

## Secure Remote Access — SSH

SSH encrypts everything (passwords, commands, output). It replaced the old cleartext `telnet` and `rlogin`.

### ssh — Remote Login

```bash
ssh remote-sys                  # connect as current user
ssh pango@remote-sys            # connect as specific user
ssh remote-sys 'df -h'         # run a single command, output comes back locally
ssh remote-sys 'ls ~' > list.txt  # redirect remote output to a local file
```

**First connection** shows a fingerprint prompt — say `yes` to add the host to `~/.ssh/known_hosts`.

**If the fingerprint changes** (reinstalled OS, key rotation, or MITM warning):
```bash
# Remove the stale key (line number shown in the error message)
vim ~/.ssh/known_hosts
# or surgically:
ssh-keygen -R remote-sys
```

**Key-based auth** (no password prompts):
```bash
ssh-keygen                        # generate ~/.ssh/id_ed25519 + id_ed25519.pub
ssh-copy-id pango@remote-sys     # install your pubkey on the remote host
```

**SSH tunneling / X forwarding:**
```bash
ssh -X remote-sys                 # forward X11 (run GUI apps remotely)
ssh -Y remote-sys                 # trusted X11 forwarding (use if -X fails)
```

### scp — Secure Copy

Works like `cp` but with remote paths prefixed as `host:path`.

```bash
scp remote-sys:~/document.txt .               # remote → local
scp file.txt pango@remote-sys:~/docs/         # local → remote
scp -r mydir/ remote-sys:~/backup/            # recursive directory copy
```

### sftp — Secure FTP

Interactive FTP-like session over SSH. Needs only an SSH server on the remote — **no FTP server required**.

```bash
sftp remote-sys
sftp> ls                    # list remote
sftp> lcd ~/Downloads       # change local directory
sftp> get ubuntu.iso        # download file
sftp> put backup.tar.gz     # upload file
sftp> bye
```

> In Dolphin (KDE), you can open `sftp://user@remote-sys` directly in the address bar and browse/drag-drop files.

---

## Your Setup at a Glance

- **Primary connection:** WiFi (`wlo1`, 192.168.1.225, DHCP via 192.168.1.1)
- **Ethernet:** `enp6s0` — present but DOWN (no cable)
- **VPN:** ProtonVPN WireGuard (`proton0`, 10.2.0.2) — all traffic tunnels through here when active
- **Docker:** `docker0` + bridge `br-4ea0ab36b154` — DOWN when no containers running
- **`netstat` is gone** on Fedora 43 — use `ss` and `ip route` instead
