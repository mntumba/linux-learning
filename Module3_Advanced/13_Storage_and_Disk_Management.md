# 13 — Storage and Disk Management

> **Module 3 · Lesson 3** | Difficulty: ★★★☆☆ Intermediate-Advanced | Time: ~90 min

---

## Learning Objectives

- Partition disks with fdisk, parted, and gdisk
- Create and manage filesystems (ext4, xfs, btrfs)
- Mount and unmount filesystems
- Configure /etc/fstab for persistent mounts
- Set up LVM (Logical Volume Manager)
- Configure RAID
- Monitor disk health with SMART

---

## 1. Disk and Partition Management

### Viewing Storage Devices

```bash
lsblk                          # list block devices (tree view)
lsblk -f                       # with filesystem info
fdisk -l                       # detailed partition table
blkid                          # block device UUIDs and types
cat /proc/partitions           # raw partition info

# Example lsblk output:
# NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# sda      8:0    0   50G  0 disk
# ├─sda1   8:1    0  512M  0 part /boot/efi
# ├─sda2   8:2    0    1G  0 part /boot
# └─sda3   8:3    0 48.5G  0 part /
# sdb      8:16   0  100G  0 disk (new empty disk)
```

### Partitioning with fdisk (MBR/GPT)

```bash
sudo fdisk /dev/sdb            # open disk for partitioning

# fdisk commands:
# p  — print partition table
# n  — new partition
# d  — delete partition
# t  — change partition type
# w  — write and exit
# q  — quit without saving
# m  — help

# Create a new partition:
# Press n → p (primary) → 1 → default start → +20G
# Press t → 82 (Linux swap) or 83 (Linux)
# Press w to write

# GPT disks — use gdisk or parted
sudo gdisk /dev/sdb
sudo parted /dev/sdb

# parted commands
sudo parted /dev/sdb mklabel gpt                      # create GPT
sudo parted /dev/sdb mkpart primary 0% 100%           # single partition
sudo parted /dev/sdb print                             # view
```

### Creating Filesystems

```bash
# ext4 (most common)
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 -L "mydata" /dev/sdb1          # with label

# xfs (high performance)
sudo mkfs.xfs /dev/sdb1
sudo mkfs.xfs -L "mydata" /dev/sdb1

# btrfs (copy-on-write, snapshots)
sudo mkfs.btrfs /dev/sdb1
sudo mkfs.btrfs -L "mydata" /dev/sdb1

# swap
sudo mkswap /dev/sdb2
sudo swapon /dev/sdb2
sudo swapoff /dev/sdb2
```

### Mounting Filesystems

```bash
# Temporary mount
sudo mkdir /mnt/newdisk
sudo mount /dev/sdb1 /mnt/newdisk

# Mount with options
sudo mount -o ro /dev/sdb1 /mnt/newdisk                # read-only
sudo mount -o rw,noexec,nosuid /dev/sdb1 /mnt/newdisk

# Mount ISO file
sudo mount -o loop disk.iso /mnt/iso

# Unmount
sudo umount /mnt/newdisk
sudo umount -l /mnt/newdisk        # lazy unmount (busy filesystem)

# View mounts
mount
cat /proc/mounts
findmnt                            # tree view
df -h                              # with usage
```

### /etc/fstab — Persistent Mounts

```bash
# Format: device  mountpoint  fstype  options  dump  pass
# /dev/sda1  /  ext4  defaults  0  1

# View current fstab
cat /etc/fstab

# Use UUID (more reliable than device name)
blkid /dev/sdb1                    # get UUID
# UUID="550e8400-e29b-41d4-a716-446655440000" TYPE="ext4"

# Add entry to fstab:
echo "UUID=550e8400-e29b-41d4-a716-446655440000 /mnt/data ext4 defaults 0 2" \
    | sudo tee -a /etc/fstab

# Test fstab without rebooting
sudo mount -a                      # mount all in fstab

# fstab options:
# defaults    — rw, suid, dev, exec, auto, nouser, async
# noatime     — don't update access time (better performance)
# noexec      — can't execute files (security)
# nosuid      — ignore SUID bits (security)
# ro          — read-only
# user        — any user can mount
# auto        — mount at boot
# nofail      — don't fail boot if device missing
```

---

## 2. LVM — Logical Volume Manager

LVM adds a flexible abstraction layer between disks and filesystems.

```
Physical Disks (/dev/sdb, /dev/sdc)
          │
          ▼
Physical Volumes (PV)
          │
          ▼
Volume Group (VG) — pool of storage
          │
          ▼
Logical Volumes (LV) — flexible partitions
          │
          ▼
Filesystems (ext4, xfs, etc.)
```

```bash
# Install LVM
sudo apt install lvm2

# Step 1: Create Physical Volumes
sudo pvcreate /dev/sdb /dev/sdc
sudo pvdisplay
sudo pvs

# Step 2: Create Volume Group
sudo vgcreate myvg /dev/sdb /dev/sdc
sudo vgdisplay myvg
sudo vgs

# Step 3: Create Logical Volumes
sudo lvcreate -L 10G -n data myvg      # 10GB LV named "data"
sudo lvcreate -l 100%FREE -n backup myvg  # use remaining space
sudo lvdisplay
sudo lvs

# Step 4: Create filesystem
sudo mkfs.ext4 /dev/myvg/data

# Step 5: Mount
sudo mkdir /mnt/data
sudo mount /dev/myvg/data /mnt/data

# Extend a Logical Volume (online, no downtime!)
sudo lvextend -L +5G /dev/myvg/data        # add 5GB
sudo resize2fs /dev/myvg/data               # resize ext4 filesystem
# or for xfs:
sudo xfs_growfs /mnt/data

# Extend Volume Group (add disk)
sudo pvcreate /dev/sdd
sudo vgextend myvg /dev/sdd

# Create snapshot (for backups)
sudo lvcreate -s -n data_snap -L 1G /dev/myvg/data

# Remove
sudo lvremove /dev/myvg/data_snap
sudo lvremove /dev/myvg/data
sudo vgremove myvg
sudo pvremove /dev/sdb /dev/sdc
```

---

## 3. RAID Configuration

```bash
sudo apt install mdadm

# Create RAID 1 (mirror) with 2 disks
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# Create RAID 5 (striping with parity) - needs 3+ disks
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# View RAID status
cat /proc/mdstat
sudo mdadm --detail /dev/md0

# Create filesystem and mount
sudo mkfs.ext4 /dev/md0
sudo mount /dev/md0 /mnt/raid

# Save configuration
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
```

---

## 4. Disk Health Monitoring (SMART)

```bash
sudo apt install smartmontools

# View disk SMART info
sudo smartctl -i /dev/sda           # basic info
sudo smartctl -H /dev/sda           # health check
sudo smartctl -a /dev/sda           # full report

# Run tests
sudo smartctl -t short /dev/sda     # short test (~2 min)
sudo smartctl -t long /dev/sda      # long test (hours)

# Check test results
sudo smartctl -l selftest /dev/sda

# Monitor temperature
sudo hddtemp /dev/sda               # sudo apt install hddtemp

# Enable automatic monitoring
sudo systemctl enable --now smartd
sudo vim /etc/smartd.conf
```

---

## Practice Exercises

### Exercise 13.1 — Create and Mount a Disk Image

```bash
# Create a 100MB "virtual disk"
dd if=/dev/zero of=/tmp/virtual_disk.img bs=1M count=100
losetup /dev/loop0 /tmp/virtual_disk.img
fdisk /dev/loop0   # create partition
mkfs.ext4 /dev/loop0p1
mount /dev/loop0p1 /mnt/test
df -h /mnt/test
umount /mnt/test
losetup -d /dev/loop0
```

### Exercise 13.2 — LVM Setup

```bash
# Practice LVM with loop devices
dd if=/dev/zero of=/tmp/disk{1,2}.img bs=1M count=500
sudo losetup /dev/loop1 /tmp/disk1.img
sudo losetup /dev/loop2 /tmp/disk2.img
sudo pvcreate /dev/loop1 /dev/loop2
sudo vgcreate testvg /dev/loop1 /dev/loop2
sudo lvcreate -L 200M -n testlv testvg
sudo mkfs.ext4 /dev/testvg/testlv
sudo mkdir /mnt/lvm_test
sudo mount /dev/testvg/testlv /mnt/lvm_test
# Extend the LV
sudo lvextend -L +100M /dev/testvg/testlv
sudo resize2fs /dev/testvg/testlv
df -h /mnt/lvm_test
```

---

## Key Takeaways

- `lsblk` and `fdisk -l` show disk layout
- **GPT** for modern disks (>2TB); **MBR** for legacy
- **ext4** is stable and default; **xfs** for high performance; **btrfs** for snapshots
- **`/etc/fstab`** with UUIDs ensures consistent mounts across reboots
- **LVM** enables flexible disk management and online resizing
- **SMART** monitoring detects failing disks before data loss

---

➡️ [14 — Advanced Networking](14_Advanced_Networking.md)

*Module 3 · Lesson 3 of 5 | [Course Index](../INDEX.md)*
