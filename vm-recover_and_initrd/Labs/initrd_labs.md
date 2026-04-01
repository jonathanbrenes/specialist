# Greater Insight Initrd Lab — Initrd/Initramfs Recovery for Non-Boot Scenarios

## Title and Scenario

This module provides three hands-on labs simulating real-world Azure Linux VM non-boot scenarios caused by initrd/initramfs issues. You will troubleshoot and recover VMs affected by:

- Corrupt initrd/initramfs image file preventing kernel boot
- Missing hv_storvsc (LIS) driver in initrd/initramfs causing storage initialization failure
- Missing all LIS drivers (hv_vmbus, hv_netvsc, hv_storvsc) in initrd/initramfs causing complete Hyper-V integration failure

Each lab deploys a pre-configured Azure VM with an injected issue. You must diagnose the problem using Boot Diagnostics and Serial Console, apply the correct recovery technique via a chroot repair environment, and validate that the VM returns to a healthy state.

- This Lab was created for Greater Insight Initrd.
- It will take approximately 60 minutes
- This lab has three activities:

> 1. Fix VM noboot issue caused by corrupt initrd/initramfs.
> 2. Fix VM noboot issue caused by missing hv_storvsc of LIS driver in initrd/initramfs.
> 3. Fix VM noboot issue caused by missing LIS driver in initrd/initramfs.

---

## Deployment

All lab environments are deployed using the buttons below. Each button deploys a pre-configured RHEL 9 VM with an injected initrd issue for you to investigate.

**Lab 1 — RHEL 9 VM (corrupt initrd/initramfs):**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://labboxprod.azurewebsites.net/api/labbox?url=https://dev.azure.com/LinuxNinjas/Azure%20Linux%20Academy%20-%20CSS/_git/AzureLinuxAcademy?path=/Azure%20Linux%20Academy/Azure_Linux_Specialist_Self_Paced/vm-recover_and_initrd/Labs/Lab01.json)

**Lab 2 — RHEL 9 VM (missing hv_storvsc driver in initrd):**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://labboxprod.azurewebsites.net/api/labbox?url=https://dev.azure.com/LinuxNinjas/Azure%20Linux%20Academy%20-%20CSS/_git/AzureLinuxAcademy?path=/Azure%20Linux%20Academy/Azure_Linux_Specialist_Self_Paced/vm-recover_and_initrd/Labs/Lab02.json)

**Lab 3 — RHEL 9 VM (missing all LIS drivers in initrd):**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://labboxprod.azurewebsites.net/api/labbox?url=https://dev.azure.com/LinuxNinjas/Azure%20Linux%20Academy%20-%20CSS/_git/AzureLinuxAcademy?path=/Azure%20Linux%20Academy/Azure_Linux_Specialist_Self_Paced/vm-recover_and_initrd/Labs/Lab03.json)

---

## Skills Required

- Linux command-line fundamentals (file editing, process management, log inspection)
- Understanding of Linux boot process, initrd/initramfs, and kernel module loading
- Familiarity with `dracut` and its configuration (`/etc/dracut.conf`)
- Understanding of Linux Integration Services (LIS) drivers for Hyper-V (hv_vmbus, hv_netvsc, hv_storvsc)
- Experience with chroot environments for offline system repair
- Basic understanding of Azure VM management (portal navigation, Serial Console, Boot Diagnostics)
- Working knowledge of Azure CLI and the `az vm repair` extension

---

## Recommended Prerequisites

- Completion of the Azure Linux Foundation modules (OS Fundamentals I & II)
- Access to an Azure subscription with permission to create VMs and resource groups
- Familiarity with at least one text editor (`vi`, `vim`, `nano`)
- WSL2, Cloud Shell, or another SSH client installed and configured
- Basic understanding of the Azure VM Repair extension workflow

---

## Objectives

After completing all three labs, you will be able to:

- Identify non-boot scenarios caused by initrd/initramfs corruption or misconfiguration through Boot Diagnostics and Serial Console logs
- Create a Repair VM using the `az vm repair` extension and attach the faulty OS disk as a data disk
- Create and connect to a chroot environment on a rescue VM
- Back up and rebuild initrd/initramfs images using `dracut`
- Identify and correct missing LIS driver configurations in `/etc/dracut.conf`
- Understand the role of `hv_vmbus`, `hv_netvsc`, and `hv_storvsc` drivers in Azure Linux VMs
- Swap the repaired OS disk back to the original VM and verify successful boot
- Use ALAR scripts as an alternative recovery method

---

## Environment Overview

Each lab deploys a single Azure VM with the following characteristics:

| Lab | VM Name | Distribution | VM Size | NICs | Boot Diagnostics |
|-----|---------|-------------|---------|------|------------------|
| 1 | lab01initrd | RHEL 9 (Gen2) | Standard_B2s | 1 | Enabled |
| 2 | lab02initrd | RHEL 9 (Gen2) | Standard_B2s | 1 | Enabled |
| 3 | lab03initrd | RHEL 9 (Gen2) | Standard_B2s | 1 | Enabled |

All VMs are deployed with:

- A single NIC with a static private IP (10.1.0.10) and a Standard public IP
- An NSG allowing inbound SSH (port 22) from the AzureCloud service tag
- Managed boot diagnostics enabled (required for Serial Console access)
- No data disks
- Password-based authentication

---

## Your Mission

### Lab 1: Fix VM no boot issue caused by corrupt initrd/initramfs

#### Instructions

1. Deploy a broken Red Hat VM.

    Deployment: See the Deployment section.

2. Check on the VM Serial Console log and Boot Diagnostics screenshot and Serial log to confirm the no-boot status.  VM is in a non-boot scenario due to corrupt initramfs/initrd for current kernel. You'll find a screen with a kernel panic, example:
    ![initramfs file_corrupted](https://dev.azure.com/LinuxNinjas/aa969835-d5b5-4c66-a74c-74d1f9d57eed/_apis/git/repositories/16b54735-533f-46a2-a894-32099518c4eb/items?path=/Azure%20Linux%20Academy/Azure_Linux_Specialist_Self_Paced/vm-recover_and_initrd/images/initramfs-lab1-error.png&versionDescriptor%5BversionOptions%5D=0&versionDescriptor%5BversionType%5D=0&versionDescriptor%5Bversion%5D=master&resolveLfs=true&%24format=octetStream&api-version=5.0)

3. Create a Repair VM and attach an OS disk copy of damage VM as data disk.
4. Create and connect to a chroot environment following the public documentation: [Chroot environment in a Linux rescue VM](https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/chroot-environment-linux)
5. Take a backup of current iniramfs/initrd file using command **cp**
6. Rebuild initramfs, remember to keep the appropiate path while executing this command:

   `# dracut -f -v initramfs-<kernel=version>.img <kernel-version>`

7. Exit chroot environmet and umount OS disk copy, then proceed to swap the OS disk in the failing VM.
8. Start the VM and verify the VM is booting as expected.

**NOTE:**  Another way to fix this scenario is using ALAR scripts you can find more information here: [Use Azure Linux Auto Repair (ALAR) to fix a Linux VM](https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/repair-linux-vm-using-alar)

---

### Lab 2: Fix VM noboot issue caused by missing hv_storvsc of LIS driver in initrd/initramfs

#### Scenario

On this Lab the hv_storvsc driver has been removed from the Initrd configuration.

Your task is to set a repair VM to fix the situation.

Once you've added the missing driver into the Initrd configuration file, make the necessary configuration changes to ensure the VM boots up properly.

#### Symptom

![initramfs driver_missing](https://dev.azure.com/LinuxNinjas/aa969835-d5b5-4c66-a74c-74d1f9d57eed/_apis/git/repositories/16b54735-533f-46a2-a894-32099518c4eb/items?path=/Azure%20Linux%20Academy/Azure_Linux_Specialist_Self_Paced/vm-recover_and_initrd/images/initramfs-lab2-error.png&versionDescriptor%5BversionOptions%5D=0&versionDescriptor%5BversionType%5D=0&versionDescriptor%5Bversion%5D=master&resolveLfs=true&%24format=octetStream&api-version=5.0)

#### Instructions

1.  Deploy a broken Red Hat VM.

    Deployment: See the Deployment section.

2. Check in Serial Console log and Boot Diagnostics that VM is in a non-boot scenario and check on the error.
3. Create a Repair VM environment and attach a OS disk copy to this environment as data disk.
4. Create and connect to a chroot environment following the public documentation: [Chroot environment in a Linux rescue VM](https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/chroot-environment-linux)
5. Create a backup of the problematic initramfs using command *cp*
6. Modify configuration file and comment out the line that says "omit_drivers+=" hv_storvsc " and add the add_drivers line like below.  Then, proceed to rebuild the initrd file for the current kernel using the command below (*Remember to include the correct path on the command*):

          #vi /etc/dracut.conf
          add_drivers+=" hv_storvsc "
          #dracut -f -v <initramfsversion> <kernelversion>

7. Exit chroot and unmount the OS disk copy from the troubleshooting VM, after you've done that, reassemble the original VM by switching the OS disk.

8. The VM should be now able to boot after Initrd configuration gets changed.

---

### Lab 3: Fix VM non-boot issue caused by missing of all LIS driver in Initrd

#### Symptom

![initramfs driver_missing](https://dev.azure.com/LinuxNinjas/aa969835-d5b5-4c66-a74c-74d1f9d57eed/_apis/git/repositories/16b54735-533f-46a2-a894-32099518c4eb/items?path=/Azure%20Linux%20Academy/Azure_Linux_Specialist_Self_Paced/vm-recover_and_initrd/images/initramfs-lab3-error.png&versionDescriptor%5BversionOptions%5D=0&versionDescriptor%5BversionType%5D=0&versionDescriptor%5Bversion%5D=master&resolveLfs=true&%24format=octetStream&api-version=5.0)

#### Instructions

1. Deploy one broken Red Hat VM.

    Deployment: See the Deployment section.

2. Check in Serial Console log and Boot Diagnostics that VM is in a non-boot scenario and check on the error.
3. Create a Repair VM environment and attach a OS disk copy to this environment as data disk.
4. Create and connect to a chroot environment following the public documentation: [Chroot environment in a Linux rescue VM](https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/chroot-environment-linux)
5. Modify configuration file and delete or comment out the line that says "omit_drivers+=" hv_vmbus hv_netvsc hv_storvsc " and add the line add_drivers as explained below.  Then, proceed to rebuild the initrd file for the current kernel using the command below (*Remember to include the correct path on the command*):

          #vi /etc/dracut.conf
          add_drivers+=" hv_vmbus hv_netvsc hv_storvsc "
          #dracut -f -v <initramfsversion> <kernelversion>

6. Exit chroot and unmount the OS disk copy from the troubleshooting VM, after you've done that, reassemble the original VM by switching the OS disk.

8. The VM should be now able to boot after Initrd configuration gets changed.

---

## Analytical Guidance

When troubleshooting initrd/initramfs non-boot scenarios, follow a structured approach:

- Always start with Boot Diagnostics: review the Serial Console log and screenshot to identify the type of kernel panic or boot failure.
- Distinguish between a fully corrupt initramfs image (Lab 1 pattern — requires full rebuild) and a misconfigured initramfs with missing drivers (Labs 2/3 pattern — requires configuration correction before rebuild).
- For driver-related boot failures, identify which specific drivers are missing from the error output. The kernel panic message and module loading failures in the Serial Console log provide the key diagnostic information.
- Understand the dependency chain: `hv_vmbus` is the parent bus driver, `hv_netvsc` provides network, and `hv_storvsc` provides storage. Missing `hv_storvsc` alone prevents disk access; missing all three prevents all Hyper-V integration.
- Always create a backup of the existing initramfs before making changes — even a corrupt file may contain useful diagnostic information.
- When working in a chroot environment, verify you are operating on the correct disk and partition before modifying configuration files.

---

## Validation Criteria

Each lab is considered complete when:

- **Lab 1:** The VM boots successfully without kernel panic, and SSH access is restored. The initramfs file has been rebuilt for the current kernel.
- **Lab 2:** The VM boots successfully, the `hv_storvsc` driver is loaded and functional, and SSH access is restored. The `/etc/dracut.conf` no longer contains `omit_drivers` entries excluding `hv_storvsc`.
- **Lab 3:** The VM boots successfully, all LIS drivers (`hv_vmbus`, `hv_netvsc`, `hv_storvsc`) are loaded and functional, and SSH access is restored.

---

## Documentation Expectations

For each lab, document:

- The exact kernel panic message or boot failure symptoms observed in Boot Diagnostics
- Which disk and partition you identified as the faulty OS disk on the repair VM
- The chroot steps you followed and any issues encountered during setup
- The specific changes made to `/etc/dracut.conf` (Labs 2 and 3)
- The exact `dracut` command used to rebuild the initramfs
- The verification steps performed to confirm the fix after OS disk swap

---

## What Not To Do

- Do not skip creating a backup of the existing initramfs file before rebuilding
- Do not attempt to rebuild initramfs without first entering a properly configured chroot environment
- Do not modify `/etc/dracut.conf` without understanding the difference between `add_drivers`, `omit_drivers`, and their syntax requirements
- Do not forget to unmount all filesystems and exit chroot before swapping the OS disk
- Do not assume the repair VM disk layout matches the faulty VM — always identify the correct data disk and partition
- Do not skip Boot Diagnostics review to understand the specific failure mode before creating a repair VM

---

## Real-World Context

These labs simulate scenarios encountered regularly in Azure Linux support:

- **Initramfs corruption** can occur during kernel updates, storage failures, or incomplete patching cycles. A corrupt initramfs prevents the kernel from loading the initial root filesystem, resulting in an immediate kernel panic at boot.
- **Missing LIS drivers in initramfs** is a common issue when customers manually modify `/etc/dracut.conf` or when automation tools inadvertently alter the dracut configuration. Without `hv_storvsc`, the VM cannot access its virtual disk; without `hv_vmbus`, no Hyper-V paravirtualized devices function at all.
- The **chroot repair workflow** is the standard approach for offline initramfs repair: attach the OS disk to a rescue VM, mount the filesystem hierarchy, chroot into it, and rebuild the initramfs with the correct configuration.
- **ALAR (Azure Linux Auto Recovery)** scripts provide an automated alternative to manual chroot repair, particularly useful when the repair needs to be performed at scale or by operators who may not be familiar with the chroot workflow.
- The **Azure VM Repair extension** (`az vm repair`) automates the rescue VM creation and OS disk attachment, reducing the manual Azure CLI steps required to set up the repair environment.

---

## Optional Advanced Exploration

- Compare the contents of a healthy initramfs with a corrupt one using `lsinitrd` to understand what modules and files are included.
- Investigate what happens when you use `dracut --list-modules` to see all available dracut modules on the rescue VM.
- Explore what other ALAR run-ids are available beyond `linux-alar-fki` and what scenarios they address.
- Attempt to recover Lab 1 using ALAR scripts instead of the manual chroot approach — compare the effort and reliability.
- Investigate the difference between `add_drivers` and `force_drivers` in `/etc/dracut.conf` and when each should be used.
- Review the cloud-init logs (`/var/log/cloud-init.log`, `/var/log/cloud-init-output.log`) on the rescue VM to understand how the original VM was provisioned.

