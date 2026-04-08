# Storage besides LVM — Laboratories

## Title and Scenario

These hands-on laboratories accompany the Storage — LVM, Blobfuse, and Fileshares module. The labs cover four core areas of Linux storage management in Azure: Logical Volume Management (LVM) creation and resizing, Azure File Share mounting via SMB and NFS, BlobFuse2 container mounting, and filesystem mount options via `/etc/fstab`. Each lab builds on practical, real-world storage operations encountered when managing Linux virtual machines on Azure.

## Deployment

All lab deployments are consolidated in this section. Use the buttons below to deploy the required Azure infrastructure for each lab.

**Lab 1 — LVM (Scenarios 1–4):** Deploy one RHEL 9 VM with a 4 GB data disk for LVM exercises.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fstorage_besides_LVM%2FLabs%2FStorageLab1.json)

**Lab 2 — Azure File Share (NFS and SMB):** Deploy one VM with storage accounts for NFS and SMB file shares.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fstorage_besides_LVM%2FLabs%2FStorageLab2.json)

**Lab 3 — BlobFuse2:** Deploy one RHEL 8 VM with a blob storage account and container for BlobFuse exercises.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fstorage_besides_LVM%2FLabs%2FStorageLab3.json)

**Lab 4 — fstab options:** Uses the same VM from a previous lab. No additional deployment required.

## Skills Required

- Linux command-line proficiency (shell navigation, file operations, text editing with `vi`)
- Familiarity with disk partitioning using `fdisk`
- Basic understanding of filesystems (XFS, ext4)
- Experience connecting to Linux VMs via SSH
- Ability to navigate the Azure Portal (VMs, disks, storage accounts, file shares)

## Recommended Prerequisites

Completion of the Storage — LVM, Blobfuse, and Fileshares knowledge module or equivalent experience is recommended before starting these labs.

## Objectives

- Create LVM Disks using Raw and partitioned, Resize and extend the Logical Volume.
- Extend the existing disk and resize the LVM (in this case LVM is created on top of whole disk).
- Extend the existing disk and resize the LVM (in this case LVM is created on top of a partition).
- Extend the existing disk and resize the LVM by creating new partitions.
- Extend the LV by adding new disk to the VG.
- Create and configure an Azure File Share and mount it to a Linux VM using both NFS and SMB protocols.
- Understand the prerequisites required for mounting NFS and SMB file shares on Linux.
- Install and configure BlobFuse2 to mount a blob storage container.
- Verify the impact of `/etc/fstab` mount options (read-only, noexec, nosuid) on the operating system.

## Environment Overview

These labs use Azure RHEL virtual machines with attached data disks. Lab 1 deploys a RHEL 9 VM with a 4 GB data disk for LVM exercises. Lab 2 deploys its own RHEL 9 VM along with NFS and SMB storage accounts, private endpoints, and private DNS zones. Lab 3 deploys a RHEL 8 VM with a blob storage account and container. Lab 4 reuses a VM from a previous lab and does not require an additional deployment. All labs require SSH access and root privileges.

## Your Mission

Complete each lab sequentially. Work through the LVM scenarios to build confidence with physical volume, volume group, and logical volume operations including creation, extension, and filesystem resizing. Then configure cloud-native storage with Azure File Shares and BlobFuse2. Finally, explore how fstab mount options affect filesystem behavior and security.

## Lab 1:  LVM
### About this Lab 

- This course/module was created for Module Storage besides LVM.
- It will take approximately 45 minutes.
- This module introduces you to the tools for LVM.
- This Lab provides hands-on activities.
- After this course/module you will be able to:
    - Create LVM Disks using Raw and partitioned, Resize and extend the Logical Volume.
    - Extend the existing disk and resize the LVM (in this case LVM is created on top of whole disk).
    - Extend the existing disk and resize the LVM (in this case LVM is created by creating a partition on the disk).
    - Extend the existing disk and resize the LVM by creating new partitions.
    - Extend the LV by adding new disk to the VG.
 
### Scenario 1
#### Extending a disk and resizing the LVM (In this case LVM is created on top of whole disk)

Sometimes there is a need to use LVM as part of a VM for flexibility purposes in terms of mountpoints and filesystems.

In this lab, we are going to create data disks and build LVM volumes on top of them.

#### Deployment instructions

Deployment: See the [Deployment](#deployment) section.

- Connect to the VM and switch to root account using the following command:

   ```bash
   sudo -i
   ```

- Identify the data disk that is not in use on the OS. Use the following commands to discover attached disks:

  ```bash
  lsblk
  lsblk -f
  lsscsi
  ls -lR /dev/disk/azure
  ```

The output of these commands will be similar to the examples below.

With `lsblk`, look at the **TYPE** column for entries marked as "disk" — those are the disks attached to the VM. The **NAME** column shows the device names (e.g., sda, sdb, sdc) and the **SIZE** column shows the capacity. Look for the disk that has no **MOUNTPOINTS** — that is the empty data disk. The attached data disk is 4 GB, so you can also identify it by size.

The `lsscsi` command shows SCSI devices with their LUN numbers, which helps you correlate Azure disk names with Linux device names. The `lsblk -f` variant adds the **FSTYPE** column, showing the filesystem type on each partition — useful for identifying unformatted disks. The `ls -lR /dev/disk/azure` command shows the Azure-specific disk symlinks, including LUN mappings under `/dev/disk/azure/scsi1/`.

The disk we will modify following this example will be the disk named lun0. Use the block device `/dev/disk/azure/scsi1/lun0`. Remember that in Azure device names are not guaranteed to persist across reboots, so we could use these block devices.

- Create a physical volume with command:

   ```bash
   pvcreate <disk>
   ```

- Create a volume group with command:

   ```bash
   vgcreate <volume_group_name> <physical_volume>
   ```

- Create a logical volume assigning all the free space with command:

   ```bash
   lvcreate -l 100%FREE -n <logical_volume_name> <volume_group_name>
   ```

- Verify the changes with the following commands:

   ```bash
   pvs  # check on the physical volumes
   vgs  # check on the volume groups
   lvs  # check on the logical volumes
   ```

- Format the logical volume with xfs filesystem type, create an empty directory to mount the filesystem there, add the FSTAB entry, mount the volume and verify:

  ```bash
  mkfs.xfs <logical_volume_path>   # Format the logical volume
  mkdir <path_to_new_directory>    # Create an empty directory
  echo "<logical_volume_path>   <path_to_new_directory>   xfs defaults,nofail 0 0" >> /etc/fstab  # Add the entry in fstab file
  mount <path_to_new_directory>  # Mount the filesystem
  df -Th | grep <path_to_new_directory>  # Verify
  ```

- Create one file of size 1GB and verify the md5sum and note it down to check later after resize of disk, change the names if needed:

  ```bash
  dd if=/dev/zero of=/testdisk/myfile.dat bs=1024k count=1000
  md5sum /testdisk/myfile.dat
  ```

- Go to the Azure Portal, stop the VM named storagelab01 and resize the disk `LUN 0` from 4GB to 8GB.  Start the VM, once ready, connect to the VM, switch to root account and verify from OS level.

  You can verify using the below commands:

  ```bash
  pvdisplay /dev/disk/azure/scsi1/lun0
  df -Th | grep testdisk
  ```

- Resize the physical volume and verify the size:

  ```bash
  pvresize /dev/disk/azure/scsi1/lun0
  vgdisplay <volume_group>
  ```

- Extend the logical volume using 3GB and verify the size:

  ```bash
  lvextend -L +3G /dev/testvg/testlv
  lvs
  df -Th |grep test
  ```

 **Note:** The logical volume size has changed, but the filesystem size has not. We need to extend the filesystem using the `xfs_growfs` command for XFS filesystems and `resize2fs` for ext4 filesystems.

- Extend the filesystem using the below commands and verify the size and check the `myfile.dat` checksum for consistency:

  ```bash
  xfs_growfs /testdisk
  df -Th | grep test
  md5sum /testdisk/myfile.dat
  ```

### Scenario 2
#### Extending a disk and resizing the LVM (In this case LVM is created on a partition)

- Create and attach a new empty disk of 4GB to the `storagelab01` VM. Use `LUN 1` for this.
- Identify the disk from OS perspective:

  ```bash
  lsblk
  ls -lR /dev/disk/azure
  ```

- Create a partition using new disk following the below commands:

  ```bash
  fdisk <disk_path>  # You can use /dev/disk/azure/scsi1/lun1
  n # Create a new partition
  p # Hit "enter" instead of writing letter "p" this value will takes up the default value, which is primary partition.
  1  # Partition number, default will be 1, enter or choose the number of your partition.
  2048 # First sector of partition, default will select the next available cylinder on the drive, hit on enter o type the first available number.
  8388607 # Last sector, add certain amount of sectors or add certain amount of space, enter will select last sector (using entire disk).
  p  # Print partition table.
  w  # Write table to disk and exist. 
  ```

- Run the **partprobe** command to inform the OS of partition table changes:

  ```bash
  partprobe
  ```

- Change partition ID/Type for LVM:

  ```bash
  fdisk /dev/disk/azure/scsi1/lun1
  t  # Change partition ID, it will chose the one existing by default
     # Select L to list all codes.
  8e # Linux LVM type code.
  w  # Write table to disk and exist.
  fdisk -l /dev/disk/azure/scsi1/lun1 # Verify the change
  ```

- Create a second physical volume, volume group and logical volume and verify:

  ```bash
  pvcreate <path_to_partition>
  vgcreate <volume_group_name> <path_to_partition>
  lvcreate -l 100%FREE -n <logical_volume_name> <volume_group_name>
  pvs | grep <disk_name>
  vgs | grep <volume_group_name>
  lvs | grep <logical_volume_name>
  ```

 - Format the logical volume using xfs filesystem type and add the fstab entry, mount and verify:

   ```bash
   mkfs.xfs <logical_volume_path>
   mkdir <new_directory_path>
   echo "<logical_volume_path>  <mount_directory_path> xfs defaults,nofail 0 0" >> /etc/fstab
   mount -a
   df -Th | grep <new_directory_name>
   ```

 
- Create a file with a size of 1GB, verify the md5sum and write it down for later comparison after resize of disk.

  ```bash
  dd if=/dev/zero of=<file_path_name> bs=1024k count=1000
  md5sum <file_path_name>
  ```

- Extend the `LUN 1`. Stop the VM and change it from 4GB to 8 GB. Start the VM and verify from OS level. 

  ```bash
  lsblk
  lsblk -f
  lsscsi
  ls -lR /dev/disk/azure
  ```

  **Note:** We can change the size only at the disk level, not at the partition level. We can delete and recreate the partition again. Data on the disk will not be lost; only the partition mapping will be deleted and created again.

- Delete and create a new partition from the resized disk:

  ```bash
  fdisk <disk_path>
  p # Verify initial information of partitions.
  d # Delete the current partition.
  n # Create a new partition.
  p # Select primary, you can just hit on enter, primary is the default.
  1 # Select the partition number or just hit enter to select the default one.
  2048  # Using the information from the previous print, locate the previous start sector. This need to match previous value
  16777215 # Last sector, add certain amount of sectors or add certain amount of space, enter will select last sector (using entire disk).
  N # For the question "Do you want to remove the signature? [Y]es/[N]o", select no by typing an uppercase N.
  p # Print partition table. Validate the partition size ow is 8GB.
  w # Write table to disk and exist. 
  ```

- Resize the physical volume and verify. Extend the logical volume by 3GB and then resize the filesystem. As last step, check consistency of the file using md5sum:

   ```bash
   pvresize /dev/disk/azure/scsi1/lun1
   pvs
   vgs
   lvextend -L +3GB <logical_volume>
   lvs
   xfs_growfs <filesystem>
   df -h <filesystem>
   md5sum <file_path>
   ```

### Scenario 3
#### Extend the existing disk and resize the LVM by creating new partitions

- Stop the VM and resize the `LUN 1` disk attached to it from 8GB to 16GB. Start the VM, connect to it, switch to root account and check on it.
- Identify the disk and create the second partition:

  ```bash
  lsblk
  fdisk /dev/disk/azure/scsi1/lun1
  n # Create a new partition
  p # Hit "enter" instead of writing letter "p" this value will takes up the default value, which is primary partition.
  2 # Partition number, default will be 2, enter or choose the number of your partition.
  16777216 # First sector of partition, default will select the next available cylinder on the drive, hit on enter o type the first available number.
  33554431 # Last sector, add certain amount of sectors or add certain amount of space, enter will select last sector (using entire disk).
  p  # Print partition table.
  w  # Write table to disk and exist. 
  ```

- Run `partprobe` to scan the partition tables:

  ```bash
  partprobe
  fdisk -l <disk_path>
  ```

- Toggle the partition ID for LVM:

  ```bash
  fdisk /dev/disk/azure/scsi1/lun1
  t # Change partition ID, it will chose the one existing by default
    # Select L to list all codes.
  2 # Partition number
  8e # Linux LVM type code.
  w  # Write table to disk and exist.
  fdisk -l /dev/disk/azure/scsi1/lun1 # Verify the change
  ```

- Create a physical volume on the second partition. Extend the volume group and resize the logical volume. Grow the filesystem and check with following commands:

  ```bash
  pvcreate <second_partition_path>
  vgextend <volume_group> <second_partition_path>
  pvs #To check on the physical volumes
  vgs #To check on volume groups
  lvextend -L +8G <logical_volume>
  lvs #To check on logical volume
  xfs_growfs <filesystem>
  df -Th | grep <filesystem>
  ```
   

### Scenario 4
#### Extend the logical volume by adding new disk to the volume group

- Create and attach an empty disk of 4GB to the VM. Use `LUN 2`. Identify it from operating system perspective with command:

  ```bash
  lsblk
  lsblk -f
  lsscsi
  ls -lR /dev/disk/azure
  ```

- Create a new partition using entire disk as described previously.

- Create a new physical volume, extend the volume group and check:

  ```bash
  pvcreate <partition_path>
  vgextend <volume_group_name> <partition_path>
  pvs
  vgs
  ```

- Extend the logical volume, resize the filesystem and check:

  ```bash
  lvextend -L +4G <logical_volume>
  xfs_growfs <filesystem>
  df -Th |grep <filesystem>
  ```

  **Note:**  Remember to delete all the resources once you complete the Laboratories. 

 ### Your Goal 

 At the end of this lab, you should know how to: 

 - Resize an existing physical volume, volume group and logical volume.

   LVM offers great flexibility for performing hot resizes without the need to unmount anything. It is recommended you test more with LVM such as deleting volumes, adding a new disk, extending an existing volume group, or creating a new one, growing and shrinking, creating snapshots, etc.

### References

[Increasing the Size of an XFS File System](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/storage_administration_guide/xfsgrow)

[How to extend a logical volume and its filesystem online in Red Hat Enterprise Linux?](https://access.redhat.com/solutions/24770)

[Linux LVM Cheat Sheet / Quick Reference](https://unixutils.com/lvm-cheat-sheet-quick-reference/)

## Lab 2: Azure File Share (NFS and SMB)

### About this Lab

- This course/module was created for Storage Lab Azure file share
- It will take approximately 30 minutes
- This module introduces you to the tools needed to mount Azure File Shares on Linux using both NFS and SMB protocols.
- This Lab provides hands-on activities.
- After this course/module you will be able to:
      - Install required client packages for NFS and SMB on Linux.
      - Create and mount an NFS file share using the standard NFS client.
      - Create and configure an SMB file share and mount it to the VM.
      - Validate mount behavior and persistence via `/etc/fstab`.

### Scenario
This laboratory covers mounting Azure File Shares using both NFS and SMB protocols on a Linux VM. The ARM template deploys a dedicated RHEL 9 VM along with NFS and SMB storage accounts, private endpoints, and private DNS zones.

### Instructions

#### Part 1: Install Required Client Packages

- Using your preferred SSH client, connect to the Azure VM and switch to root account:

  ```bash
  sudo -i
  ```

- Install the required NFS and SMB client packages for your distribution:

  On **RHEL**:
  ```bash
  dnf install -y nfs-utils cifs-utils
  ```

  On **Ubuntu**:
  ```bash
  apt-get update
  apt-get install -y nfs-common cifs-utils
  ```

  On **SLES**:
  ```bash
  zypper refresh
  zypper install -y nfs-client cifs-utils
  ```

  For additional NFS prerequisite details, follow the instructions provided in the following link:

  [Mount NFS Azure file share on Linux Prerequisites](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-how-to-mount-nfs-shares#prerequisites)

  For additional SMB prerequisite details, follow the instructions provided in the following link, if necessary, change the tab so you use instructions for the correct operating system:

  [Mount SMB Azure file share on Linux Prerequisites](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-use-files-linux?tabs=RHEL%2Csmb311#prerequisites)

- Create local mount points:

  ```bash
  mkdir -p /media/nfs /media/smb
  ```

#### Part 2: Mount NFS File Share

The ARM template deployed a Premium FileStorage account with an NFS-enabled file share named `nfsshare`, a private endpoint, and the corresponding private DNS zone.

- Identify the NFS storage account. In the Azure Portal, go to your resource group and look for the storage account whose name starts with **nfs**. Open the storage account, navigate to **Data storage > File shares**, and confirm the `nfsshare` share exists. Note the storage account name.

- On the VM, verify DNS resolution to the NFS storage account resolves to the private endpoint IP:

  ```bash
  nslookup <nfsStorageAccountName>.file.core.windows.net
  ```

- Mount the NFS share using the standard NFS client:

  ```bash
  mount -t nfs -o vers=4,minorversion=1,sec=sys <nfsStorageAccountName>.file.core.windows.net:/<nfsStorageAccountName>/nfsshare /media/nfs
  ```

- Validate the NFS mount:

  ```bash
  mount | grep /media/nfs
  df -h | grep /media/nfs
  touch /media/nfs/test-native-$(hostname)
  ls -l /media/nfs
  ```

- Optional `/etc/fstab` entry pattern for NFS persistence:

  ```bash
  <nfsStorageAccountName>.file.core.windows.net:/<nfsStorageAccountName>/nfsshare /media/nfs nfs vers=4,minorversion=1,sec=sys,_netdev,nofail 0 0
  ```

#### Part 3: SMB Prerequisites

Before mounting SMB on the Linux VM, ensure all prerequisites are met:

- Protocol and OS support
   - SMB 3.x capable clients.
   - `cifs-utils` installed (already done in Part 1).
- Network access to SMB endpoint
    - TCP **445** must be allowed from VM to storage endpoint.
    - Name resolution must resolve the storage account SMB FQDN.
    - If private endpoint is used, private DNS zone/link must be correct.
- Authentication method
    - One of the following must be configured:
      - Storage account key (lab/simple scenario)
      - Microsoft Entra ID / AD DS / Entra DS (enterprise scenario)

    > **Warning:** Even though `allowSharedKeyAccess` is set in the ARM template, a subscription policy can still flip this setting. Check **Allow storage account key access** in the storage account configuration. If you enable it and after clicking refresh it goes back to disabled, you were most likely blocked by policy.
- Secure transport and policy alignment
    - SMB encryption/signing requirements must match account policy.
    - Any firewall or NSG controls must explicitly allow storage traffic.
- Share permissions + NTFS/ACL alignment
    - Share-level permissions are not enough alone in identity-based setups.
    - File/directory ACLs must allow the target user/group.

#### Part 4: Mount SMB File Share

The ARM template deployed a Standard StorageV2 account with an SMB-enabled file share named `smbshare`, a private endpoint, and the corresponding private DNS zone.

- Identify the SMB storage account. In the Azure Portal, go to your resource group and look for the storage account whose name starts with **smb**. Open the storage account, navigate to **Data storage > File shares**, and confirm the `smbshare` share exists. Note the storage account name.

- Retrieve the storage account key. In the Azure Portal, go to the SMB storage account, select **Security + networking > Access keys**, and click **Show** to copy key1.

  Alternatively, use the Azure CLI:

  ```bash
  az storage account keys list -g <resource_group> -n <smbStorageAccountName> --query '[0] .value' -o tsv
  ```

- On the VM, verify DNS resolution to the SMB storage account resolves to the private endpoint IP:

  ```bash
  nslookup <smbStorageAccountName>.file.core.windows.net
  ```

- Create a credential file for the SMB mount:

  ```bash
  mkdir -p /etc/smbcredentials
  cat > /etc/smbcredentials/<smbStorageAccountName>.cred <<EOF
  username=<smbStorageAccountName>
  password=<STORAGE_ACCOUNT_KEY>
  EOF
  chmod 600 /etc/smbcredentials/<smbStorageAccountName>.cred
  ```

- Mount the SMB share:

  ```bash
  mount -t cifs //<smbStorageAccountName>.file.core.windows.net/smbshare /media/smb \
      -o vers=3.1.1,credentials=/etc/smbcredentials/<smbStorageAccountName>.cred,serverino,dir_mode=0770,file_mode=0660,mfsymlinks,actimeo=30
  ```

- Check you have the file share mounted and it's added to the fstab with the following commands:

  ```bash
  df -h | grep /media/smb
  mount | grep /media/smb
  ```

- Create a test file and verify:

  ```bash
  touch /media/smb/test-smb-$(hostname)
  ls -l /media/smb
  ```

- Optional `/etc/fstab` entry pattern for SMB persistence:

  ```bash
  //<smbStorageAccountName>.file.core.windows.net/smbshare /media/smb cifs nofail,_netdev,vers=3.1.1,credentials=/etc/smbcredentials/<smbStorageAccountName>.cred,serverino,dir_mode=0770,file_mode=0660,mfsymlinks,actimeo=30 0 0
  ```

### Your Goal
At the end of this lab, you should know how to install NFS and SMB client packages, create Azure File Shares using both NFS and SMB protocols, mount them on a Linux VM, validate access, and understand the prerequisites required for each protocol.

### References

[What is Azure Files?](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-introduction)

[Mount NFS Azure file share on Linux](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-how-to-mount-nfs-shares)

[Mount SMB Azure file share on Linux](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-use-files-linux)

## Lab 3: BlobFuse2

### About this Lab

- This course/module was created for Storage Lab 3 BlobFuse
- It will take approximately **30** minutes
- This module introduces you to the use of blobfuse2.
- This Lab provides hands-on activities.
- After this course/module you will be able to:
      - Install and configure blobfuse2 to mount the container from blob storage using configuration file.

### Deployment instructions

Deployment: See the [Deployment](#deployment) section.

The ARM template creates a blob storage account with a container named `blobfuse`, a private endpoint, and the corresponding private DNS zone.

- Once the VM is created, connect to it using SSH protocol, switch to root account and check the Linux version by running the following commands:

  ```bash
  sudo -i
  cat /etc/*-release
  ```

- Download BlobFuse2 using Microsoft software repositories for Linux:

  ```bash
  rpm -Uvh https://packages.microsoft.com/config/rhel/8/packages-microsoft-prod.rpm
  ```

- Install the blobfuse2 package:

   ```bash
   dnf install blobfuse2 -y
   ```

- Create an empty directory to be used as mount point for the blob storage container:

   ```bash
   mkdir <directory_path>
   ```

- Identify the blob storage account. In the Azure Portal, go to your resource group and look for the storage account (the only one in the resource group). Open the storage account, navigate to **Data storage > Containers**, and confirm the `blobfuse` container exists. Note the storage account name.

- Retrieve the storage account key.

  **Using the Azure Portal:** Go to the storage account, select **Security + networking > Access keys**, and click **Show** to reveal key1. Copy the key value.

  **Using the Azure CLI:**

  ```bash
  az storage account keys list -g <resource_group> -n <blobfuseStorageAccountName> --query '[0].value' -o tsv
  ```

  > **Warning:** Even though `allowSharedKeyAccess` is set in the ARM template, a subscription policy can still flip this setting. Check **Allow storage account key access** in the storage account configuration. If you enable it and after clicking refresh it goes back to disabled, you were most likely blocked by policy.

- On the Azure VM, create a YAML configuration file for mounting blobfuse. In this example we create it under the `/etc` directory. The file should contain the following information:

  ```yaml
  allow-other: true
   
  logging:
    type: syslog
    level: log_debug
   
  components:
    - libfuse
    - file_cache
    - attr_cache
    - azstorage
   
  libfuse:
    attribute-expiration-sec: 120
    entry-expiration-sec: 120
    negative-entry-expiration-sec: 240
   
  file_cache:
    path: <temporary_directory_that_will_be_used_by_blobfuse>
    timeout-sec: 120
    max-size-mb: 4096
   
  attr_cache:
    timeout-sec: 7200
   
  azstorage:
    type: block
    account-name: <your_storage_account_name>
    account-key: '<your_storage_account_key>'
    endpoint: https://<your_storage_account_name>.blob.core.windows.net
    mode: key
    container: <container_name>
  ```

- Mount the blobfuse in the directory created with command:

  ```bash
  blobfuse2 mount <directory_name> --config-file=<configuration_file>
  ```

- Validate the mount:

  ```bash
  df -h | grep <directory_name>
  blobfuse2 mount list
  touch <directory_name>/test-blobfuse-$(hostname)
  ls -l <directory_name>
  ```

### References

[What is BlobFuse?](https://learn.microsoft.com/en-us/azure/storage/blobs/blobfuse2-what-is)

[How to mount an Azure Blob Storage container on Linux with BlobFuse2](https://learn.microsoft.com/en-us/azure/storage/blobs/blobfuse2-how-to-deploy?tabs=RHEL)

[BaseConfig.yaml](https://github.com/Azure/azure-storage-fuse/blob/main/setup/baseConfig.yaml)


## Lab 4: fstab options

### About this Lab

- This course/module was created to demonstrate `fstab` file options
- It will take approximately 30 minutes
- This Lab provides hands-on activities.
- After this course/module you will be able to:
      - Verify the mount options in the `/etc/fstab` file and understand how they impact the operating system.

### Instructions

- For this laboratory use the VM from the previous lab. Confirm there is a data disk already present `LUN 0`

  ```bash
  sudo -i
  lsblk
  ls -lR /dev/disk/azure
  ```

- Create a partition on the attached disk `/dev/disk/azure/scsi1/lun0` using all available space:

  ```bash
  fdisk /dev/disk/azure/scsi1/lun0
  n # Create a new partition
  p # Hit "enter" instead of writing letter "p" this value will takes up the default value, which is primary partition.
  1  # Partition number, default will be 1, enter or choose the number of your partition.
  2048 # First sector of partition, default will select the next available cylinder on the drive, hit on enter o type the first available number.
  8388607 # Last sector, add certain amount of sectors or add certain amount of space, enter will select last sector (using entire disk).
  p  # Print partition table.
  w  # Write table to disk and exist. 
  ```

- Run `partprobe` to scan the partition tables, and then verify the partitions:

  ```bash
  partprobe
  cat /proc/partitions
  fdisk -l
  lsblk | grep <partition_name>
  ```

- Format created partition with XFS filesystem type.

  ```bash
  mkfs.xfs <partition_path>
  ```

- Create an empty directory to be used as mount point:

  ```bash
  mkdir <new_directory_path>
  ```

- Get the UUID of the partition, add the entry in fstab file using the read-only(ro) option, mount it and check:

  ```bash
  blkid <partition_path>
  echo "UUID=<UUID>   <mount_directory_path>   xfs   defaults,ro  0 0" >> /etc/fstab
  mount -a
  df -Th | grep <mount_point>
  ```

- Go to the mounted directory and create an empty file:

  ```bash
  cd <mounted_directory>
  touch <file_name>
  ```

  **Note:** filesystem is read-only. Default behavior is read write 

- Try `noexec` option, this will not allow the execution in the filesystem. Proceed to umount the filesystem, modify the previous entry in `/etc/fstab` change the read-only `ro` option by `noexec`, and mount the filesystem again:

  ```bash
  cd / # Step out of the mount point
  umount <directory>
  vi /etc/fstab   # Replace the ro by noexec
  cat /etc/fstab | grep <directory> # Verify the changes
  mount -a
  df -Th | grep <directory>
  ```

- Create a new file and add the contents below following these commands, replace the directory name below:

  ```bash
  cat <<EOF > <directory>/file1.sh
  #!/bin/bash
  mkdir <directory>/testdir
  EOF
  ```

- Add execution permission to the file and try to execute it:

  ```bash
  chmod +x <directory>/file1.sh
  <directory>/file1.sh
  ```

- Umount the filesystem, remove the `noexec` option, mount the filesystem back and try to execute the script again:

  ```bash
  umount <directory>
  vi /etc/fstab # Remove noexec option and save the file
  cat /etc/fstab | grep <directory> # Verify the changes
  mount -a
  cd <directory>
  ./file1.sh
  ```

**NOSUID Option**

- Filesystems mounted with the `nosuid` option do not allow set-user-identifier (SUID) or set-group-identifier (SGID) bits to take effect. In other words, even if an executable file has the SUID bit set, it will not be executed as the file's owner. Instead, it will execute with the privileges of the user who is executing it.

  Example files:

  ```bash
  /bin/passwd
  /bin/su
  /usr/bin/mount 
  ```

#### Your goal
At the end of this lab, you should know how to:
- Use mount point options in the `fstab` file and understand how they affect the operating system.

#### References 
[Filesystem table (/etc/fstab) Cheatsheet](https://dcjtech.info/wp-content/uploads/2016/06/Filesystem-Table-etc-fstab-Cheatsheet.pdf)

---

## Analytical Guidance

- Follow each lab sequentially; later scenarios build on resources created in earlier ones.
- Before executing any LVM operation, verify the current state with `pvs`, `vgs`, and `lvs` to establish a baseline.
- Always confirm disk identity using `lsblk` before performing destructive operations — Azure does not guarantee persistent device names across reboots.
- When resizing partitions (Scenario 2, step 10), pay close attention to the starting sector — it must match the original to preserve data.
- After every resize operation, verify filesystem integrity using `md5sum` on test files to confirm data consistency.
- In Lab 4, observe the behavioral differences between mount options (`ro`, `noexec`, `nosuid`) and understand why each is relevant for security hardening.
- Compare the BlobFuse2 YAML configuration parameters with real-world requirements to understand caching and performance trade-offs.

## Validation Criteria

After completing all labs, you should be able to:

- Create LVM physical volumes from both raw disks and partitions.
- Create volume groups and logical volumes, format them, mount them, and configure persistent mounts via `/etc/fstab`.
- Resize Azure data disks and propagate the size change through the LVM stack (PV → VG → LV → filesystem).
- Extend a volume group by adding a new physical volume from a second partition or a new disk.
- Verify data integrity after resize operations using `md5sum`.
- Create an Azure File Share and mount it on a Linux VM using SMB/CIFS.
- Install and configure BlobFuse2 to mount a blob container using a YAML configuration file.
- Demonstrate the effects of `ro`, `noexec`, and `nosuid` mount options on filesystem behavior.

## Documentation Expectations

- Record the exact LVM commands and their output for each scenario.
- Note the `md5sum` values before and after disk resize operations to document data integrity verification.
- Document the Azure portal steps taken for disk resize and file share configuration.
- Capture the BlobFuse2 YAML configuration used and any troubleshooting steps if the mount fails.
- Record the behavior observed with each fstab mount option (ro, noexec, nosuid) including exact error messages.

## What Not To Do

- Do not skip the `partprobe` command after partition table changes — the kernel may not detect the new layout without it.
- Do not assume device names persist across VM stop/start cycles in Azure — always re-identify disks with `lsblk`.
- Do not answer "Yes" when asked to remove the LVM signature during partition recreation (Scenario 2, step 10) — this destroys the LVM metadata.
- Do not forget to grow the filesystem after extending the logical volume — `lvextend` alone does not resize the filesystem.
- Do not use `resize2fs` on XFS filesystems or `xfs_growfs` on ext4 filesystems — use the correct tool for the filesystem type.
- Do not store storage account keys in plain text in production — the YAML configuration in Lab 3 is for lab purposes only.
- Do not leave lab resources running after completion — delete the resource group to avoid unnecessary charges.

## Real-World Context

These labs simulate the most common storage operations encountered in Azure Linux support: extending data disks, resizing LVM structures, configuring cloud file shares, and troubleshooting mount issues. In production, support engineers frequently encounter VMs that fail to boot due to incorrect fstab entries, filesystems that were not grown after LVM extension, or BlobFuse configurations that cause mount failures. The skills practiced here map directly to real case resolution workflows.

## Optional Advanced Exploration

- Experiment with shrinking an ext4-based logical volume (note: XFS does not support shrinking).
- Create an LVM snapshot of a logical volume and test restoring from it.
- Configure BlobFuse2 with managed identity authentication instead of storage account keys.
- Mount an NFS Azure File Share and compare behavior with the SMB mount from Lab 2.
- Test the `nofail` mount option by detaching a data disk and rebooting to observe boot behavior with and without it.
- Explore the `x-systemd.requires` and `_netdev` mount options for network-dependent filesystems in fstab.
