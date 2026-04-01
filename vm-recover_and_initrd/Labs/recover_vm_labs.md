# VM Recover Lab — SSH Configuration Recovery, Filesystem Repair, and Initrd Recovery

## Title and Scenario

This module provides four hands-on labs simulating real-world Azure Linux VM recovery scenarios. You will troubleshoot and recover VMs affected by:

- SSH daemon configuration corruption preventing remote access
- SSH listening port misconfiguration blocking connections on the default port
- Incorrect `/etc/fstab` entries causing non-boot scenarios
- Corrupted initrd/initramfs images resulting in kernel panics

Each lab deploys a pre-configured Azure VM with an injected issue. You must diagnose the problem, apply the correct recovery technique, and validate that the VM returns to a healthy state.

- This course/module was created for VM Recover LAB
- It will take aproximately 60 minutes.
- This module introduces you to the tools to Recover a VM.
- This Lab provides hands-on activities.
- After this course/module you will be able to recover a Linux configuration file using the sed tool.

---

## Deployment

All lab environments are deployed using the buttons below. Each button deploys a pre-configured VM with an issue for you to investigate.

**Lab 1 — Ubuntu 24.04 LTS VM (SSH configuration issue):**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fvm-recover_and_initrd%2FLabs%2FVMRecoverLab1.json)

**Lab 2** uses the same VM deployed in Lab 1. No additional deployment is required.

**Lab 3 — SLES 15 SP6 VM (non-boot fstab scenario):**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fvm-recover_and_initrd%2FLabs%2FVMRecoverLab3.json)

**Lab 4 — RHEL 9 VM (non-boot initrd scenario):**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fvm-recover_and_initrd%2FLabs%2FVMRecoverLab4.json)

---

## Skills Required

- Linux command-line fundamentals (file editing, process management, log inspection)
- Familiarity with SSH daemon configuration (`sshd_config`)
- Understanding of Linux boot process, `/etc/fstab`, and filesystem mounting
- Basic understanding of Azure VM management (portal navigation, Serial Console, Run Command)
- Working knowledge of Azure CLI (`az vm repair`)

---

## Recommended Prerequisites

- Completion of the Azure Linux Foundation modules (OS Fundamentals I & II)
- Access to an Azure subscription with permission to create VMs and resource groups
- Familiarity with at least one text editor (`vi`, `vim`, `nano`)
- WSL2, Cloud Shell, or another SSH client installed and configured

---

## Objectives

After completing all four labs, you will be able to:

- Verify the integrity of the `sshd_config` file using `sshd -t`
- Delete erroneous entries in the `sshd_config` file
- Manage the SSH daemon process using `systemctl`
- Verify daemon processes using `ps` and `ss`
- Run Linux commands using the "Run Command" feature in the Azure Portal
- Modify the SSH listening port using `sed`
- Identify non-boot scenarios through Serial Console logs and Boot Diagnostics
- Use the VM Serial Console to fix `/etc/fstab` errors
- Use the `az vm repair` extension to create a rescue VM and attach a faulty OS disk as a data disk
- Boot into single-user mode using the kernel parameter `init=/bin/bash`
- Correct or comment out offending `/etc/fstab` entries
- Use ALAR scripts via `az vm repair run` to recover a corrupt or missing initrd/initramfs image
- Verify successful boot and login after recovery

---

## Environment Overview

Each lab deploys a single Azure VM with the following characteristics:

| Lab | VM Name | Distribution | VM Size | NICs | Boot Diagnostics |
|-----|---------|-------------|---------|------|------------------|
| 1 & 2 | recoverlab1 | Ubuntu 24.04 LTS (Gen1) | Standard_B2s | 1 | Enabled |
| 3 | recoverlab3 | SLES 15 SP6 (Gen2) | Standard_B2s | 1 | Enabled |
| 4 | recoverlab4 | RHEL 9 (Gen2) | Standard_B2s | 1 | Enabled |

All VMs are deployed with:

- A single NIC with a static private IP (10.1.0.10) and a Standard public IP
- An NSG allowing inbound SSH (port 22) from the AzureCloud service tag
- Managed boot diagnostics enabled (required for Serial Console access)
- No data disks

---

## Your Mission

### Lab 1: Corruption of the sshd_config file

#### Scenario

In this scenario your customer may have upgraded a VM, installed new software or has accidentaly _mangled_ the sshd_config file.  Upon restarting the sshd daemon no new sshd connections can be established with the VM. You need to remove the offending lines.

#### Procedure

1. Deploy an Ubuntu 24.04 LTS VM. It will be asking for a password, be ready to provide it.  Also you need to complete information like Resource Group, pay attention to the empty spaces and change them.

    Deployment: See the Deployment section.

2. Once the VM is created, try to connect using SSH protocol, you can use any client you prefer, some examples are WSL2, CMD, CloudShell, etc.

3. Since SSH connections are failing, access the VM using the Serial Console. Press Enter to get the login prompt and log in with your azureadmin credentials.
4. Once connected via the Serial Console verify the status of the SSH daemon:

    ```bash
    sudo -i #to switch to root account
    ```

    ```bash
    systemctl status sshd  #If this command doesn't return you to the prompt, press letter "q" to quit the pager.
    ```

5. You can see the service has failed to start.   Try restarting it using:

    ```bash
    systemctl restart sshd
    ```

6. Check the status of the service again:

    ```bash
    systemctl status sshd
    ```

7.  As it continues failing, now let's go and check on the contents of the configuration file:

    ```bash
    cat /etc/ssh/sshd_config
    ```

8.  As you see, there is 1 erroneous line at the very end of the file, this text is causing the SSHD to fail when being restarted.
9.  The SSHD binary has a "-t" option that will test the integrity of its configuration file.  Run the command to verify:

     ```bash
     sshd -t
     ```

  **Note:** It's possible that a message pops up that says "missing privilege separation directory: /run/sshd" it's safe to ignore this warning.

You should see the configuration errors.

10. The configuration file can be corrected using various Linux tools with editors such as:  **vi, vim, nano** or file manipulation commands, for example: **sed, awk, nawk**
11. Use your favorite editor to remove the problematic line in sshd configuration file.
12. Verify you removed the line:

    ```bash
    cat /etc/ssh/sshd_config
    sshd -t
    ```

  **Note:** It's possible that a message pops up that says "missing privilege separation directory: /run/sshd" it's safe to ignore this warning.

13. Once the file has been corrected restart and verify the SSH service:

    ```bash
    systemctl restart sshd
    systemctl status sshd
    ```

14. List the daemon process using _ps_ command:

    ```bash
    ps -eaf |grep -i sshd
    ```

15.  Check if it is listening for new connections:

     ```bash
     ss -tupln|grep -i ssh
     ```

16. Ensure you are stil _root_; you can check using the comand _id_

    ```bash
    id
    ```

17. Track the SSH connection requests using the following command:

    ```bash
    tail -f /var/log/auth.log
    ```

18. Use a SSH client tool, such as _putty_ or _WSL_ to connect to this VM.

19. Check back in the Serial Console, the log file you are tailing in the Serial Console will show the connect and eventual disconnect messages, in this particular case we will not have any new lines (this is expected).  Press _Control+C_ to interrupt.  The service is currently up and running, but the SSH connections are still failing.  These scenarios occur in real life where is not just one thing that could be happening.  In Lab2, we'll continue the troubleshooting to finish fixing current scenario.

#### Your Goal

Let's summarize what you have learned after this lab:
- Verify the integrity of the sshd_config file using _sshd -t_.
- Delete erroneous entries in the sshd_config file.
- Manage the SSH daemon process using _systemctl_.
- Verify the daemon process using _ps, ss_.

---

### Lab 2: Run command to reset sshd_config default port

- The time to complete this lab is 30 minutes.
- After this lab you will be able to recover a Linux configuration file using _sed_ tool.

#### Scenario

In this scenario your customer may have changed the SSH default port and restarted the sshd service.  Now other users are unable to login.   You have to modify the sshd configuration file to set the port to default (22) through "run command" from the portal.

#### Procedure

For this lab we'll continue using the VM created previously as connections using SSH are currently not available using port 22.

1.  To start troubleshooting the issue you can verify the service status.

    ```bash
    systemctl status sshd
    ```

2.  Checking on the output of that command you can see the port is different that the default one (22), if you're not able to see it in the logs you can also check with command:

    ```bash
    journalctl -u sshd
    ```

3.  Another way to check on current listening port is filtering the configuration file and check directly in the port.   If the Port line starts with # symbol that means the VM is using default port to listen, which is 22.

    ```bash
    grep -i port /etc/ssh/sshd_config
    ```

4.  To fix the issue, go to the Azure Portal, select the VM.  Then, go to "Run command", select "RunShellScript" and add the below three commands to the "Linux Shell Script" section:

    ```bash
    sed -i 's/Port 2222/Port 22/g' /etc/ssh/sshd_config
    sshd -t
    systemctl restart sshd
    ```

5.  Select "Run" and wait until the execution is done, you'll get an output.  Check on it, the standard output(stdout) and standard error (stderr) should be empty.

6. Try to loging to the VM using an SSH client and port 22 and verify now you can access to it.  If you can't connect it, review the previous steps as you could miss some of them.

7.  Check the status of the sshd service with the following commands(in the connection you just established in previous step):

    ```bash
    systemctl status sshd
    journalctl -u sshd
    ss -tulpn |grep sshd
    ```

#### Your Goal

Let's summarize what you have learned after this lab:
- Run Linux command using the "run command" feature on the portal.
- Verify the integrity of the sshd_config file using _sshd -t_ command.
- Change the port to default in the sshd_config file using _sed_ command.
- Manage the SSH daemon process using _systemctl_ command.
- Verify the daemon process using _ss_ command.

---

### Lab 3: Recover failed VM due to fstab error with the help of _vm repair_ extension

- The time to complete this lab is 60 minutes.

#### Scenario

A VM may have stopped booting due to errors in the _/etc/fstab_ file or because the customer is using the scsi device name such as _/dev/sdc1_.  The scsi device name could get remapped to a different device.  This is not Azure functionality but is how SCSI standard functions.

Best practice is to utilize the UUID when mounting additional data disks or RAID volumes as this will guarantee the correct underlying disks are mounted.

#### Procedure

1. Deploy a SLES 15 SP6 VM. It will be asking for a password, be ready to provide it.

    Deployment: See the Deployment section.

2. Once the VM is created, confirm in the Serial Console and Serial log in Boot Diagnostics, you are in a non-boot scenario.  Check in the logs and identify the error.

3.  Using WSL or Cloud Shell proceed to install the _vm repair_ extension:

    ```bash
    az extension add --name vm-repair
    ```

4. Create a repair vm using the extension, please proceed to replace the information betwen "<>" symbols with the correct one:

    ```bash
    az vm repair create -g <resource_group_name_of_failing_vm> -n <failed_vm_name> --repair-username <temporary_username> --repair-password <temporary_password> --verbose
    ```

 **Note:** The execution on this command wil ask you if the VM requires a public ip, answer "y".

5. At the end of the command execution you will have the Repair VM name, please proceed to connect to it using the username and password selected and SSH client you prefer, next commands need to be executed in that connection.

6. Switch to root account using command:

    ```bash
    sudo -i
    ```

7. Proceed to identify the data disk, as it is the copy of the damage OS disk and the one we need to fix.  You can do that using commands:

    ```bash
    dmesg
    ```

   or

    ```bash
    tail -10 /var/log/kern.log
    ```

   the disk will be the last attached to the OS disk.   Another way to check is with the command:

    ```bash
    lsblk
    ```

   Identify the disks using _TYPE_ column, then check which is the disk that doesn't have _/_ mounted, that will be your disk.

8.  Proceed to identify the root partition, that will be the one that has bigger size in the data disk.
9. Create an empty directory using command:

    ```bash
    mkdir /rescue
    ```

10. Proceed to mount the OS partition with command:

    ```bash
    mount <partition> /rescue
    ```

    for example: _mount /dev/sdc1 /rescue_  remember to mount the correct partition, the partition name and number can change. You should have no output while mounting.

11. Edit the _/rescue/etc/fstab_ file and choose your preferred option to fix the issue:
- removing the problematic lines.
- commenting out the problematic lines adding a # symbol at the beginning of them.
- adding the option nofail in the problematic lines.

12.  Save the file, and double check you made the changes needed.

13.  Proceed to umount the filesystem with command:

     ```bash
     umount /rescue
     ```

14.  Swap the OS disk in the damage VM and delete the repair VM using the extension:

     ```bash
     az vm repair restore -g <resource_group_name_of_failing_vm> -n <failed_vm_name> --verbose
     ```

15. Check the VM now has a boot scenario.

#### Recover from the Serial Console

16.  Using Serial Console connect to the Lab VM and execute the following commands:

     ```bash
     echo "UUID=4ed50c8a-125c-4a9d-8ef0-846b40492f53  /datadrive   ext4   defaults   1   2" >> /etc/fstab
     reboot
     ```

17.  Check you have a non-boot scenario in place.
18.  From Serial Console Log screen in Azure Portal, proceed to restart the VM and stop the boot process pressing key _ESC_
19.  In the Grub Menu, proceed to edit the first line pressing letter _e_, it will allow you to edit the selected kernel, first line is the default one.
20.  Scroll down in the file and search for the line that starts with "linux" add the following at the end of the line:

     ```bash
     init=/bin/bash
     ```

21. Press _Ctrl+x_ to save changes and boot the operative system.  It will stop in single-user mode.
22. At this moment you have access to the system but it's read-only access, so change to read-write using the command:

    ```bash
    mount -o remount,rw /
    ```

23.  Now you can edit the file _/etc/fstab_ and fix the issue as you did previously in the repair vm.
24.  Save the changes and check you made them.
25.  Reboot the VM.

Utilizing the serial-console is the preferred way to fix simple non-boot scenario. The help of a recovery VM is always required if there is no access to the serial-console or recovery-steps require a full working environment.

#### Your Goal

You should know how to resolve incorrect entries in the /etc/fstab by:
- Identifying through serial log / boot diagnostics when a drive is not being mounted.
- Using the VM serial console to easily fix the fstab file.
- Using the vm repair extension to automatically create a rescue VM and attach the faulty OS disk as a data disk.
- Using the kernel parameter 'init' with the value '/bin/bash' to boot into the system without a password.
- Correcting or commenting out from /etc/fstab the offending entry or entries.

#### Reference

[Repair a Linux VM by using the Azure Virtual Machine repair commands - Virtual Machines | Microsoft Docs](https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/repair-linux-vm-using-azure-virtual-machine-repair-commands)

---

### Lab 4: Recover failed VM due to corrupt initrd with 'ALAR' scripts

- The time to complete this lab is 30 minutes.
- After this lab you will be able to recover a non boot-scenario with the help of the ALAR

#### Scenario

In this scenario your customer may have installed the new kernel or patched the VM and rebooted it.   Now, the VM stuck in boot due to corrupt or missing initrd.   You have to use ALAR scripts to regenerate initrd/initramfs image file and reboot the VM to make it available.

#### Procedure

1. Deploy a RHEL 9 VM. It will be asking for a password, be ready to provide it.

    Deployment: See the Deployment section.

2. Once the VM is created, check the non-boot scenario, the VM will stuck on boot due to initrd issue and a kernel panic will be showed in the Serial Console. Analyze the error.
3. Using WSL or Cloud Shell fix the issue executing the following commands, remember to replace the information like resource group with the correct one:

    ```bash
    az vm repair create --resource-group "<resource_group_of_failed_vm>" --name "<Failed_VM_Name>" --verbose --repair-username "<temporary_username>" --repair-password "<password>" #This command will ask if you need a public ip address, answer "n"
    az vm repair run --verbose --resource-group "<resource_group_of_failed_vm>" --name "<Failed_VM_Name>" --run-id linux-alar-fki --parameters initrd --run-on-repair
    az vm repair restore --verbose --resource-group "<resource_group_of_failed_vm>" --name "<Failed_VM_Name>"
    ```

4. Verify the VM is in a boot scenario.

#### Your Goal

Let's summarize what you have learned after this lab:
- Run CLI commands with ALAR scripts.
- Recover corrupt/missing initrd/initramfs image file with ALAR.
- Verify the succesful login to the VM after the recovery.

---

## Analytical Guidance

When troubleshooting VM recovery scenarios, follow a structured approach:

- Always start with the Azure Portal: check Boot Diagnostics, Serial Console logs, and Activity Log for recent changes.
- For SSH connectivity issues, investigate systematically: service status → configuration validation → listening ports → authentication logs.
- For non-boot scenarios, the Serial Console Log in Boot Diagnostics is your first source of diagnostic information — review it before taking action.
- Determine whether in-place repair (Serial Console) or offline repair (VM Repair extension) is the appropriate recovery path.
- When comparing configurations, identify what changed relative to the expected default behavior.
- For filesystem issues, always validate changes before rebooting — an incorrect repair can make the situation worse.

---

## Validation Criteria

Each lab is considered complete when:

- **Lab 1:** The SSH daemon is running, its configuration passes `sshd -t` validation, and the daemon is listening on the expected port.
- **Lab 2:** SSH connections to port 22 succeed and the service status confirms proper operation.
- **Lab 3:** The VM boots successfully, all filesystems mount correctly, and SSH access is restored. Both the VM Repair extension and Serial Console recovery paths have been completed.
- **Lab 4:** The VM boots without kernel panic, and SSH access is restored.

---

## Documentation Expectations

For each lab, document:

- The exact error messages or symptoms observed during diagnosis
- Which commands you used and in what order
- The root cause you identified
- How you remediated the issue
- The verification steps you performed to confirm the fix
- Any differences you observed between expected and actual system behavior

---

## What Not To Do

- Do not skip configuration validation (`sshd -t`, `mount -a`) before rebooting or restarting services
- Do not assume a single root cause — multiple issues may coexist in the same VM
- Do not edit configuration files without understanding the expected default state
- Do not rush to reboot a non-boot VM before confirming your changes are correct
- Do not skip Boot Diagnostics review when the Serial Console is also available
- Do not use the VM Repair extension without first checking whether the Serial Console can resolve the issue faster

---

## Real-World Context

These labs simulate scenarios encountered regularly in Azure Linux support:

- **SSH misconfiguration** is one of the most common causes of lost remote access to Linux VMs. Configuration file corruption can occur during package upgrades, manual edits, or automation drift.
- **Incorrect fstab entries** frequently cause VMs to fail to boot after a reboot, particularly when device names (e.g., `/dev/sdc1`) shift or when non-existent UUIDs are referenced. Using UUIDs and the `nofail` mount option are standard best practices.
- **Initrd/initramfs corruption** can occur during kernel updates, storage failures, or incomplete patching cycles. The ALAR (Azure Linux Auto Recovery) scripts are the standard tooling used by Azure support to recover from these scenarios without manual chroot operations.
- The **Azure VM Repair extension** (`az vm repair`) automates the rescue VM workflow: creating a repair VM, attaching the faulty OS disk, and restoring it after repair — a process that would otherwise require multiple manual Azure CLI steps.

---

## Optional Advanced Exploration

- Investigate the cloud-init logs (`/var/log/cloud-init.log`, `/var/log/cloud-init-output.log`) to understand how the VM was initially provisioned and configured.
- Attempt a CLI-only recovery for Lab 3 without using the Azure Portal at all.
- For Lab 3, compare the Serial Console recovery path with the VM Repair extension path: which is faster? Which is more reliable?
- Explore what other ALAR run-ids are available beyond `linux-alar-fki` and what scenarios they address.
- For Lab 4, manually perform the initrd regeneration inside a chroot environment instead of relying on ALAR scripts.
