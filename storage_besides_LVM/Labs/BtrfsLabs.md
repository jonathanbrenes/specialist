# Btrfs Subvolumes on SLES 16 — Laboratories

## Title and Scenario

These hands-on laboratories accompany the Btrfs and Subvolumes on SLES 16 module. The labs cover core Btrfs storage operations on Azure Linux VMs: understanding the default SLES 16 subvolume layout, creating and managing subvolumes on data disks, working with Btrfs snapshots, and performing multi-device filesystem operations. Each lab builds on practical, real-world storage tasks encountered when managing SLES 16 virtual machines on Azure.

## Deployment

All lab deployments are consolidated in this section. Use the button below to deploy the required Azure infrastructure.

**Btrfs Labs:** Deploy one SLES 16 VM with data disks for Btrfs exercises.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fstorage_besides_LVM%2FLabs%2FBtrfsLab1.json)

## Skills Required

- Linux command-line proficiency (shell navigation, file operations, text editing with `vi`)
- Familiarity with Linux storage concepts (block devices, filesystems, mount points)
- Basic understanding of `/etc/fstab` and persistent mounts
- Experience connecting to Linux VMs via SSH
- Ability to navigate the Azure Portal (VMs, disks)

## Recommended Prerequisites

Completion of the Btrfs and Subvolumes on SLES 16 knowledge module or equivalent experience is recommended before starting these labs. Familiarity with LVM concepts is helpful for understanding the comparison between Btrfs subvolumes and LVM logical volumes.

## Objectives

- Explore and understand the default SLES 16 Btrfs subvolume layout.
- Create a Btrfs filesystem on a data disk and manage subvolumes.
- Configure `/etc/fstab` entries for Btrfs subvolumes.
- Create read-only and read-write snapshots and verify data integrity.
- Add a second data disk to an existing Btrfs filesystem and balance data across devices.
- Resize a Btrfs filesystem after an Azure disk resize.

## Environment Overview

These labs use an Azure SLES 16 virtual machine with attached data disks. The VM runs SUSE Linux Enterprise Server 16.0 with a Btrfs root filesystem and subvolume-based layout. Data disks are attached for hands-on exercises. All labs require SSH access and root privileges.

## Your Mission

Complete each lab sequentially. Begin by exploring the default SLES 16 Btrfs layout to understand how subvolumes replace traditional partitions. Then create and manage your own subvolumes on data disks, work with snapshots for data protection, and perform multi-device operations including adding disks and resizing. By the end, you should be comfortable with Btrfs as a primary storage management tool on SLES 16.

## Lab 1: Exploring the SLES 16 Btrfs Layout

### About this Lab

- This lab introduces the default Btrfs subvolume layout on SLES 16.
- It will take approximately 20 minutes.
- After this lab you will be able to:
    - Identify the Btrfs filesystem and subvolume structure on a SLES 16 VM.
    - Interpret `/etc/fstab` entries for Btrfs subvolumes.
    - Use Btrfs commands to inspect filesystem properties and usage.

### Instructions

Deployment: See the [Deployment](#deployment) section.

- Connect to the VM and switch to root account:

  ```bash
  sudo -i
  ```

- Verify the SLES version:

  ```bash
  cat /etc/*release
  ```

- Examine the block device layout:

  ```bash
  lsblk
  lsblk -f
  ```

  Observe how the root partition uses Btrfs and how multiple mount points share the same partition. Compare this to a traditional layout where each mount point would require a separate partition or LVM logical volume.

- Examine the `/etc/fstab` file:

  ```bash
  cat /etc/fstab
  ```

  Notice that all Btrfs entries share the same UUID. Each line mounts a different subvolume using the `subvol=` option. The root entry (`/`) mounts the default subvolume without specifying a subvolume path.

- List all Btrfs subvolumes on the root filesystem:

  ```bash
  btrfs subvolume list /
  ```

  This shows every subvolume with its ID, generation number, and path. The `@` prefix is the SLES convention for naming subvolumes.

- List subvolumes in table format for a cleaner view:

  ```bash
  btrfs subvolume list -t /
  ```

- Check the default subvolume:

  ```bash
  btrfs subvolume get-default /
  ```

- Show detailed information about a specific subvolume:

  ```bash
  btrfs subvolume show /
  btrfs subvolume show /home
  btrfs subvolume show /var
  ```

- Examine the Btrfs filesystem information:

  ```bash
  btrfs filesystem show /
  ```

  This displays the filesystem label, UUID, total size, and device information.

- Check detailed space usage:

  ```bash
  btrfs filesystem usage /
  btrfs filesystem df /
  ```

  Compare the output of `btrfs filesystem usage` with the traditional `df -h` output. The Btrfs command provides more detailed information about data, metadata, and system space allocation.

  ```bash
  df -h /
  ```

- Inspect the Azure disk symlinks to understand device naming:

  ```bash
  ls -lR /dev/disk/azure
  ```

### Your Goal

At the end of this lab, you should understand how SLES 16 organizes its root filesystem using Btrfs subvolumes, how to read fstab entries for subvolume mounts, and how to use Btrfs commands to inspect filesystem properties.

## Lab 2: Creating and Managing Btrfs Subvolumes

### About this Lab

- This lab covers creating a Btrfs filesystem on a data disk and managing subvolumes.
- It will take approximately 30 minutes.
- After this lab you will be able to:
    - Create a Btrfs filesystem on a data disk.
    - Create, list, and delete subvolumes.
    - Mount subvolumes and configure persistent mounts via `/etc/fstab`.

### Instructions

Deployment: See the [Deployment](#deployment) section.

- Connect to the VM and switch to root account:

  ```bash
  sudo -i
  ```

- Identify the data disks attached to the VM:

  ```bash
  lsblk
  lsblk -f
  ls -lR /dev/disk/azure
  ```

  Look for the data disk that has no filesystem. Note its Azure persistent path under `/dev/disk/azure/scsi1/`.

- Create a Btrfs filesystem on the first data disk. Use a label to identify it:

  ```bash
  mkfs.btrfs -L btrfsdata /dev/disk/azure/scsi1/lun0
  ```

- Verify the filesystem was created:

  ```bash
  btrfs filesystem show
  lsblk -f
  ```

- Create a mount point and mount the new Btrfs filesystem:

  ```bash
  mkdir -p /mnt/btrfsdata
  mount /dev/disk/azure/scsi1/lun0 /mnt/btrfsdata
  ```

  > **Note:** This is a temporary mount used as a workspace to create subvolumes. It will not persist after a reboot. Persistent fstab entries for the individual subvolumes are added later in this lab.

- Verify the mount:

  ```bash
  df -h /mnt/btrfsdata
  mount | grep btrfsdata
  btrfs filesystem usage /mnt/btrfsdata
  ```

- Create subvolumes on the data disk. These act similarly to LVM logical volumes — each can be independently mounted and managed:

  ```bash
  btrfs subvolume create /mnt/btrfsdata/appdata
  btrfs subvolume create /mnt/btrfsdata/logs
  btrfs subvolume create /mnt/btrfsdata/backups
  ```

- List the subvolumes and verify:

  ```bash
  btrfs subvolume list /mnt/btrfsdata
  btrfs subvolume list -t /mnt/btrfsdata
  ```

- Show detailed information about a subvolume:

  ```bash
  btrfs subvolume show /mnt/btrfsdata/appdata
  ```

- Create mount points for the subvolumes and mount them individually. First, get the UUID of the Btrfs filesystem:

  ```bash
  blkid /dev/disk/azure/scsi1/lun0
  ```

- Create mount points and mount each subvolume:

  ```bash
  mkdir -p /data/app /data/logs /data/backups
  mount -o subvol=appdata /dev/disk/azure/scsi1/lun0 /data/app
  mount -o subvol=logs /dev/disk/azure/scsi1/lun0 /data/logs
  mount -o subvol=backups /dev/disk/azure/scsi1/lun0 /data/backups
  ```

- Verify all mounts:

  ```bash
  df -h | grep data
  mount | grep btrfs
  ```

- Write test data to each subvolume:

  ```bash
  dd if=/dev/urandom of=/data/app/testfile.dat bs=1M count=100
  md5sum /data/app/testfile.dat
  echo "Log entry $(date)" > /data/logs/app.log
  cp /data/app/testfile.dat /data/backups/
  md5sum /data/backups/testfile.dat
  ```

- Add persistent fstab entries for the subvolumes. Use the UUID obtained earlier:

  ```bash
  echo "UUID=<btrfs-uuid>  /data/app      btrfs  defaults,nofail,subvol=appdata  0 0" >> /etc/fstab
  echo "UUID=<btrfs-uuid>  /data/logs     btrfs  defaults,nofail,subvol=logs     0 0" >> /etc/fstab
  echo "UUID=<btrfs-uuid>  /data/backups  btrfs  defaults,nofail,subvol=backups  0 0" >> /etc/fstab
  ```

- Verify the fstab entries:

  ```bash
  cat /etc/fstab | grep btrfs
  ```

- Test the fstab entries by unmounting and remounting:

  ```bash
  umount /data/app /data/logs /data/backups
  mount -a
  df -h | grep data
  ```

- Verify data integrity after remount:

  ```bash
  md5sum /data/app/testfile.dat
  md5sum /data/backups/testfile.dat
  ```

- Delete a subvolume (optional — to practice the operation, create a temporary one first):

  ```bash
  btrfs subvolume create /mnt/btrfsdata/temp
  btrfs subvolume list /mnt/btrfsdata
  btrfs subvolume delete /mnt/btrfsdata/temp
  btrfs subvolume list /mnt/btrfsdata
  ```

### Your Goal

At the end of this lab, you should be able to create a Btrfs filesystem on a data disk, create and manage subvolumes, mount them independently, configure persistent mounts via `/etc/fstab`, and verify data integrity.

## Lab 3: Btrfs Snapshots

### About this Lab

- This lab covers Btrfs snapshot creation, verification, and management.
- It will take approximately 25 minutes.
- After this lab you will be able to:
    - Create read-only and read-write snapshots.
    - Verify data integrity using snapshots.
    - Understand snapshot space usage.
    - Restore data from a snapshot.

### Instructions

This lab uses the Btrfs filesystem and subvolumes created in Lab 2.

- Connect to the VM and switch to root account:

  ```bash
  sudo -i
  ```

- Verify the subvolumes from Lab 2 are mounted:

  ```bash
  btrfs subvolume list /mnt/btrfsdata
  df -h | grep data
  ```

- Create a read-only snapshot of the `appdata` subvolume:

  ```bash
  btrfs subvolume snapshot -r /mnt/btrfsdata/appdata /mnt/btrfsdata/appdata-snap-ro
  ```

- Create a read-write snapshot of the `appdata` subvolume:

  ```bash
  btrfs subvolume snapshot /mnt/btrfsdata/appdata /mnt/btrfsdata/appdata-snap-rw
  ```

- List all subvolumes to see the snapshots:

  ```bash
  btrfs subvolume list /mnt/btrfsdata
  btrfs subvolume list -s /mnt/btrfsdata
  ```

  The `-s` flag lists only snapshots.

- Verify the snapshot contains the same data as the source:

  ```bash
  md5sum /mnt/btrfsdata/appdata/testfile.dat
  md5sum /mnt/btrfsdata/appdata-snap-ro/testfile.dat
  md5sum /mnt/btrfsdata/appdata-snap-rw/testfile.dat
  ```

  All three md5sums should match.

- Check space usage. Snapshots initially consume almost no space:

  ```bash
  btrfs filesystem usage /mnt/btrfsdata
  ```

- Modify data in the original subvolume to observe CoW behavior:

  ```bash
  dd if=/dev/urandom of=/data/app/newfile.dat bs=1M count=50
  echo "Modified after snapshot" >> /data/app/testfile.dat
  ```

- Verify the snapshot still has the original data (unchanged):

  ```bash
  md5sum /mnt/btrfsdata/appdata-snap-ro/testfile.dat
  ls -l /mnt/btrfsdata/appdata-snap-ro/
  ```

  The snapshot should NOT contain `newfile.dat`, and `testfile.dat` should have its original checksum.

- Try writing to the read-only snapshot (this should fail):

  ```bash
  touch /mnt/btrfsdata/appdata-snap-ro/should-fail.txt
  ```

- Write to the read-write snapshot (this should succeed):

  ```bash
  touch /mnt/btrfsdata/appdata-snap-rw/new-in-snapshot.txt
  ls -l /mnt/btrfsdata/appdata-snap-rw/
  ```

- Check space usage again after modifications. Notice how the divergence between source and snapshot now consumes additional space:

  ```bash
  btrfs filesystem usage /mnt/btrfsdata
  ```

- Simulate a data restore from snapshot. Delete a file from the original and restore it from the read-only snapshot:

  ```bash
  rm /data/app/testfile.dat
  ls /data/app/
  cp /mnt/btrfsdata/appdata-snap-ro/testfile.dat /data/app/
  md5sum /data/app/testfile.dat
  ```

- Clean up snapshots:

  ```bash
  btrfs subvolume delete /mnt/btrfsdata/appdata-snap-ro
  btrfs subvolume delete /mnt/btrfsdata/appdata-snap-rw
  btrfs subvolume list /mnt/btrfsdata
  ```

### Your Goal

At the end of this lab, you should understand the difference between read-only and read-write snapshots, how copy-on-write preserves snapshot data, how to verify data integrity across snapshots, and how to restore data from a snapshot.

## Lab 4: Multi-Device Btrfs and Disk Resize

### About this Lab

- This lab covers adding devices to a Btrfs filesystem, balancing data, and resizing after an Azure disk resize.
- It will take approximately 30 minutes.
- After this lab you will be able to:
    - Add a data disk to an existing Btrfs filesystem.
    - Balance data across multiple devices.
    - Resize a Btrfs filesystem after an Azure disk resize.
    - Remove a device from a Btrfs filesystem.

### Instructions

This lab uses the Btrfs filesystem created in Lab 2 and requires a second data disk.

- Connect to the VM and switch to root account:

  ```bash
  sudo -i
  ```

- Identify all available disks:

  ```bash
  lsblk
  ls -lR /dev/disk/azure
  ```

  Identify the second data disk (it should have no filesystem).

- Check the current Btrfs filesystem devices:

  ```bash
  btrfs filesystem show /mnt/btrfsdata
  btrfs device usage /mnt/btrfsdata
  ```

  Note the current total size and the single device.

- Add the second data disk to the existing Btrfs filesystem:

  ```bash
  btrfs device add /dev/disk/azure/scsi1/lun1 /mnt/btrfsdata
  ```

  **Note:** Unlike LVM where you need to create a physical volume (`pvcreate`), then extend a volume group (`vgextend`), and then extend the logical volume (`lvextend`), Btrfs adds the device and makes the space available in a single command.

- Verify the device was added:

  ```bash
  btrfs filesystem show /mnt/btrfsdata
  btrfs device usage /mnt/btrfsdata
  lsblk
  ```

- Check the filesystem size — it should reflect the combined capacity:

  ```bash
  btrfs filesystem usage /mnt/btrfsdata
  df -h /mnt/btrfsdata
  ```

- The data currently resides only on the first device. Balance the data across both devices:

  ```bash
  btrfs balance start /mnt/btrfsdata
  ```

  For large filesystems, you can check the balance status:

  ```bash
  btrfs balance status /mnt/btrfsdata
  ```

- After the balance completes, verify data distribution:

  ```bash
  btrfs device usage /mnt/btrfsdata
  ```

- Verify data integrity after the balance:

  ```bash
  md5sum /data/app/testfile.dat
  ```

- Run an online scrub to verify all checksums:

  ```bash
  btrfs scrub start /mnt/btrfsdata
  btrfs scrub status /mnt/btrfsdata
  ```

- Now simulate an Azure disk resize. Go to the Azure Portal, stop the VM and resize `LUN 0` (the first data disk). Increase it by a few GB. Start the VM, reconnect, and switch to root.

- After the VM is back, verify the disk size change at OS level:

  ```bash
  lsblk
  btrfs filesystem show /mnt/btrfsdata
  ```

  The Btrfs filesystem still shows the old size. Unlike LVM where you need to run `pvresize` and then `lvextend` and then `xfs_growfs`, with Btrfs you only need one command:

  ```bash
  btrfs filesystem resize max /mnt/btrfsdata
  ```

  Alternatively, resize by a specific amount:

  ```bash
  btrfs filesystem resize +2G /mnt/btrfsdata
  ```

- Verify the resize:

  ```bash
  btrfs filesystem show /mnt/btrfsdata
  btrfs filesystem usage /mnt/btrfsdata
  df -h /mnt/btrfsdata
  ```

- Verify data integrity after the resize:

  ```bash
  md5sum /data/app/testfile.dat
  ```

- Optional: Remove the second device from the filesystem. Btrfs will automatically move all data from that device to the remaining device(s) before removal. The remaining device must have enough free space:

  ```bash
  btrfs device remove /dev/disk/azure/scsi1/lun1 /mnt/btrfsdata
  ```

  This operation may take time depending on data volume. Check progress:

  ```bash
  btrfs device usage /mnt/btrfsdata
  btrfs filesystem show /mnt/btrfsdata
  ```

  **Note:** Remember to delete all the resources once you complete the laboratories.

### Your Goal

At the end of this lab, you should be able to add and remove devices from a Btrfs filesystem, balance data across multiple devices, resize a Btrfs filesystem after an Azure disk resize, and verify data integrity throughout these operations.

### References

[Btrfs Wiki — Multiple Devices](https://btrfs.readthedocs.io/en/latest/mkfs.btrfs.html)

[SUSE Documentation — Btrfs](https://documentation.suse.com/sles/16.0/)

[Azure — Expand Virtual Hard Disks on a Linux VM](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/expand-disks)

---

## Analytical Guidance

- Begin with Lab 1 to build familiarity with the SLES 16 Btrfs layout before modifying anything.
- Always verify device identity using `lsblk` and `/dev/disk/azure/` before performing operations — Azure does not guarantee persistent device names across reboots.
- After every snapshot or balance operation, verify data integrity using `md5sum` on test files.
- When comparing Btrfs to LVM, pay attention to the reduced number of steps needed for common operations (e.g., resize is one command instead of three).
- Use `btrfs filesystem usage` instead of `df -h` for accurate space reporting on Btrfs — the traditional `df` output can be misleading with CoW filesystems.
- When adding or removing devices, ensure the remaining device(s) have enough space for all data.
- Run `btrfs scrub` after multi-device operations to verify data integrity at the block level.

## Validation Criteria

After completing all labs, you should be able to:

- Describe the SLES 16 Btrfs default subvolume layout and explain each subvolume's purpose.
- Create a Btrfs filesystem on a data disk and create multiple subvolumes.
- Mount subvolumes independently and configure persistent mounts via `/etc/fstab`.
- Create read-only and read-write snapshots and explain the difference.
- Restore data from a snapshot.
- Add a second device to an existing Btrfs filesystem and balance data.
- Resize a Btrfs filesystem after an Azure disk resize using a single command.
- Use `btrfs scrub` to verify data integrity.

## Documentation Expectations

- Record the output of `btrfs subvolume list` and `btrfs filesystem show` at each stage of the labs.
- Note `md5sum` values before and after snapshot, balance, and resize operations to document data integrity.
- Document the fstab entries created for subvolume mounts.
- Capture the output of `btrfs filesystem usage` before and after adding a device to show space changes.
- Compare the number of steps required for common operations between Btrfs and LVM.

## What Not To Do

- Do not use `mkfs.btrfs` on a disk that already has a Btrfs filesystem you want to keep — it will destroy all data.
- Do not remove a device from a Btrfs filesystem if the remaining devices do not have enough space for the data.
- Do not use `btrfs check --repair` on a mounted filesystem — it requires the filesystem to be unmounted and can cause data loss if used incorrectly.
- Do not assume `df -h` gives accurate free space numbers on Btrfs — use `btrfs filesystem usage` instead.
- Do not skip the `btrfs balance` after adding a device — without balancing, new data is written to the new device but existing data remains on the old device only.
- Do not use Btrfs RAID profiles for production workloads on Azure — Azure managed disks already provide storage redundancy.
- Do not leave lab resources running after completion — delete the resource group to avoid unnecessary charges.

## Real-World Context

As SLES 16 adopts Btrfs with subvolumes as the default root filesystem layout, Azure support engineers will increasingly encounter VMs where traditional partition and LVM troubleshooting does not apply. Understanding Btrfs operations is essential for tasks such as recovering from snapshot-based rollbacks, diagnosing space usage issues on CoW filesystems, expanding data disks, and troubleshooting boot failures related to subvolume mounts. The snapshot capabilities also provide new options for system recovery that did not exist with XFS or ext4.

## Optional Advanced Exploration

- Explore Btrfs quota groups (`qgroup`) to set space limits on individual subvolumes.
- Configure transparent compression (`compress=zstd`) and measure the space savings.
- Use `snapper` (SLES snapshot management tool) to create and manage automated snapshots on the root filesystem.
- Experiment with Btrfs `send` and `receive` to replicate snapshots between filesystems or VMs.
- Test the `nofail` and `x-systemd.requires` fstab options with Btrfs subvolume mounts to understand boot behavior if a device is missing.
- Compare the performance of Btrfs with CoW enabled vs. disabled (`nodatacow`) for database workloads.
