# AWS EBS Volume: Attach, Partition, Format, Mount & Persist

## 1. What We Practiced

In this lab, we simulated a real DevOps/Linux administration scenario:

> A new 100 GB EBS volume was attached to an EC2 instance. We needed to prepare it for use, mount it at `/data`, and configure it so the mount survives an EC2 reboot.

We started with an EC2 instance whose root disk was 8 GB and then attached a new 100 GB EBS volume.

### Final architecture

```text
AWS EBS Volume (100 GB)
        |
        v
/dev/nvme1n1
        |
        v
/dev/nvme1n1p1
        |
        v
XFS filesystem
        |
        v
/data
        |
        v
/etc/fstab (UUID + nofail)
        |
        v
Persistent across reboot
```

---

# 2. Important Commands and What They Mean

## `lsblk`

Shows block devices, disks, partitions, sizes, and mount points.

```bash
lsblk
```

Initially:

```text
nvme0n1       8G  disk
├─nvme0n1p1   8G  part /
├─nvme0n1p127 1M  part
└─nvme0n1p128 10M part /boot/efi

nvme1n1       100G disk
```

This told us that Linux detected the newly attached 100 GB EBS volume as:

```text
/dev/nvme1n1
```

### Key point

`lsblk` shows block devices whether or not they have a filesystem or are mounted.

---

# 3. `df -h`

Shows mounted filesystems and their disk usage.

```bash
df -h
```

Initially, the 100 GB disk did NOT appear in `df -h`.

That was expected because:

```text
/dev/nvme1n1
    |
    +-- no filesystem
    |
    +-- not mounted
```

### Important interview distinction

**`lsblk` vs `df -h`:**

- `lsblk` → shows disks, partitions, and block-device relationships.
- `df -h` → shows mounted filesystems and their space usage.

Therefore, a disk can appear in `lsblk` but not in `df -h`.

---

# 4. Check Whether the New Disk Has a Filesystem

We ran:

```bash
sudo blkid /dev/nvme1n1
```

It returned nothing.

Then:

```bash
sudo file -s /dev/nvme1n1
```

Output:

```text
/dev/nvme1n1: data
```

This confirmed that the volume was essentially a blank/unformatted block device.

### Why check this?

Before formatting a disk, you should verify whether it already contains a filesystem/data.

**Warning:** Never blindly run `mkfs` on a disk in production. Formatting destroys existing filesystem data.

---

# 5. Create a Partition

We created a partition using:

```bash
sudo fdisk /dev/nvme1n1
```

Inside `fdisk`:

```text
n
p
1
Enter
Enter
w
```

Meaning:

- `n` → create a new partition
- `p` → primary partition
- `1` → partition number 1
- First `Enter` → accept default first sector
- Second `Enter` → use the remaining disk space
- `w` → write the changes

After that:

```bash
lsblk
```

showed:

```text
nvme1n1       100G disk
└─nvme1n1p1   100G part
```

### Important distinction

Before:

```text
/dev/nvme1n1
```

After partitioning:

```text
/dev/nvme1n1
└── /dev/nvme1n1p1
```

The disk is `/dev/nvme1n1`; the partition is `/dev/nvme1n1p1`.

---

# 6. Create the Filesystem

We chose XFS:

```bash
sudo mkfs.xfs /dev/nvme1n1p1
```

This created an XFS filesystem on the partition.

Then:

```bash
lsblk -f
```

showed:

```text
nvme1n1
└─nvme1n1p1 xfs  1db9cc22-7e70-4811-8941-2bcedde9f91b
```

The filesystem UUID was:

```text
1db9cc22-7e70-4811-8941-2bcedde9f91b
```

### What does `mkfs.xfs` do?

It creates an XFS filesystem structure on the partition so Linux can store files and directories on it.

Conceptually:

```text
Raw disk
   ↓
Partition
   ↓
Filesystem
   ↓
Files/directories
```

---

# 7. Create the Mount Point

We created:

```bash
sudo mkdir /data
```

`/data` is simply the directory where we wanted the filesystem to be accessible.

---

# 8. Mount the Filesystem

We manually mounted it:

```bash
sudo mount /dev/nvme1n1p1 /data
```

Then:

```bash
df -h
```

showed:

```text
/dev/nvme1n1p1    100G    ...    ...    ...    /data
```

This proved that the filesystem was successfully mounted.

### What does mounting mean?

Mounting connects a filesystem to a directory in Linux.

For example:

```text
/dev/nvme1n1p1
        |
        | mount
        v
       /data
```

Files written to `/data` are stored on the 100 GB EBS volume.

---

# 9. Why We Need `/etc/fstab`

A manual mount:

```bash
sudo mount /dev/nvme1n1p1 /data
```

does not necessarily survive a reboot.

After reboot, we want Linux to automatically mount the EBS filesystem.

Linux uses:

```text
/etc/fstab
```

for persistent filesystem mounts.

---

# 10. Why Use UUID Instead of Device Name?

Our filesystem UUID was:

```text
1db9cc22-7e70-4811-8941-2bcedde9f91b
```

We added:

```text
UUID=1db9cc22-7e70-4811-8941-2bcedde9f91b  /data  xfs  defaults,nofail  0  2
```

### Why UUID?

Instead of depending on:

```text
/dev/nvme1n1p1
```

we identify the filesystem using its UUID.

Device naming can potentially change depending on how devices are discovered/attached.

The UUID identifies the filesystem itself.

### Interview answer

> I prefer UUIDs in `/etc/fstab` because device names are not the best persistent identifier. The filesystem UUID provides a stable way to identify the filesystem that should be mounted.

---

# 11. Understanding the `/etc/fstab` Entry

Our entry:

```text
UUID=1db9cc22-7e70-4811-8941-2bcedde9f91b  /data  xfs  defaults,nofail  0  2
```

It has six fields:

```text
<source> <mount-point> <filesystem> <options> <dump> <fsck>
```

### Field 1 — Source

```text
UUID=1db9cc22-7e70-4811-8941-2bcedde9f91b
```

Identifies the filesystem.

### Field 2 — Mount point

```text
/data
```

Where the filesystem will be mounted.

### Field 3 — Filesystem type

```text
xfs
```

The filesystem type.

### Field 4 — Mount options

```text
defaults,nofail
```

`defaults` provides the standard mount options.

`nofail` means the system should not fail the boot process just because this filesystem is unavailable.

This is particularly useful for an additional EBS volume that is not required for the operating system itself.

### Field 5 — Dump

```text
0
```

Controls the legacy `dump` backup utility.

### Field 6 — fsck order

```text
2
```

Controls filesystem check ordering for filesystems that support `fsck`.

For XFS, this does not mean a traditional boot-time `fsck` repair is performed in the same way as ext filesystems.

---

# 12. Why `nofail` Matters in AWS

Imagine:

```text
EC2
 |
 +-- Root EBS 8 GB
 |
 +-- Data EBS 100 GB
```

If the data EBS volume becomes unavailable and `/etc/fstab` requires it without appropriate handling, boot behavior can be affected.

With:

```text
nofail
```

the instance can continue booting even if the optional data volume is unavailable.

### Interview answer

> I use `nofail` for non-root EBS mounts when appropriate so an unavailable data volume does not prevent the EC2 instance from completing boot.

---

# 13. The Most Important Safety Step: Test `fstab`

Never make an `/etc/fstab` change and immediately reboot a production server.

We first unmount the manually mounted filesystem:

```bash
sudo umount /data
```

Then ask Linux to process `/etc/fstab`:

```bash
sudo mount -a
```

If `mount -a` completes without an error, verify:

```bash
df -h /data
```

And:

```bash
findmnt /data
```

This validates that the `/etc/fstab` entry works.

### Why `mount -a`?

`mount -a` attempts to mount filesystems defined in `/etc/fstab` that are not already mounted.

It gives us a way to test the configuration before rebooting.

---

# 14. Useful Verification Commands

## See disks and partitions

```bash
lsblk
```

## See filesystem types and UUIDs

```bash
lsblk -f
```

## Check filesystem UUID

```bash
sudo blkid /dev/nvme1n1p1
```

## Check disk usage

```bash
df -h
```

## Check a specific mount

```bash
df -h /data
```

## See exactly what is mounted

```bash
findmnt /data
```

## Check `/etc/fstab`

```bash
cat /etc/fstab
```

or:

```bash
tail -n 5 /etc/fstab
```

---

# 15. Complete Lab Commands

For revision, the workflow is:

```bash
# 1. Identify disks
lsblk

# 2. Check whether the new disk has a filesystem
sudo blkid /dev/nvme1n1
sudo file -s /dev/nvme1n1

# 3. Create partition
sudo fdisk /dev/nvme1n1

# fdisk:
# n
# p
# 1
# Enter
# Enter
# w

# 4. Verify partition
lsblk

# 5. Create XFS filesystem
sudo mkfs.xfs /dev/nvme1n1p1

# 6. Verify filesystem and UUID
lsblk -f
sudo blkid /dev/nvme1n1p1

# 7. Create mount point
sudo mkdir /data

# 8. Mount
sudo mount /dev/nvme1n1p1 /data

# 9. Verify
df -h /data
findmnt /data

# 10. Edit fstab
sudo vi /etc/fstab

# Add:
UUID=<UUID>  /data  xfs  defaults,nofail  0  2

# 11. Test fstab
sudo umount /data
sudo mount -a

# 12. Verify
df -h /data
findmnt /data
```

---

# 16. What We Actually Accomplished

This lab covered the complete lifecycle of preparing a new EBS volume for Linux:

### Step 1 — Detection

AWS attached the EBS volume.

Linux detected it as:

```text
/dev/nvme1n1
```

### Step 2 — Partitioning

We created:

```text
/dev/nvme1n1p1
```

### Step 3 — Formatting

We created an XFS filesystem:

```text
/dev/nvme1n1p1 → XFS
```

### Step 4 — Mounting

We mounted it at:

```text
/data
```

### Step 5 — Persistence

We added its UUID to:

```text
/etc/fstab
```

with:

```text
defaults,nofail
```

### Step 6 — Validation

We tested the configuration using:

```bash
sudo umount /data
sudo mount -a
```

This is important because it verifies the `fstab` configuration without requiring a reboot.

---

# 17. Interview Scenario

### Question

> You have an EC2 instance with a 100 GB EBS volume attached, but `df -h` doesn't show it. What would you do?

### Strong answer

> First, I would use `lsblk` to confirm that Linux detects the volume. Then I would check whether it already has a filesystem using `blkid` or `lsblk -f`. If it is a new empty volume, I would create a partition if required, format the partition with the required filesystem such as XFS, create a mount point, and mount it. Finally, I would add the filesystem UUID to `/etc/fstab`, preferably with `nofail` for an optional data volume, and test the entry using `umount` followed by `mount -a` before rebooting.

---

# 18. Common Interview Questions

## Q1. Why does `lsblk` show a disk but `df -h` doesn't?

Because `lsblk` shows block devices, while `df` shows mounted filesystems.

## Q2. What is the difference between a disk, partition, and filesystem?

Example:

```text
/dev/nvme1n1       → disk
/dev/nvme1n1p1     → partition
XFS                → filesystem
/data              → mount point
```

## Q3. What happens if you run `mkfs.xfs` on a disk containing data?

It can destroy the existing filesystem/data. Always verify the device before formatting.

## Q4. Why use UUID in `/etc/fstab`?

It provides a stable filesystem identifier rather than relying only on a device name.

## Q5. Why use `nofail`?

It prevents an optional filesystem failure from preventing normal system boot.

## Q6. How do you test `/etc/fstab` without rebooting?

```bash
sudo mount -a
```

A common safe workflow is:

```bash
sudo umount /data
sudo mount -a
```

Then verify:

```bash
df -h /data
```

## Q7. How do you find the UUID?

```bash
blkid
```

or:

```bash
lsblk -f
```

## Q8. How do you check what is mounted at `/data`?

```bash
findmnt /data
```

or:

```bash
df -h /data
```

---

# 19. Troubleshooting Checklist

If `/data` does not mount:

### Check the device

```bash
lsblk
```

### Check filesystem

```bash
lsblk -f
```

### Check UUID

```bash
blkid /dev/nvme1n1p1
```

### Check mount directory

```bash
ls -ld /data
```

### Check `/etc/fstab`

```bash
cat /etc/fstab
```

### Test

```bash
sudo mount -a
```

### Check the error

```bash
findmnt /data
```

and inspect system logs if needed:

```bash
sudo journalctl -b
```

---

# 20. Important AWS/Linux Mental Model

Remember this chain:

```text
AWS Layer
─────────
EBS Volume
    ↓
Linux Layer
───────────
Block Device
    ↓
Partition
    ↓
Filesystem
    ↓
Mount Point
    ↓
Persistent Configuration
/etc/fstab
```

For our lab:

```text
100 GB EBS
    ↓
/dev/nvme1n1
    ↓
/dev/nvme1n1p1
    ↓
XFS
    ↓
/data
    ↓
UUID in /etc/fstab
```

If you understand this chain, you can troubleshoot most basic "I attached an EBS volume but I can't use it" scenarios.

---

# 21. Production Considerations

In a real production environment:

1. Confirm the correct EBS volume before formatting.
2. Check whether the disk contains existing data.
3. Prefer UUID or another stable identifier for persistent mounts.
4. Test `/etc/fstab` with `mount -a` before rebooting.
5. Use `nofail` when appropriate for non-critical/optional data volumes.
6. Verify the filesystem and mount point after changes.
7. Consider permissions and ownership on the mount point before giving an application access.
8. For application data, make sure backups/snapshots and recovery procedures are understood.
9. Monitor disk utilization with CloudWatch/Linux monitoring.
10. Do not assume that attaching an EBS volume automatically makes it available under a usable directory.

---

# 22. Final Revision Cheat Sheet

```text
lsblk
    → What disks/partitions exist?

blkid / lsblk -f
    → What filesystem and UUID exist?

fdisk
    → Create/manage partitions

mkfs.xfs
    → Create XFS filesystem

mkdir /data
    → Create mount point

mount
    → Attach filesystem to directory

df -h
    → Check filesystem space/usage

findmnt
    → Check what is mounted where

/etc/fstab
    → Configure persistent mounts

UUID=... /data xfs defaults,nofail 0 2
    → Persistent XFS mount

mount -a
    → Test fstab without rebooting
```

## One-line interview summary

> We attached a 100 GB EBS volume to EC2, detected it as `/dev/nvme1n1`, created `/dev/nvme1n1p1`, formatted it with XFS, mounted it on `/data`, and configured its UUID in `/etc/fstab` with `nofail` so the volume can be mounted persistently and safely across reboots.
