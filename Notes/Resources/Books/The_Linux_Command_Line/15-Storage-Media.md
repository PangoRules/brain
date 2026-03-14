# Chapter 15 — Storage Media

Linux maintains a **single file system tree** with storage devices attached at various points, unlike Windows which assigns a separate tree per drive (C:\, D:\, etc.). Attaching a device to the tree is called **mounting**.

## Commands

| Command | Description |
| ------- | ----------- |
| `mount` | Mount a file system |
| `umount` | Unmount a file system |
| `fsck` | Check and repair a file system |
| `fdisk` | Manipulate disk partition table |
| `mkfs` | Create a file system |
| `dd` | Copy raw data blocks between devices or files |
| `genisoimage` | Create an ISO 9660 image file from a directory |
| `wodim` | Write an image to optical media |
| `md5sum` | Calculate an MD5 checksum |

## Mounting and Unmounting

```bash
mount                              # list all currently mounted file systems
mount -t iso9660 /dev/sdc /mnt/cdrom   # manually mount a device
umount /dev/sdc                    # unmount a device
```

> **Why unmounting matters:** The OS buffers writes in RAM and flushes them to disk lazily. Unmounting forces all pending data to be written. Removing a device without unmounting can corrupt the file system.

> **Gotcha:** You cannot unmount a device if your current working directory is inside its mount point. `cd` elsewhere first.

### /etc/fstab — Auto-mount at Boot

`/etc/fstab` lists devices to mount automatically at startup.

```
LABEL=/12    /       ext4    defaults    1 1
LABEL=/home  /home   ext4    defaults    1 2
```

| Field | Meaning |
| ----- | ------- |
| 1 — Device | Device file (e.g. `/dev/sda1`) or label/UUID |
| 2 — Mount point | Directory where the device is attached |
| 3 — FS type | `ext4`, `vfat`, `ntfs`, `iso9660`, etc. |
| 4 — Options | Mount options (e.g. `defaults`, `ro`) |
| 5 — Frequency | Whether `dump` backs it up (0 = no) |
| 6 — Order | `fsck` check order at boot (0 = skip) |

## Device Names

All devices live in `/dev`. Storage device naming patterns:

| Pattern | Device |
| ------- | ------ |
| `/dev/sd*` | SCSI/SATA disks, USB drives, flash drives (modern standard) |
| `/dev/hd*` | IDE (PATA) disks on older systems |
| `/dev/sr*` | Optical drives (CD/DVD) |
| `/dev/fd*` | Floppy disks |
| `/dev/lp*` | Printers |

Partitions are numbered: `/dev/sda` = whole disk, `/dev/sda1` = first partition.

### Finding a Device Name After Plugging In

```bash
sudo tail -f /var/log/messages    # watch kernel messages in real time
# plug in the device, look for lines like: [sdb] Attached SCSI removable disk
# then Ctrl-C
```

## Creating a New File System

### 1. Partition with `fdisk`

```bash
sudo umount /dev/sdb1
sudo fdisk /dev/sdb    # always specify the whole disk, not a partition
```

Useful `fdisk` commands (entered at the prompt):

| Key | Action |
| --- | ------ |
| `p` | Print current partition table |
| `n` | Add a new partition |
| `d` | Delete a partition |
| `t` | Change a partition type (e.g. `83` = Linux, `b` = FAT32) |
| `l` | List known partition types |
| `w` | Write changes to disk and exit |
| `q` | Quit without saving |
| `m` | Show help menu |

> Changes are held in memory until you press `w`. Press `q` to abandon safely.

### 2. Format with `mkfs`

```bash
sudo mkfs -t ext4 /dev/sdb1     # format as Linux ext4
sudo mkfs -t vfat /dev/sdb1     # format as FAT32
```

## Checking and Repairing — `fsck`

```bash
sudo fsck /dev/sdb1    # device must be unmounted first
```

- Checks file system integrity (also runs automatically at boot per `/etc/fstab` field 6)
- Can repair corruption; recovered file fragments go to `lost+found/` at the root of the file system

## Copying Raw Data — `dd`

```bash
dd if=input_file of=output_file [bs=block_size [count=blocks]]

# Clone one flash drive to another
dd if=/dev/sdb of=/dev/sdc

# Back up a flash drive to an image file
dd if=/dev/sdb of=flash_drive.img

# Create an ISO image from a CD-ROM
dd if=/dev/cdrom of=ubuntu.iso
```

> **Warning:** `dd` is sometimes called "destroy disk" — a typo in `if=` or `of=` can wipe the wrong device. Double-check before pressing Enter.

## Working with ISO Images

### Create an ISO from a Directory

```bash
genisoimage -o cd-rom.iso -R -J ~/cd-rom-files
# -R = Rock Ridge (long filenames + POSIX permissions for Linux)
# -J = Joliet extensions (long filenames for Windows)
```

### Mount an ISO Without Burning It

```bash
mkdir /mnt/iso_image
mount -t iso9660 -o loop image.iso /mnt/iso_image
```

### Blank a Rewritable CD-RW

```bash
wodim dev=/dev/cdrw blank=fast
```

### Burn an ISO to Disc

```bash
wodim dev=/dev/cdrw image.iso
wodim dev=/dev/cdrw -v -dao image.iso    # verbose, disc-at-once mode
```

## Verifying Integrity — `md5sum`

```bash
md5sum image.iso             # generate checksum of a file
md5sum /dev/cdrom            # generate checksum directly from a disc
```

Compare the output against the checksum published by the distributor to confirm the download or burn is uncorrupted.
