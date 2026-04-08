# Btrfs and Subvolumes on SLES 16

## Objectives

At the end of this training/module you will be able to:

- Understand the Btrfs filesystem and its role in SLES 16.
- Grasp the concept of subvolumes and how they replace traditional partitioning.
- Perform common storage operations using Btrfs commands.
- Manage snapshots for data protection and rollback.
- Work with multi-device Btrfs filesystems.
- Configure `/etc/fstab` entries for Btrfs subvolumes.

## Background

This is an intermediate-level module. Familiarity with Linux storage concepts (partitions, filesystems, LVM) is expected. Starting with SLES 16, the default root filesystem is Btrfs with subvolumes, replacing the traditional partition-based or LVM-based layouts used in previous SUSE releases.

## Overview

Btrfs (B-tree filesystem) is a modern copy-on-write (CoW) filesystem for Linux that provides advanced features natively — features that traditionally required separate tools like LVM, mdadm, or external snapshot utilities. Btrfs integrates volume management, snapshots, checksumming, and multi-device support directly into the filesystem layer.

In SLES 16, Btrfs is the default filesystem for the root partition. The operating system uses **subvolumes** to organize the filesystem hierarchy. Each subvolume functions like an independent filesystem branch but shares the same underlying storage pool. This approach provides flexibility similar to LVM logical volumes but with tighter integration into the filesystem itself.

### Key advantages of Btrfs over traditional layouts

- **Subvolumes** replace the need for separate partitions or LVM logical volumes.
- **Snapshots** are instantaneous and space-efficient (copy-on-write).
- **Online resize** — grow or shrink the filesystem while mounted.
- **Built-in checksumming** — detects silent data corruption.
- **Multi-device support** — span a filesystem across multiple disks with RAID profiles.
- **Online scrub** — verify data integrity without unmounting.
- **Compression** — transparent compression (zstd, lzo, zlib).

## Btrfs Concepts

### Subvolumes

A **subvolume** is a separately mountable POSIX file tree within a Btrfs filesystem. Subvolumes are not block devices — they are logical divisions of the filesystem that can be independently mounted, snapshotted, and managed. Each subvolume has its own inode namespace and can have its own mount options.

In SLES 16, the default subvolume naming convention uses the `@` prefix:

- `@` — the root subvolume
- `@/home` — the home directory
- `@/var` — variable data
- `@/.snapshots` — snapshot storage
- `@/opt`, `@/srv`, `@/root`, `@/usr/local`, etc.

### Snapshots

A **snapshot** is a special type of subvolume that shares its data with another subvolume using Btrfs copy-on-write semantics. At creation time, a snapshot occupies almost no additional space. As the source or snapshot diverges, only the changed blocks consume additional storage.

Snapshots can be:

- **Read-only** — for backup and archival purposes.
- **Read-write** — for testing or rollback scenarios.

### Copy-on-Write (CoW)

Btrfs never overwrites data in place. When a block is modified, a new copy is written to a different location, and the metadata is updated to point to the new block. The old block remains available until it is no longer referenced. This is the foundation for snapshots and data integrity.

#### How CoW enables snapshots

Traditional filesystems like XFS or ext4 overwrite blocks in place. If you want a snapshot, you must copy all the data first — otherwise the original is lost when it is modified. Btrfs works differently because of CoW:

1. **At snapshot creation**, the snapshot and the source subvolume point to the exact same data blocks. No data is copied — only the metadata references are duplicated. This is why a snapshot is instantaneous and initially consumes almost no space.

2. **When the source is modified**, Btrfs does not overwrite the existing block. Instead, it writes the new data to a free location and updates only the source's metadata to point to the new block. The snapshot's metadata still points to the original block.

3. **When the snapshot is modified** (read-write snapshots only), the same principle applies: the modified block is written to a new location and only the snapshot's metadata is updated.

4. **Space is consumed only for blocks that diverge** between the source and the snapshot. Blocks that remain identical are shared and stored once. This means a snapshot of a 50 GB subvolume where only 2 GB has changed since the snapshot was taken consumes roughly 2 GB of additional space — not 50 GB.

5. **Deletion of a snapshot** does not affect the source. Btrfs tracks reference counts on each data block. A block is freed only when no subvolume or snapshot references it anymore.

This behavior contrasts with LVM snapshots, which use a separate copy-on-write pool that can fill up and break the snapshot. Btrfs CoW is integrated at the filesystem level and shares the same storage pool, making it simpler to manage and monitor.

### Scrub

A **scrub** is an online data integrity verification operation. Btrfs stores a checksum (CRC32C by default) for every data and metadata block. When you run `btrfs scrub start /mnt`, the filesystem reads every block from disk and compares it against its stored checksum. If a block does not match its checksum, the data has been silently corrupted — a condition known as **bit rot** that traditional filesystems like XFS or ext4 cannot detect at all.

On a single-device filesystem, the scrub reports the corruption so you know the data is damaged. On a multi-device filesystem with a redundant profile (RAID1, RAID10), the scrub automatically repairs the corruption by copying the good block from the mirror.

Scrubs run online — the filesystem remains mounted and available during the operation. They generate I/O load, so on production systems you may want to schedule them during low-activity windows. A typical practice is running a monthly scrub via cron or systemd timer.

This has no equivalent in XFS or ext4. The closest traditional tool is `fsck`, but `fsck` requires the filesystem to be unmounted and does not verify data checksums — it only checks metadata structural integrity.

### Defragmentation

Because Btrfs uses copy-on-write, it never modifies data in place. Every write to an existing block creates a new block at a different physical location. Over time, this causes **fragmentation**: files that were originally contiguous on disk become scattered across many non-adjacent locations. This fragmentation increases seek times and degrades sequential read performance, especially on spinning disks (HDDs).

The `btrfs filesystem defragment` command rewrites file data into contiguous extents, improving sequential read performance. You can defragment individual files, directories, or entire subvolumes:

```bash
btrfs filesystem defragment /mnt/somefile
btrfs filesystem defragment -r /mnt/somedir    # recursive
```

**Important:** Defragmenting a file that is shared between snapshots (via CoW) breaks the sharing. The defragmented file is rewritten as a new, independent copy. This means defragmenting a subvolume that has snapshots will increase disk usage because the shared blocks are duplicated. Only defragment when the performance benefit outweighs the space cost.

On SSDs, fragmentation has less impact because random I/O is fast, so defragmentation is rarely needed. The `autodefrag` mount option enables background defragmentation of small random writes, which is useful for workloads like databases on HDDs.

### Online Resize

Btrfs resize operations (`btrfs filesystem resize`) require the filesystem to be **mounted**. This is the opposite of traditional tools like `resize2fs` (ext4), which can work on unmounted filesystems. The reason is architectural: Btrfs resize works through the filesystem's own allocator. When growing, the filesystem extends its internal space maps to include the new blocks. When shrinking, it must actively relocate data out of the space being released, which requires the allocator and CoW machinery to be running.

This design means resize is always **online** — you never need to unmount to grow or shrink. It also means Btrfs can **shrink** a filesystem, which XFS cannot do at all.

```bash
btrfs filesystem resize +5G /mnt    # grow by 5G
btrfs filesystem resize -2G /mnt    # shrink by 2G
btrfs filesystem resize max /mnt    # grow to fill the entire device
```

### Multi-Device Support

A Btrfs filesystem can span multiple block devices. Unlike LVM where you create physical volumes and volume groups, Btrfs manages multiple devices directly. You can add or remove devices from a mounted filesystem and choose data/metadata distribution profiles.

#### Data and metadata profiles

Btrfs separates the allocation strategy for data and metadata. Each can use a different profile:

| Profile | Description | Equivalent |
|---|---|---|
| `single` | Each block is stored on one device. Space from all devices is pooled together but blocks are not split across devices. This is **not striping** — it is simple concatenation, similar to LVM linear mode (`lvcreate` without `-i`). | LVM linear / JBOD |
| `dup` | Each block is stored twice on the **same** device. Used for metadata by default on single-device filesystems to protect against localized corruption. | No direct equivalent |
| `raid0` | Data is striped across all devices. This **is** striping — blocks are split and distributed evenly across devices for higher throughput. There is no redundancy; losing one device loses the entire filesystem. | mdadm RAID0 / LVM striping (`lvcreate -i`) |
| `raid1` | Each block is stored on two different devices (mirroring). Survives one device failure. | mdadm RAID1 / LVM mirroring |
| `raid10` | Striped mirrors — combines RAID1 mirroring with RAID0 striping. Requires at least 4 devices. | mdadm RAID10 |

By default, when you add a device to an existing Btrfs filesystem, the data profile is `single` and the metadata profile is `dup` (single device) or `raid1` (multiple devices). You can change profiles with:

```bash
btrfs balance start -dconvert=raid0 -mconvert=raid1 /mnt
```

#### Is adding a device the same as striping?

**No.** Adding a device with `btrfs device add` uses the `single` profile by default, which is concatenation — all device space is pooled, but each block lives entirely on one device. New writes go to the device with the most free space. This is equivalent to `vgextend` in LVM (adding a PV to a volume group), not to striping.

To get actual striping (RAID0), you must explicitly convert the data profile after adding the device:

```bash
btrfs device add /dev/sdY /mnt
btrfs balance start -dconvert=raid0 /mnt
```


#### Balance operation

The `btrfs balance` command redistributes existing data across devices according to the current profile. After adding a new device, existing data remains on the old device — only new writes use the new space. Running a balance moves existing data to achieve even distribution. On large filesystems this can be I/O intensive and take a long time.

## SLES 16 Default Btrfs Layout

On a default SLES 16 installation, the root partition uses Btrfs with the following subvolume structure:

| Subvolume | Mount Point | Purpose |
|---|---|---|
| `@` | `/` | Root filesystem |
| `@/.snapshots` | `/.snapshots` | Snapper snapshot storage |
| `@/home` | `/home` | User home directories |
| `@/opt` | `/opt` | Optional software packages |
| `@/root` | `/root` | Root user home directory |
| `@/srv` | `/srv` | Service data |
| `@/var` | `/var` | Variable data (logs, spool, cache) |
| `@/boot/writable` | `/boot/writable` | Writable boot content |
| `@/usr/local` | `/usr/local` | Locally installed software |
| `@/boot/grub2/i386-pc` | `/boot/grub2/i386-pc` | GRUB2 BIOS modules |
| `@/boot/grub2/x86_64-efi` | `/boot/grub2/x86_64-efi` | GRUB2 EFI modules |

All subvolumes share the same UUID and the same underlying Btrfs filesystem on a single partition. Each is mounted separately via `/etc/fstab` with the `subvol=` option.

Example fstab entries from a SLES 16 installation:

```bash
UUID=c72a5daa-6dc6-4a72-a193-245fbd42b2c2 /           btrfs defaults 0 0
UUID=c72a5daa-6dc6-4a72-a193-245fbd42b2c2 /.snapshots  btrfs defaults,subvol=@/.snapshots 0 0
UUID=c72a5daa-6dc6-4a72-a193-245fbd42b2c2 /home        btrfs defaults,subvol=@/home 0 0
UUID=c72a5daa-6dc6-4a72-a193-245fbd42b2c2 /var         btrfs defaults,subvol=@/var 0 0
```

**Why the sixth field is `0` (not `1` or `2`):**

The sixth field in `/etc/fstab` controls `fsck` pass order at boot — `1` means check first (root), `2` means check after root, and `0` means skip. For traditional filesystems like ext4 or XFS, `fsck` verifies and repairs metadata structures while the filesystem is unmounted.

Btrfs does **not** use this field because:

- `fsck.btrfs` exists but is intentionally a **no-op**. It prints a message and exits because Btrfs does its own integrity management through its transactional CoW design.
- Btrfs uses a **transaction log** and **CoW metadata updates**, so on-disk structures are always consistent — there is no "dirty" state that needs a traditional `fsck` repair after an unclean shutdown.
- The equivalent integrity check for Btrfs is `btrfs scrub` (online, verifies checksums) or `btrfs check` (offline, structural verification). Neither is triggered by the fstab field.

Setting the sixth field to `1` or `2` for Btrfs is harmless (the no-op `fsck.btrfs` simply exits), but it is set to `0` because running it serves no purpose and adds unnecessary boot time.

## Operations Comparison: LVM / Partitions vs Btrfs

| Operation | LVM / Partitions | Btrfs Equivalent |
|---|---|---|
| Create a filesystem | `mkfs.xfs /dev/sdX` | `mkfs.btrfs /dev/sdX` |
| Create a logical division | `lvcreate -n mylv vg0` | `btrfs subvolume create /mnt/mysubvol` |
| List logical divisions | `lvs` | `btrfs subvolume list /mnt` |
| Delete a logical division | `lvremove /dev/vg0/mylv` | `btrfs subvolume delete /mnt/mysubvol` |
| Create a snapshot | `lvcreate -s -n snap /dev/vg0/mylv` | `btrfs subvolume snapshot /src /dst` |
| Extend storage pool | `vgextend vg0 /dev/sdY` | `btrfs device add /dev/sdY /mnt` |
| Resize filesystem (grow) | `lvextend + xfs_growfs` | `btrfs filesystem resize +5G /mnt` |
| Resize filesystem (shrink) | `resize2fs` (ext4 only) | `btrfs filesystem resize -5G /mnt` |
| Check storage usage | `pvs`, `vgs`, `lvs`, `df -h` | `btrfs filesystem usage /mnt` |
| Show devices | `pvdisplay` | `btrfs filesystem show /mnt` |
| Remove a device | `vgreduce vg0 /dev/sdY` | `btrfs device remove /dev/sdY /mnt` |
| Online integrity check | N/A | `btrfs scrub start /mnt` |
| Balance data across devices | N/A | `btrfs balance start /mnt` |
| Enable compression | N/A (filesystem level) | `mount -o compress=zstd` |

## Key Btrfs Commands

### Filesystem Operations

```bash
# Create a Btrfs filesystem
mkfs.btrfs /dev/sdX

# Create with a label
mkfs.btrfs -L mydisk /dev/sdX

# Show filesystem information
btrfs filesystem show
btrfs filesystem show /mnt

# Show detailed space usage
btrfs filesystem usage /mnt
btrfs filesystem df /mnt

# Resize the filesystem (must be mounted)
btrfs filesystem resize +5G /mnt    # grow by 5G
btrfs filesystem resize -2G /mnt    # shrink by 2G
btrfs filesystem resize max /mnt    # use all available space
```

### Subvolume Operations

```bash
# Create a subvolume
btrfs subvolume create /mnt/mysubvol

# List subvolumes
btrfs subvolume list /mnt
btrfs subvolume list -t /mnt    # table format

# Show subvolume details
btrfs subvolume show /mnt/mysubvol

# Delete a subvolume
btrfs subvolume delete /mnt/mysubvol

# Get the default subvolume
btrfs subvolume get-default /mnt

# Set the default subvolume
btrfs subvolume set-default <subvolid> /mnt
```

### Snapshot Operations

```bash
# Create a read-write snapshot
btrfs subvolume snapshot /mnt/source /mnt/snap1

# Create a read-only snapshot
btrfs subvolume snapshot -r /mnt/source /mnt/snap1-ro

# Delete a snapshot (same as deleting a subvolume)
btrfs subvolume delete /mnt/snap1
```

### Multi-Device Operations

```bash
# Add a device to an existing mounted filesystem
btrfs device add /dev/sdY /mnt

# Remove a device from a filesystem
btrfs device remove /dev/sdY /mnt

# Show device usage per device
btrfs device usage /mnt

# Balance data across devices after adding/removing
btrfs balance start /mnt

# Check balance status
btrfs balance status /mnt

# Convert data profile to RAID0 (striping)
btrfs balance start -dconvert=raid0 /mnt

# Convert data to single, metadata to RAID1
btrfs balance start -dconvert=single -mconvert=raid1 /mnt
```

#### Understanding `btrfs device usage` output

The `btrfs device usage` command shows how space is allocated on each device. Example:

```
/dev/sda3, ID: 1
   Device size:            29.50GiB
   Device slack:            3.50KiB
   Data,single:             5.01GiB
   Metadata,DUP:          512.00MiB
   System,DUP:             16.00MiB
   Unallocated:            23.97GiB
```

| Field | Meaning |
|---|---|
| **Device size** | Total size of the block device as seen by Btrfs. |
| **Device slack** | Unusable space at the end of the device that is too small for Btrfs to allocate (alignment waste). This is normal and negligible. |
| **Data,single** | Space allocated for data blocks. The `single` profile means each data block is stored once on this device (no duplication or striping). |
| **Metadata,DUP** | Space allocated for metadata (B-tree nodes, inode info, extent maps). The `DUP` profile means metadata is stored **twice on the same device** for resilience against localized corruption. This is the default for single-device filesystems. On multi-device setups, metadata defaults to `raid1` instead. |
| **System,DUP** | Space allocated for the system chunk — a small internal structure that tells Btrfs where to find the rest of the metadata. Also duplicated for safety. |
| **Unallocated** | Space on the device not yet assigned to any purpose. Btrfs allocates space in chunks (typically 1 GiB for data, 256 MiB for metadata). Unallocated space is the pool from which new chunks are carved as the filesystem fills up. |

Btrfs allocates space in a two-level system: first device space is carved into **chunks** (shown here), then within each chunk, individual blocks are allocated for files and metadata. The `Unallocated` space does not mean wasted — it is available for future data or metadata chunks as needed.

### Maintenance Operations

```bash
# Start an online scrub (verify all checksums)
btrfs scrub start /mnt

# Check scrub status and results
btrfs scrub status /mnt

# Defragment a file (rewrites into contiguous extents)
btrfs filesystem defragment /mnt/somefile

# Defragment a directory recursively (caution: breaks snapshot sharing)
btrfs filesystem defragment -r /mnt/somedir

# Defragment with compression
btrfs filesystem defragment -r -czstd /mnt/somedir

# Offline filesystem check (unmounted only — not equivalent to fsck)
btrfs check /dev/sdX

# Offline check with repair (use only as last resort, can cause data loss)
# btrfs check --repair /dev/sdX
```

### Mounting Subvolumes

```bash
# Mount the default subvolume
mount /dev/sdX /mnt

# Mount a specific subvolume by name
mount -o subvol=@/home /dev/sdX /mnt/home

# Mount a specific subvolume by ID
mount -o subvolid=258 /dev/sdX /mnt/home

# Mount with compression
mount -o compress=zstd /dev/sdX /mnt

# Remount with different options
mount -o remount,compress=zstd /mnt
```

### fstab Entries for Subvolumes

Each subvolume that needs to be independently mounted requires its own fstab entry. All entries share the same UUID since they reside on the same Btrfs filesystem:

```bash
UUID=<btrfs-uuid>  /mnt/data         btrfs  defaults,nofail 0 0
UUID=<btrfs-uuid>  /mnt/data/logs    btrfs  defaults,nofail,subvol=logs 0 0
UUID=<btrfs-uuid>  /mnt/data/app     btrfs  defaults,nofail,subvol=app 0 0
```

Common mount options for Btrfs:

| Option | Description |
|---|---|
| `subvol=<path>` | Mount a specific subvolume by path |
| `subvolid=<id>` | Mount a specific subvolume by ID |
| `compress=zstd` | Enable zstd compression |
| `noatime` | Do not update access times (performance) |
| `space_cache=v2` | Use free space cache v2 (default in recent kernels) |
| `autodefrag` | Enable automatic defragmentation |
| `nofail` | Do not fail boot if the device is not present |
| `_netdev` | Wait for network before mounting (for network-backed devices) |

## Btrfs vs LVM: When to Use What

### Use Btrfs subvolumes when

- The distribution uses Btrfs by default (SLES 16).
- You need fast, space-efficient snapshots.
- You want integrated checksumming and data integrity.
- You need online scrub capability.
- You want to avoid the complexity of managing a separate LVM layer.

### Use LVM when

- The distribution or workload requires XFS or ext4.
- You need thin provisioning across many logical volumes.
- You need LVM-based encryption (LUKS on LV).
- Existing infrastructure and automation is built around LVM.
- You need striping or mirroring with fine-grained control separate from the filesystem.

## Important Considerations for Azure

- **Device names are not persistent across reboots.** Use `/dev/disk/azure/scsi1/lunN` or UUID-based references in fstab.
- **Btrfs RAID is not recommended for production** in Azure. Azure managed disks already provide redundancy at the storage layer. Use Btrfs in `single` profile for data disks.
- **Disk resize propagation**: After resizing an Azure data disk, use `btrfs filesystem resize max /mnt` to extend the Btrfs filesystem to use the new space — no intermediate steps like `pvresize` are needed.
- **Snapshots consume space over time** as data diverges. Monitor usage with `btrfs filesystem usage`.

## References

[Btrfs Wiki](https://btrfs.readthedocs.io/en/latest/)

[SUSE Documentation — Btrfs](https://documentation.suse.com/sles/16.0/)

[Btrfs Administration Guide](https://btrfs.readthedocs.io/en/latest/Administration.html)

[Azure Linux VM — Attach Data Disks](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/attach-disk-portal)
