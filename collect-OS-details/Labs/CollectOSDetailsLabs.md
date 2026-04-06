# Collect OS Details — Hands-On Laboratory

**Estimated Duration:** ~30 minutes

---

## Title and Scenario

You are a Linux support engineer tasked with collecting system diagnostic information from two Azure virtual machines running different Linux distributions. Your environment includes a Red Hat Enterprise Linux (RHEL) VM and a SUSE Linux Enterprise Server (SLES) VM. You must use the appropriate OS-level diagnostic tooling for each distribution to generate, configure, and analyze system reports. You will also use Azure portal capabilities — Serial Console and Boot Diagnostics — to access and compare system boot information across both platforms.

---

## Deployment

Deploy both virtual machines using the links below. Fill in the required parameters (resource group, etc.) when prompted.

**Red Hat VM:**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fcollect-OS-details%2FLabs%2FCollectOSDetails-Lab1RHEL.json)

**SuSE VM:**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fcollect-OS-details%2FLabs%2FCollectOSDetails-Lab1SuSE.json)

---

## Skills Required

- Basic Linux command-line navigation and file management
- Package management fundamentals (yum/zypper)
- Familiarity with tar archive extraction
- Basic understanding of Linux system services and logging
- Azure portal navigation (Serial Console, Boot Diagnostics)

---

## Recommended Prerequisites

- Completion of OS Fundamentals I and II modules, or equivalent experience
- Basic familiarity with RHEL and SLES distributions
- Access to an Azure subscription with permissions to create resource groups and deploy VMs

---

## Objectives

At the end of this lab you will be able to:

- Create sos or supportconfig reports
- Enable specific modules
- Cope with not enough space issues
- Use Azure Serial Console and Boot Diagnostics to access and compare VM boot information

---

## Environment Overview

This lab deploys two Azure virtual machines:

| Component | Details |
|---|---|
| **RHEL VM** | `collectosdetails-redhat` — Red Hat Enterprise Linux 9 (Standard_B2s) |
| **SLES VM** | `collectosdetails-suse` — SUSE Linux Enterprise Server 15 SP7 (Standard_B2s) |
| **Network** | Single VNet (10.1.0.0/16), single subnet (10.1.0.0/24), one NIC per VM |
| **Access** | SSH via public IP; Azure Serial Console via portal |
| **Boot Diagnostics** | Enabled on both VMs |

Each VM is a single-NIC, single-disk instance with no additional data disks or complex networking. Boot diagnostics are enabled at deployment.

---

## Your Mission

### Part 1 — At the Red Hat VM

1. Check whether the sos package is installed, if not install it.

2. Switch to root account and generate a sosreport:

   ```bash
   sosreport
   ```

   - Review what plugins are currently enabled/disabled

     ```bash
     sosreport -l
     ```

   - What plugins are available
   - Edit the sosreport configuration file /etc/sos/sos.conf
   - Disable SELinux plugin
   - Run sosreport again and compare size and contents with previous report.

3. Generate a new report with Azure plugin:

   ```bash
   sosreport -e azure
   ```

   - At the end of the command execution, the output will show you a file name inside the /var/tmp path that start with "sosreport-\<something-variable\>". That's a tar.xz file. Review the report and contents of it. To do that you need to decompress information of the tar.xz file with command:

     ```bash
     tar xvf <sosreport-file-path>
     ```

   - After the command execution it will create a directory and inside it will have the collected WALinuxAgent information. You can review it with your preferred commands. The Azure plug-in module collects WALinuxAgent log and configuration details.

4. Generate a report with all logs:

   - Edit the sosreport configuration file /etc/sos/sos.conf
   - Enable SELinux plugin
   - Run the report:

     ```bash
     sosreport --all-logs
     ```

   - Compare the size of the original sosreport archive with this one. (Hint: `ls -l` provides you more detailed information on the files)
   - If you ran out of space, use the _--tmp-dir_ option to specify an alternate location.

### Part 2 — At the SuSE VM

1. Switch to root account and install the supportutils package:

   ```bash
   zypper install supportutils
   ```

2. Generate a report:

   ```bash
   supportconfig
   ```

   - Review the contents of the archive generated. Review the contents of several of the *.txt files created.

3. Run a report with the minimal option:

   ```bash
   supportconfig -m
   ```

   - Compare the size with the original archive.

4. Run a report again limiting the report to the BOOT topic:

   ```bash
   supportconfig -i BOOT
   ```

   - Review the information generated.

### Part 3 — Any of the VMs Created

1. Select one of the 2 VMs created on this Lab.
2. On the Portal, use the Serial Console to access the OS.
3. Use boot diagnostics to review the Serial Log.
4. Compare the information in the boot.txt file generated in the SLES supportconfig to the equivalent in the Red Hat sosreport.

---

## Analytical Guidance

- When comparing sosreport archives of different sizes, consider what additional data each option collects and why size differences matter for storage-constrained systems.
- When reviewing plugin output, pay attention to the directory structure within the extracted archive — each plugin maps to a specific subdirectory.
- When working with supportconfig, note how the file naming convention and organization differs from sosreport.
- When comparing boot information across Azure portal tools and OS-level reports, observe what information overlaps and what is unique to each source.

---

## Validation Criteria

- [ ] sosreport package is confirmed installed on the RHEL VM
- [ ] A baseline sosreport archive is generated and its size is recorded
- [ ] The SELinux plugin is successfully disabled via `/etc/sos/sos.conf` and a second report is generated with a measurable size difference
- [ ] A report with the Azure plugin (`-e azure`) is generated and WALinuxAgent data is verified inside the extracted archive
- [ ] The SELinux plugin is re-enabled and a report with `--all-logs` is generated; file size is compared against the baseline
- [ ] supportutils is installed on the SLES VM
- [ ] A full supportconfig archive is generated and its contents are reviewed
- [ ] A minimal supportconfig (`-m`) is generated and its size is compared to the full archive
- [ ] A BOOT-topic-only supportconfig (`-i BOOT`) is generated and reviewed
- [ ] Serial Console access is verified via the Azure portal on at least one VM
- [ ] Boot diagnostics Serial Log is reviewed via the portal
- [ ] Boot information from the SLES supportconfig `boot.txt` is compared with the equivalent data in the Red Hat sosreport

---

## Documentation Expectations

For each part of this lab, document:

- The commands executed and their output (or key excerpts)
- Archive file paths, names, and sizes at each stage
- Observations about plugin structure and archive contents
- Size comparison notes between different report configurations
- Any errors encountered and how they were addressed (e.g., space issues)

---

## What Not To Do

- Do not skip the size comparison steps — understanding report size implications is part of the learning objective
- Do not extract archives without reviewing the directory structure before diving into individual files
- Do not ignore errors from sosreport or supportconfig — these may indicate missing packages, permissions, or disk space issues
- Do not skip the Azure portal exercises (Serial Console and Boot Diagnostics) — these are integral to the full diagnostic workflow

---

## Real-World Context

In production support scenarios, sosreport and supportconfig are the first tools a support engineer runs when a Linux system issue is escalated. The ability to quickly generate, transfer, and analyze these archives is a foundational skill. Understanding which plugins to enable (or disable for privacy/size reasons), how to work around disk space constraints, and how to correlate OS-level diagnostic data with Azure platform telemetry (Serial Console output, Boot Diagnostics) is essential for effective incident triage. These reports are routinely required when opening support tickets with Red Hat and SUSE, respectively.

---

## Optional Advanced Exploration

- Experiment with generating sosreport using the newer `sos report` command syntax (available on recent RHEL versions) and compare behavior with the legacy `sosreport` command.
- Use `sosreport -o <plugin-name>` to generate a report limited to a single plugin and inspect the resulting archive structure.
- Use `sosreport -k yum.yumlist` to explore plugin-specific options and compare the additional data collected.
- On the SLES VM, explore `supportconfig -F` to list all available feature keywords, then generate topic-specific reports for two or three different keywords.
- Compare the directory/file layout of an extracted sosreport archive vs. an extracted supportconfig archive — document the structural differences.
- Try running sosreport with `--tmp-dir` pointing to a location with limited space to observe how the tool behaves under storage constraints.


