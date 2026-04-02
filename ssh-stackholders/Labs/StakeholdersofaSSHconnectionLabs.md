# Stakeholders of a SSH Connection Lab — SSH Service, Routing, and Firewall Troubleshooting

## Title and Scenario

This module provides three hands-on labs simulating real-world Azure Linux VM SSH connectivity issues. You will troubleshoot and recover VMs affected by:

- SSH daemon configuration changes preventing remote access
- Missing routing table entries causing network isolation
- Firewall rule manipulation (iptables) blocking SSH traffic

Each lab deploys a pre-configured Azure VM with an injected issue. You must diagnose the problem, apply the correct fix, and validate that SSH connectivity is restored.

- This course/module was created for the module Stakeholders of a SSH connection.
- It will take approximately 60 minutes.
- This module introduces you to Stakeholders of a SSH connection.
- This Lab provides hands-on activities.
- After this course/module you will be able to:
    1. Locate an issue at service level.
    2. Understand why routing information is required and how to alter it
    3. Create simple iptables rules and understand their order

---

## Deployment

All lab environments are deployed using the buttons below. Each button deploys a pre-configured VM with an injected SSH connectivity issue for you to investigate.

**Lab 1 — Ubuntu 24.04 LTS VM (SSH service issue):**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fssh-stackholders%2FLabs%2FStakeholdersSSHConnectionLab1Ubuntu.json)

**Lab 1 — RHEL 9 VM (SSH service issue):**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fssh-stackholders%2FLabs%2FStakeholdersSSHConnectionLab1RHEL.json)

**Lab 2 — RHEL 8.10 VM (routing issue):**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fssh-stackholders%2FLabs%2FStakeholdersSSHConnectionLab2.json)

**Lab 3 — RHEL 9 VM (iptables issue):**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fssh-stackholders%2FLabs%2FStakeholdersSSHConnectionLab3.json)

---

## Skills Required

- Linux command-line fundamentals (file editing, process management, log inspection)
- Familiarity with SSH daemon configuration (`sshd_config`)
- Understanding of Linux networking concepts (routing tables, `ip route`, default gateway)
- Basic understanding of iptables firewall rules, chains, and rule ordering
- Basic understanding of Azure VM management (portal navigation, Serial Console)

---

## Recommended Prerequisites

- Completion of the Azure Linux Foundation modules (OS Fundamentals I & II)
- Access to an Azure subscription with permission to create VMs and resource groups
- Familiarity with at least one text editor (`vi`, `vim`, `nano`)
- WSL2, Cloud Shell, or another SSH client installed and configured

---

## Objectives

After completing all three labs, you will be able to:

- Identify SSH connectivity issues caused by daemon configuration changes
- Use the Azure Serial Console to access a VM when SSH is unavailable
- Diagnose and repair missing routing table entries on a Linux VM
- Understand the role of Azure wireserver (168.63.129.16) and metadata (169.254.169.254) routes
- Create, list, and delete iptables firewall rules
- Understand iptables rule ordering and why rule position matters
- Flush iptables rules across different tables (filter, nat, mangle, security)

---

## Environment Overview

Each lab deploys a single Azure VM with the following characteristics:

| Lab | VM Name | Distribution | VM Size | NICs | Boot Diagnostics |
|-----|---------|-------------|---------|------|------------------|
| 1 (Ubuntu) | sshlab1ubuntuvm | Ubuntu 24.04 LTS (Gen1) | Standard_B2s | 1 | Enabled |
| 1 (RHEL) | sshlab1rhel | RHEL 9 (Gen1) | Standard_B2s | 1 | Enabled |
| 2 | sshlab2 | RHEL 8.10 | Standard_B2s | 1 | Enabled |
| 3 | sshlab3 | RHEL 9 (Gen1) | Standard_B2s | 1 | Enabled |

All VMs are deployed with:

- A single NIC with a static private IP (10.1.0.10) and a Standard public IP
- An NSG allowing inbound SSH (port 22) from the AzureCloud service tag
- Managed boot diagnostics enabled (required for Serial Console access)
- No data disks
- Password-based authentication (user: azureuser)

---

## Your Mission

### Lab 1: SSH Service Level Issue

#### Instructions

1. Deploy an Ubuntu 24.04 LTS VM.

    Deployment: See the Deployment section.

2. Deploy a RHEL 9 VM.

    Deployment: See the Deployment section.

#### Task

Try to log in to these 2 VMs.  Find out what could be the reason you cannot connect.

---

### Lab 2: Missing Route Information

#### Instructions

1. Deploy a RHEL 8.10 VM.

    Deployment: See the Deployment section.

After deploying the VM, access to the VM using SSH protocol is lost and it is required to use Serial Console.

#### Task

1. Connect to the Serial Console using "azureuser" account and the password provided during the VM deployment.   Switch to root account and get the routing-table information.  You'll see there is no information available.
2. To get access to the internet and the datacenter add the missing route information manually to the VM.  Use `ip route` to do this.

The missing routes are:

- Host: 168.63.129.16
- Host: 169.254.169.254
- default-route
- The default network each VM is part of needs to be added to the OS as well, use `ip add` to see what network is assigned to the interface or get the info from ASC e.g., 10.240.0.0/16
- Keep an eye on the default route. If this is missing what does happen?  Remove the route again if required.

---

### Lab 3: Iptables Firewall Rules

#### Instructions

1. Deploy a RHEL 9 VM.

    Deployment: See the Deployment section.

iptables services are not installed by default on recent versions of Red Hat, so during the deployment you created a Red Hat VM and we stopped and masked firewalld and installed and started iptables services for you.

#### Task

1.  Connect to the VM's Serial Console and switch to root account.
2.  Create a drop rule for SSH to block incoming traffic:

    ```bash
    iptables -A INPUT -p tcp --dport 22 -j DROP
    ```

    Are you able to logon via SSH to the VM? if yes, restart the sshd service and try again.

3.  Create a rule to drop incoming traffic for all but your IP address; to determine your public IP on a Linux system, use `w` or `last` and check the "FROM" column for your IP, then proceed with creating the rule. Note: the source IP will be the public IP from which you are connecting to the VM, or the IP assigned to the Azure VPN gateway if you are connecting through a VPN. Change <YOUR_IP_ADDRESS> with this information in the following command:

    ```bash
    iptables -A INPUT -p tcp --dport 22 -s <YOUR_IP_ADDRESS> -j ACCEPT
    ```

    After adding this new rule are you now able to log on via SSH?

4.  Although you have added a rule to accept traffic from your IP, you are blocked.  Why?

    The reason is the order of the rules.  Use the following command to see the rules applied and ruleset-number:

    ```bash
    iptables -L -n -v --line-numbers
    ```

5.  Delete a rule.   To get access to the VM again the first rule must be deleted.  Use the following command to get the ruleset-number of the DROP rule from step 2:

    ```bash
    iptables -L --line-numbers
    ```

    Now, delete the rule:

    ```bash
    iptables -D INPUT <RULE_NUMBER>
    ```

    Access to the VM is possible after this step.

6.  List all the rules.  To see what rules are active on the VM you can use the following command:

    ```bash
    iptables -L
    ```

    By default, the filter table is displayed.  To see a different table, use the option _-t_, and the table name.   The tables are:
    - filter
    - nat
    - mangle
    - security

7.  To flush (disable) all rules use this command:

    ```bash
    iptables -F
    ```

    By default, the filter table is flushed only.   We have rules in the security table.  Please remove them.

---

## Analytical Guidance

When troubleshooting SSH connectivity issues, follow a layered approach:

- Start from the Azure platform layer: check NSG rules, Boot Diagnostics, and Serial Console availability.
- Move to the OS networking layer: verify routes exist (`ip route`), check that the SSH daemon is running and listening on the expected address family and port.
- Then check the host firewall layer: inspect iptables/firewalld rules and their ordering.
- For SSH service issues, examine the `sshd_config` file for configuration changes that alter listening behavior (address family, port, interface bindings).
- For routing issues, remember that Azure VMs require specific routes to reach the wireserver (168.63.129.16) and metadata service (169.254.169.254) in addition to the default gateway.
- For iptables issues, always consider rule ordering — rules are evaluated top-to-bottom, and the first match wins.

---

## Validation Criteria

Each lab is considered complete when:

- **Lab 1:** You have identified why SSH connections fail to both VMs and understand the configuration change that caused the issue.
- **Lab 2:** The VM has a complete routing table with all required routes (wireserver, metadata, default gateway, subnet), and SSH access is restored.
- **Lab 3:** You have successfully created, inspected, reordered, and deleted iptables rules, and understand why rule ordering matters. All iptables rules across all tables have been flushed.

---

## Documentation Expectations

For each lab, document:

- The exact symptoms observed when attempting to connect via SSH
- Which diagnostic tools and commands you used to identify the root cause
- The specific fix you applied and why it resolves the issue
- The verification steps you performed to confirm SSH connectivity was restored
- Any differences you observed between expected and actual system behavior

---

## What Not To Do

- Do not skip checking the Serial Console when SSH access fails — it is your primary recovery path
- Do not assume SSH failures are always caused by the SSH daemon — network and firewall layers must also be investigated
- Do not add iptables rules without understanding where they will be inserted relative to existing rules
- Do not flush only the default (filter) table when rules may exist in other tables (nat, mangle, security)
- Do not forget that route changes made with `ip route` are not persistent across reboots unless saved to configuration files

---

## Real-World Context

These labs simulate scenarios encountered regularly in Azure Linux support:

- **SSH configuration changes** such as restricting the address family can silently break SSH access over IPv4. This occurs when administrators or automation tools modify `sshd_config` without understanding the impact on connectivity.
- **Missing routes** can occur after network reconfigurations, failed DHCP renewals, or manual route table manipulation. Without routes to the Azure wireserver and metadata service, the VM loses access to critical platform services including extensions, password resets, and health monitoring.
- **Iptables misconfiguration** is a common cause of self-inflicted SSH lockouts. Understanding rule ordering (first-match-wins) is essential — a DENY rule placed before an ACCEPT rule for the same traffic will block access regardless of the ACCEPT rule's existence.

---

## Optional Advanced Exploration

- Investigate how `AddressFamily` settings interact with different SSH client connection attempts (IPv4 vs IPv6).
- For Lab 2, determine which routes are provided by DHCP and which must be manually maintained.
- For Lab 3, explore the difference between iptables `-I` (insert at position) and `-A` (append) and how insertion position affects rule evaluation.
- Compare the behavior of `iptables -F` (flush rules) versus `iptables -X` (delete user-defined chains) and `iptables -Z` (zero counters).
- Investigate how `firewalld` and `iptables` interact when both are present on the same system.
- Examine the `nft` (nftables) framework as the modern replacement for iptables on RHEL 9 and newer distributions.


