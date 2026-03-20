# 13. Storage and Disk Management

> **Difficulty:** Advanced | **Time Estimate:** 3–4 hours | **Prerequisites:** Lessons 1–12

---

## Learning Objectives

By the end of this lesson, you will be able to:

- Identify and describe Linux block devices and their naming conventions
- Create and manage partitions using `fdisk`, `gdisk`, and `parted`
- Create, mount, and persist filesystems using `/etc/fstab`
- Use LVM to create flexible, resizable logical volumes
- Understand RAID levels and configure software RAID with `mdadm`
- Monitor disk health with `smartctl` and perform disk imaging with `dd`
- Configure NFS and Samba network filesystems

---

## 1. Block Devices

Linux represents every storage device as a file under `/dev/`:

```
Device Type          Naming Convention
─────────────────────────────────────────────────
SATA/SAS HDD/SSD     /dev/sda, /dev/sdb, /dev/sdc
NVMe SSD             /dev/nvme0n1, /dev/nvme1n1
Virtual (VM)         /dev/vda, /dev/vdb
Partitions (SATA)    /dev/sda1, /dev/sda2
Partitions (NVMe)    /dev/nvme0n1p1, /dev/nvme0n1p2
Loop devices         /dev/loop0, /dev/loop1
CD/DVD               /dev/sr0
```

```bash
# List all block devices
lsblk
lsblk -f              # Include filesystem type and UUID
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,UUID

# Identify devices with filesystem info
sudo blkid
sudo blkid /dev/sda1  # Specific device

# Hardware info
sudo fdisk -l         # All partition tables
sudo parted -l        # All disks (parted format)
cat /proc/partitions  # Kernel view of partitions
```

### Block Device Diagram

```
Physical Drive: /dev/sda (500 GB)
┌──────────────────────────────────────────────┐
│  /dev/sda1   │   /dev/sda2   │  /dev/sda3    │
│  512 MB EFI  │  50 GB ext4   │  449 GB LVM   │
│  (ESP)       │  (/boot)      │  (PV)         │
└──────────────────────────────────────────────┘
```

---

## 2. Partition Tables: MBR vs GPT

```
Feature          MBR                    GPT
─────────────────────────────────────────────────────
Max disk size    2 TB                   9.4 ZB
Max partitions   4 primary (or 3+extended) 128 primary
Boot support     BIOS                   UEFI (also BIOS w/ workaround)
Redundancy       None                   Header + backup at end of disk
Introduced       1983                   ~2000 (part of UEFI spec)
Recommended for  Legacy systems         All modern systems
```

---

## 3. Partitioning Tools

### fdisk (MBR/GPT, interactive)

```bash
sudo fdisk /dev/sdb

# Interactive commands:
# p  → print partition table
# n  → new partition
# d  → delete partition
# t  → change partition type
# w  → write and exit
# q  → quit without saving
# m  → help menu

# Quick non-interactive example (create one partition using all space)
echo -e "n\np\n1\n\n\nw" | sudo fdisk /dev/sdb
```

### gdisk (GPT-specific)

```bash
sudo gdisk /dev/sdb
# Same commands as fdisk but GPT-native
# Type codes: 8300=Linux, 8200=swap, EF00=EFI, 8E00=LVM

# Verify GPT table
sudo gdisk -l /dev/sdb
```

### parted (scriptable, GPT and MBR)

```bash
# Interactive
sudo parted /dev/sdb

# Non-interactive (scriptable)
sudo parted /dev/sdb --script mklabel gpt
sudo parted /dev/sdb --script mkpart primary ext4 1MiB 10GiB
sudo parted /dev/sdb --script mkpart primary ext4 10GiB 100%
sudo parted /dev/sdb --script print
sudo parted /dev/sdb --script align-check optimal 1   # Check alignment
```

---

## 4. Filesystems

### Creating Filesystems

```bash
# ext4 — most common Linux filesystem
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 -L "DataDisk" -m 1 /dev/sdb1    # Label, 1% reserved

# XFS — high performance, good for large files
sudo mkfs.xfs /dev/sdb2
sudo mkfs.xfs -L "XFSDisk" /dev/sdb2

# Btrfs — modern, supports snapshots and RAID
sudo mkfs.btrfs /dev/sdb3
sudo mkfs.btrfs -L "BtrfsDisk" -d raid1 /dev/sdb3 /dev/sdc3  # RAID1

# FAT32 — cross-platform compatibility
sudo mkfs.vfat -F 32 -n "USB_DRIVE" /dev/sdb4

# NTFS — Windows compatibility
sudo mkfs.ntfs -f -L "Windows" /dev/sdb5

# Swap
sudo mkswap /dev/sdb6
sudo swapon /dev/sdb6
```

### Filesystem Comparison

```
Filesystem  Max File   Max Volume  Journaling  Snapshots  Use Case
────────────────────────────────────────────────────────────────────
ext4        16 TB      1 EB        Yes         No         General purpose
xfs         8 EB       8 EB        Yes         No         Large files, databases
btrfs       16 EB      16 EB       CoW         Yes        Modern desktops, NAS
ntfs        16 TB      256 TB      Yes         No         Windows interop
fat32       4 GB       8 TB        No          No         USB drives, EFI
```

---

## 5. Mounting Filesystems

```bash
# Mount a device
sudo mount /dev/sdb1 /mnt/data

# Mount with options
sudo mount -o ro /dev/sdb1 /mnt/data          # Read-only
sudo mount -o rw,noexec /dev/sdb1 /mnt/data   # No exec, read-write
sudo mount -o remount,rw /mnt/data            # Remount with new options
sudo mount -t ext4 /dev/sdb1 /mnt/data        # Specify filesystem type

# Mount ISO image
sudo mount -o loop disk.iso /mnt/iso

# Unmount
sudo umount /mnt/data
sudo umount -l /mnt/data    # Lazy unmount (detach when not busy)
sudo umount -f /mnt/data    # Force unmount

# View mounts
mount | column -t
findmnt
findmnt /mnt/data
cat /proc/mounts
```

### /etc/fstab — Persistent Mounts

```bash
# /etc/fstab format:
# <device>    <mountpoint>  <fstype>  <options>         <dump>  <pass>
#
# Column 5 (dump): 0=don't dump, 1=dump
# Column 6 (pass): 0=don't check, 1=check first (root), 2=check after root

# Get UUID first
sudo blkid /dev/sdb1    # Note the UUID

# Example /etc/fstab entries
cat << 'EOF'
# Static filesystem mounts
UUID=1a2b3c4d-...  /mnt/data    ext4   defaults,noatime   0  2
UUID=5e6f7a8b-...  /mnt/backup  xfs    defaults           0  2
UUID=9c0d1e2f-...  none         swap   sw                 0  0
/dev/sr0            /mnt/cdrom   iso9660 noauto,ro         0  0
tmpfs               /tmp         tmpfs  defaults,size=2G   0  0
EOF

# Test fstab without rebooting
sudo mount -a       # Mount all entries in fstab
sudo systemctl daemon-reload

# Verify all fstab mounts are OK
findmnt --verify
```

---

## 6. Disk Utilities

```bash
# Disk free space
df -hT               # Human readable with filesystem type
df -i                # Inode usage (check this too!)

# Disk usage
du -sh /var/log      # Total size of /var/log
du -sh /var/log/* | sort -rh | head -10  # Top 10 largest

# I/O statistics
iostat -xz 2 3       # Extended stats, 2s interval, 3 samples
iotop                # Real-time I/O by process (requires root)
iotop -ao            # Accumulated I/O stats

# Check and repair filesystems (unmounted!)
sudo fsck /dev/sdb1                    # Auto-check
sudo fsck -f /dev/sdb1                 # Force check even if clean
sudo e2fsck -f /dev/sdb1               # ext2/3/4 specific
sudo xfs_repair /dev/sdb1              # XFS repair

# Resize ext4 (can be done online when growing)
sudo resize2fs /dev/sdb1               # Fill available space
sudo resize2fs /dev/sdb1 20G           # Resize to specific size

# Tune ext4 filesystem
sudo tune2fs -l /dev/sdb1             # Show filesystem info
sudo tune2fs -m 0 /dev/sdb1           # Set reserved blocks to 0%
sudo tune2fs -e remount-ro /dev/sdb1  # On errors, remount read-only
```

---

## 7. LVM — Logical Volume Manager

```
Physical Volumes (PVs)   → Actual disk partitions/disks
Volume Groups (VGs)      → Pool of physical volumes
Logical Volumes (LVs)    → Virtual partitions carved from VG

/dev/sdb1 ──┐
            ├──► VG "datavg" ──► LV "data"    (/mnt/data,  50G)
/dev/sdc1 ──┘                └──► LV "backup" (/mnt/backup, 30G)
```

```bash
# === Physical Volumes ===
sudo pvcreate /dev/sdb1 /dev/sdc1    # Initialize PVs
sudo pvdisplay                        # Display PV info
sudo pvs                              # Summary
sudo pvscan                           # Scan for PVs

# === Volume Groups ===
sudo vgcreate datavg /dev/sdb1 /dev/sdc1   # Create VG
sudo vgdisplay datavg                       # Detailed info
sudo vgs                                    # Summary
sudo vgextend datavg /dev/sdd1             # Add PV to VG

# === Logical Volumes ===
sudo lvcreate -L 50G -n datalv datavg      # Fixed size
sudo lvcreate -l 100%FREE -n backuplv datavg  # Use all remaining
sudo lvcreate -l 80%VG -n datalv datavg    # 80% of VG

sudo lvdisplay datavg/datalv               # Detailed info
sudo lvs                                    # Summary
sudo lvs -o +devices                       # Show underlying devices

# Create filesystem on LV
sudo mkfs.ext4 /dev/datavg/datalv
sudo mount /dev/datavg/datalv /mnt/data

# === Resize LVs ===
# Extend (grow) — ext4 online is OK
sudo lvextend -L +20G /dev/datavg/datalv        # Add 20GB
sudo lvextend -l +100%FREE /dev/datavg/datalv   # Add all free space
sudo resize2fs /dev/datavg/datalv               # Resize ext4 to match

# XFS can only grow, not shrink
sudo xfs_growfs /mnt/data

# Shrink (ext4 — must unmount first!)
sudo umount /mnt/data
sudo e2fsck -f /dev/datavg/datalv
sudo resize2fs /dev/datavg/datalv 40G      # Shrink filesystem first
sudo lvreduce -L 40G /dev/datavg/datalv    # Then shrink LV

# === LVM Snapshots ===
sudo lvcreate -L 5G -s -n datalv_snap /dev/datavg/datalv
sudo mount -o ro /dev/datavg/datalv_snap /mnt/snap
sudo lvremove /dev/datavg/datalv_snap
```

---

## 8. RAID

### RAID Levels Comparison

```
Level   Min Disks   Redundancy   Read   Write   Usable Capacity
──────────────────────────────────────────────────────────────────
RAID 0  2           None         Fast   Fast    100% (striping)
RAID 1  2           1 disk fail  Fast   Normal  50% (mirroring)
RAID 5  3           1 disk fail  Fast   Medium  (N-1)/N disks
RAID 6  4           2 disk fail  Fast   Slower  (N-2)/N disks
RAID 10 4           1/2 per pair Fast   Fast    50% (stripe+mirror)
```

```bash
# Software RAID with mdadm
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# Check RAID status
cat /proc/mdstat
sudo mdadm --detail /dev/md0

# Monitor (watch it sync)
watch cat /proc/mdstat

# Save RAID config
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u   # Update initramfs to include config

# RAID maintenance
sudo mdadm --manage /dev/md0 --fail /dev/sdc    # Mark disk as failed
sudo mdadm --manage /dev/md0 --remove /dev/sdc  # Remove failed disk
sudo mdadm --manage /dev/md0 --add /dev/sdd     # Add replacement disk
```

---

## 9. Swap Space

```bash
# File-based swap (modern method)
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Verify
swapon --show
free -h

# Persistent swap — add to /etc/fstab
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Disable swap
sudo swapoff /swapfile

# Swappiness (0-100, lower = prefer RAM)
cat /proc/sys/vm/swappiness          # View (default: 60)
sudo sysctl vm.swappiness=10         # Temporary
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf  # Persistent
```

---

## 10. Disk Health

```bash
# smartctl — SMART disk diagnostics (install: smartmontools)
sudo smartctl -i /dev/sda            # Device info
sudo smartctl -H /dev/sda            # Health status (PASSED/FAILED)
sudo smartctl -a /dev/sda            # Full SMART report
sudo smartctl -t short /dev/sda      # Start short self-test
sudo smartctl -t long /dev/sda       # Start extended test
sudo smartctl -l selftest /dev/sda   # View test results

# badblocks — find bad sectors (DESTRUCTIVE with -w!)
sudo badblocks -v /dev/sdb           # Read-only scan (safe)
sudo badblocks -n /dev/sdb           # Non-destructive read-write
# WARNING: -w is DESTRUCTIVE — only on empty disks
sudo badblocks -w /dev/sdb           # Destructive write test

# NVMe health
sudo nvme smart-log /dev/nvme0n1
```

---

## 11. dd — Disk Imaging

```bash
# Clone entire disk to another
sudo dd if=/dev/sda of=/dev/sdb bs=64K status=progress

# Create disk image
sudo dd if=/dev/sda of=/backup/sda.img bs=64K status=progress

# Restore image to disk
sudo dd if=/backup/sda.img of=/dev/sdb bs=64K status=progress

# Compress while imaging
sudo dd if=/dev/sda bs=64K | gzip > /backup/sda.img.gz

# Restore compressed image
gunzip -c /backup/sda.img.gz | sudo dd of=/dev/sdb bs=64K

# Wipe a disk securely
sudo dd if=/dev/urandom of=/dev/sdb bs=4M status=progress

# Create a test file
dd if=/dev/zero of=/tmp/testfile bs=1M count=1024 status=progress

# Check dd progress (send SIGUSR1 to running dd)
sudo kill -USR1 $(pidof dd)
```

---

## 12. Network Filesystems

### NFS (Network File System)

```bash
# === NFS Server ===
sudo apt install nfs-kernel-server

# Define exports
cat << 'EOF' | sudo tee -a /etc/exports
/srv/nfs/data   192.168.1.0/24(rw,sync,no_subtree_check)
/srv/nfs/public *(ro,sync,no_subtree_check)
EOF

sudo exportfs -ra          # Reload exports
sudo exportfs -v           # Verify exports
sudo systemctl enable --now nfs-kernel-server

# === NFS Client ===
sudo apt install nfs-common
sudo mount -t nfs 192.168.1.100:/srv/nfs/data /mnt/nfs
# Persistent in /etc/fstab:
# 192.168.1.100:/srv/nfs/data /mnt/nfs nfs defaults,_netdev 0 0
```

### Samba (Windows/Linux file sharing)

```bash
# === Samba Server ===
sudo apt install samba

# Add share to /etc/samba/smb.conf
cat << 'EOF' | sudo tee -a /etc/samba/smb.conf
[SharedDocs]
   path = /srv/samba/shared
   browseable = yes
   read only = no
   valid users = @sambashare
EOF

# Create Samba user (must be a Linux user first)
sudo smbpasswd -a alice
sudo systemctl restart smbd

# === Samba Client ===
sudo apt install cifs-utils
sudo mount -t cifs //192.168.1.100/SharedDocs /mnt/samba \
    -o username=alice,password=secret
# Persistent with credentials file
echo -e "username=alice\npassword=secret" | sudo tee /etc/samba/creds
sudo chmod 600 /etc/samba/creds
# In /etc/fstab: //server/share /mnt/samba cifs credentials=/etc/samba/creds 0 0
```

---

## Common Mistakes

| Mistake | Problem | Fix |
|---|---|---|
| Running `fsck` on mounted filesystem | Causes corruption | Always unmount first |
| Forgetting `_netdev` in fstab for NFS | Boot hangs waiting for network | Add `_netdev` option |
| `lvreduce` before `resize2fs` | Data loss / corruption | Always shrink filesystem first, then LV |
| Not checking inodes (`df -i`) | Disk "full" but space available | Monitor inodes alongside blocks |
| Missing `e2fsck` before shrinking | resize2fs may fail | Run `e2fsck -f` before resize |
| Using `dd` with wrong `if`/`of` | Overwrites source with zeros | Triple-check `if=` and `of=` |

---

## Pro Tips

- **`lsblk -f`** is your go-to command to quickly map devices → filesystems → mount points → UUIDs
- Always use **UUID** in `/etc/fstab` instead of `/dev/sda1` — device names can change on reboot
- **LVM snapshots** before risky operations (upgrades, migrations) provide instant rollback capability
- **`noatime` mount option** significantly reduces disk writes on busy filesystems — add it to fstab
- For SSDs, add the **`discard`** mount option (or use `fstrim` via systemd) to enable TRIM
- **`smartctl -a`** on any new disk should be the first thing you run before trusting it with data

---

## Practice Exercises

1. **Partition a Disk** — Using a spare disk or VM, create a GPT table with three partitions: 512MB EFI, 20GB ext4, remainder as XFS.

2. **fstab Entry** — Mount the ext4 partition from Exercise 1 at `/mnt/test` using UUID, with `noatime` option. Verify with `findmnt --verify`.

3. **LVM Setup** — Create two PVs, combine them into a VG, create a 10GB LV, format it as ext4, mount it, then grow it by 5GB online.

4. **Snapshot Backup** — Create an LVM snapshot, mount it read-only, and use `rsync` to copy its contents to another location. Then remove the snapshot.

5. **Disk Health Report** — Run a SMART short test on a disk, capture the full `smartctl -a` output to a file, and identify the reallocated sector count.

6. **Swap File** — Create a 2GB swap file, enable it, verify with `swapon --show`, set swappiness to 10, and make it persistent across reboots.

7. **NFS Share** — Set up an NFS server exporting `/srv/share`, mount it on the same machine (or another VM) to `/mnt/nfs`, and verify read/write works.

8. **Disk Clone** — Use `dd` to image a small partition to a file, then verify the image by mounting it with a loop device: `mount -o loop disk.img /mnt/test`.

---

## Key Takeaways

- Linux block devices live under `/dev/`; use `lsblk -f` and `blkid` to explore them
- **GPT** is the modern partition table standard — use it unless you have a specific MBR requirement
- Filesystems must be **created** (`mkfs`), **mounted** (`mount`), and persisted (`/etc/fstab` with UUID)
- **LVM** separates storage administration from physical layout — enabling online resizing and snapshots
- **RAID** provides redundancy (not backup!) at various speed/capacity trade-offs
- `dd` is extremely powerful and equally dangerous — always verify `if=` and `of=` before pressing Enter
- `/etc/fstab` is critical — a bad entry can prevent boot; always test with `mount -a` after editing

---

## Next Lesson Preview

**Lesson 14: Advanced Networking** — We'll configure network interfaces with the `ip` command, build packet-filtering rules with `iptables` and `nftables`, set up UFW firewall profiles, explore DNS and DHCP servers, and master SSH tunneling and port forwarding for secure connectivity.
