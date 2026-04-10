# Repository Issues — Laboratories

## Title and Scenario

These hands-on laboratories accompany the **Repository Issues** module in the Azure Linux Academy — Specialist track. The labs cover the most common repository-related problems encountered on Azure Linux VMs: expired RHUI certificates, EUS-to-standard repository migration on end-of-life RHEL releases, outbound connectivity constraints imposed by Standard Internal Load Balancers, and broken SUSE product registration.

Each lab presents a different failure scenario. You must diagnose the issue, identify the root cause, and apply the correct remediation — using only the tools and documentation available to you.

**Estimated total time:** approximately 90 minutes across all labs.

---

## Deployment

Deploy each lab environment using the corresponding button or CLI commands below.

**Lab 1 — RHUI Certificate and EUS Migration (RHEL 8.4)**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Frepository_issues%2FLabs%2FRepositoryIssuesLab1.json)

**Lab 2 — VM Update with a Load Balancer in Place (RHEL 9)**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Frepository_issues%2FLabs%2FRepositoryIssuesLab2.json)

**Lab 3 — SUSE Base Product Registration (SLES 15 SP7)**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Frepository_issues%2FLabs%2FRepositoryIssuesLab3.json)

---

## Skills Required

- Querying and interpreting RPM package information (`rpm -qa`, `rpm -qi`)
- Inspecting X.509 certificates with `openssl`
- Understanding RHUI repository configuration and EUS vs. non-EUS repository types
- Using `yum` / `dnf` with repository filtering (`--disablerepo`, `--enablerepo`)
- Interpreting RHEL release versioning and release-lock behavior (`/etc/yum/vars/releasever`)
- Understanding Azure networking: Standard Internal Load Balancers, SNAT, NAT gateways, and outbound connectivity
- Azure CLI usage for VM and Load Balancer provisioning
- SUSE product registration using `SUSEConnect` and `registercloudguest`
- Interpreting symbolic links and product definition files on SUSE systems
- Reading and interpreting log files (`/var/log/cloudregister`)

---

## Recommended Prerequisites

- Completion of the **Repository Issues** knowledge module or equivalent experience
- Familiarity with RHEL and SUSE package management tools
- Understanding of RHUI architecture on Azure (PAYG vs. BYOS models)
- Basic Azure networking concepts (VNets, subnets, NSGs, Load Balancers)
- Ability to use the Azure CLI and Azure Serial Console

---

## Objectives

### Lab 1 — RHUI Certificate and EUS Migration

- Identify an expired RHUI certificate on a RHEL VM and understand its impact on package operations
- Determine that the VM is locked to an end-of-life EUS release (RHEL 8.4)
- Restore the RHUI certificate to a valid state
- Migrate the repository configuration from the EOL 8.4 EUS repositories to the standard RHEL 8.10 RHUI repositories
- Validate that `yum` / `dnf` operations succeed after remediation

### Lab 2 — VM Update with a Load Balancer in Place

- Identify why a VM behind a Standard Internal Load Balancer cannot reach external repositories
- Understand outbound connectivity requirements for Azure VMs behind Standard Load Balancers
- Apply a networking-level fix to restore outbound connectivity
- Validate that package updates complete successfully

### Lab 3 — SUSE Base Product Registration

- Identify why a SLES 15 SP7 VM reports as "Not Registered"
- Diagnose the root cause using SUSE registration logs and product configuration
- Apply the appropriate fix to restore registration
- Validate that the VM is registered and can receive updates

---

## Environment Overview

### Lab 1 — RHUI Certificate and EUS Migration

| Component        | Detail                          |
|------------------|---------------------------------|
| VM name          | `repositoryissues-lab1`         |
| OS               | RHEL 8.4 (raw, EUS)             |
| VM size          | Standard_B2s                    |
| NIC              | Single NIC, static IP `10.1.0.10` |
| Public IP        | Standard SKU, static            |
| NSG rules        | SSH (port 22) from AzureCloud |
| Boot diagnostics | Enabled                         |
| Admin user       | Defined at deployment           |

### Lab 2 — VM Update with a Load Balancer in Place

| Component        | Detail                          |
|------------------|---------------------------------|
| VM name          | `repositoryissues-lab2`         |
| OS               | RHEL 9 (LVM, Gen2)              |
| VM size          | Standard_B2s                    |
| NIC              | Single NIC, static IP `10.1.0.10` |
| Public IP        | None (no public IP assigned)    |
| Load Balancer    | Standard Internal (`labLoadBalancer`) |
| LB health probe  | TCP port 443                    |
| LB rule          | TCP 443 → 443                   |
| Subnet outbound  | `defaultOutboundAccess: false`  |
| NSG rules        | SSH (port 22) from AzureCloud   |
| Boot diagnostics | Enabled                         |
| Admin user       | Defined at deployment           |

### Lab 3 — SUSE Base Product Registration

| Component        | Detail                          |
|------------------|---------------------------------|
| VM name          | `repositoryissues-lab3`         |
| OS               | SLES 15 SP7 (Basic, Gen2)       |
| VM size          | Standard_B2s                    |
| NIC              | Single NIC, static IP `10.1.0.10` |
| Public IP        | Standard SKU, static            |
| NSG rules        | SSH (port 22) from AzureCloud   |
| Boot diagnostics | Enabled                         |
| Admin user       | Defined at deployment           |

---

## Your Mission

### Lab 1 — RHUI Certificate and EUS Migration

**Duration:** 30–45 minutes. If you do not make significant progress in 15 minutes, contact your instructor for hints.

1. Deploy one RHEL VM using the link in the **Deployment** section.
2. Connect to the VM via SSH or Serial Console.
3. Attempt to run a package update. Observe and document the failure.
4. Investigate the RHUI configuration: which repositories are configured, what type they are (EUS vs. standard), and whether the RHUI certificate is valid.
5. Determine the current RHEL release version and whether it is end-of-life.
6. Your objective is to:
   - Restore the VM to a state where package operations succeed.
   - Migrate the repository configuration from the EOL RHEL 8.4 EUS repositories to the standard RHEL 8.10 RHUI repositories.
   - Verify that `yum` / `dnf` can retrieve package metadata and install updates from the standard repositories.

### Lab 2 — VM Update with a Load Balancer in Place

**Duration:** 20–30 minutes. If you do not make significant progress in 15 minutes, contact your instructor for hints.

1. Deploy one RHEL 9 VM using the link in the **Deployment** section. The ARM template provisions the VM behind a Standard Internal Load Balancer with no public IP.
2. Connect to the VM (note: the VM has no public IP — consider how you will access it).
3. Attempt to update the VM using `dnf update -y`. Observe and document the failure.
4. Your objective is to make sure there are no pending updates in the VM.

### Lab 3 — SUSE Base Product Registration

**Duration:** 20–30 minutes. If you do not make significant progress in 15 minutes, contact your instructor for hints.

1. Deploy one SUSE VM using the link in the **Deployment** section.
2. Connect to the VM and switch to root account.
3. Check if the VM is registered:

   ```bash
   SUSEConnect -s
   ```

   The command should report the VM is _Not Registered_.

4. Your objective is to:
   - Diagnose why the VM is not registered.
   - Apply the appropriate fix to restore registration.
   - Verify that `SUSEConnect -s` shows the VM as Registered.
   - Confirm the VM can receive updates for the current version.

---

## Analytical Guidance

### Lab 1 — RHUI Certificate and EUS Migration

Begin by understanding the current state of the repository configuration:

- What RHUI client package is installed? Is it EUS or non-EUS?
- Is the RHUI content certificate valid? When does it expire — or has it already expired?
- What version of RHEL is the VM running? Is that version still supported under EUS?
- What is the current value of `/etc/yum/vars/releasever`? How does it affect which repositories are used?

Key diagnostic commands to consider:

```bash
rpm -qa | grep -i rhui
openssl x509 -in /etc/pki/rhui/product/content.crt -noout -text | grep -E 'Not Before|Not After'
cat /etc/redhat-release
cat /etc/yum/vars/releasever
yum repolist
```

Consider the order of operations carefully. If the certificate is expired, most `yum` operations against RHUI repositories will fail. You may need to find a way to update the RHUI package itself before changing the repository configuration.

The migration from EUS to standard RHUI involves changing both the RHUI client package and the release version lock. Refer to the Azure documentation on RHEL RHUI for guidance:

- [Red Hat Update Infrastructure for Azure](https://learn.microsoft.com/azure/virtual-machines/workloads/redhat/redhat-rhui)

### Lab 2 — VM Update with a Load Balancer in Place

This is a networking problem, not a package management problem. Before troubleshooting at the OS level, consider the network architecture:

- The VM has no public IP address.
- The VM is behind a Standard Internal Load Balancer.
- What are the outbound connectivity implications of this configuration?

Review the Azure documentation on outbound connectivity:

- [Load Balancer outbound connections](https://learn.microsoft.com/azure/load-balancer/load-balancer-outbound-connections)
- [Create a NAT gateway and associate it with an existing subnet](https://learn.microsoft.com/azure/nat-gateway/manage-nat-gateway?tabs=manage-nat-cli#create-a-nat-gateway-and-associate-it-with-an-existing-subnet)

Once you identify why the VM cannot reach external endpoints, apply a networking-level fix using the Azure CLI or portal.

### Lab 3 — SUSE Base Product Registration

Start by examining the registration logs to understand what happened during the initial registration attempt:

```bash
tail -n 100 /var/log/cloudregister
```

Then examine the product configuration:

```bash
ls -lh /etc/products.d/
```

Consider: what does the `baseproduct` symbolic link point to? Is it correct for this distribution? What happens if the registration system cannot identify the correct product?

---

## Validation Criteria

### Lab 1 — RHUI Certificate and EUS Migration

- [ ] The RHUI content certificate is valid (not expired)
- [ ] The RHUI client package corresponds to the standard (non-EUS) repository configuration
- [ ] The VM is configured for RHEL 8.10 (or the latest available standard release), not locked to 8.4
- [ ] `yum repolist` shows standard RHEL repositories (not EUS)
- [ ] `yum update -y` completes successfully
- [ ] You can explain the relationship between the expired certificate, the EUS lock, and the EOL status

### Lab 2 — VM Update with a Load Balancer in Place

- [ ] The VM has outbound internet connectivity
- [ ] `yum update -y` completes successfully with no pending updates
- [ ] You can explain why the VM could not reach RHUI before the fix
- [ ] The fix uses an appropriate Azure networking mechanism

### Lab 3 — SUSE Base Product Registration

- [ ] `SUSEConnect -s` reports the VM as Registered
- [ ] The `baseproduct` symlink points to the correct product definition
- [ ] The VM can install and update packages for the current SLES version
- [ ] You can explain what was misconfigured and why it prevented registration

---

## Documentation Expectations

For each lab, document:

- The initial error messages or symptoms observed when attempting package operations
- The sequence of diagnostic commands you ran and what each revealed
- The root cause identified
- The exact fix applied, including all commands and configuration changes
- Validation results confirming the fix (before and after states)
- Any relevant certificate dates, package versions, repository lists, or log excerpts

---

## What Not To Do

- Do not apply fixes without first completing the diagnostic investigation — understand what is broken before changing anything
- Do not blindly run `yum update` or `zypper update` repeatedly when the underlying issue is a certificate, repository configuration, or network problem
- Do not edit RHUI repository files manually — use the proper RHUI client package replacement workflow
- Do not assume that updating the RHUI package alone completes the EUS-to-standard migration — the release version lock must also be addressed
- Do not skip verifying outbound connectivity before troubleshooting package manager errors in Lab 2
- Do not overlook the registration logs on SUSE — `/var/log/cloudregister` contains critical diagnostic information
- Do not delete lab resources until all labs are complete and validated

---

## Real-World Context

### Lab 1 — RHUI Certificate and EUS Migration

Expired RHUI certificates and EUS release locks are among the most common repository issues on Azure RHEL VMs. When a RHEL minor version reaches end-of-life for EUS support, VMs locked to that version can no longer receive updates — even security patches. The migration from an EOL EUS release to the current standard RHUI is a routine but critical support operation. Getting the order of operations wrong (certificate renewal vs. package swap vs. release lock removal) can leave the VM in a state where no repositories are accessible.

### Lab 2 — VM Update with a Load Balancer in Place

VMs behind Standard Internal Load Balancers in Azure have no default outbound internet connectivity. This is by design — Azure's default outbound access does not apply when a Standard Load Balancer is associated with the VM's subnet. Support engineers frequently encounter cases where package updates, registration calls, or telemetry fail because the customer deployed a Standard ILB without provisioning an explicit outbound path (NAT gateway, public IP, or outbound rules). Understanding this networking constraint is essential for diagnosing "repository unreachable" symptoms that have no OS-level root cause.

### Lab 3 — SUSE Base Product Registration

SUSE PAYG images on Azure use `registercloudguest` to register with regional SMT/RMT servers at boot time. The registration process depends on the `baseproduct` symbolic link in `/etc/products.d/` pointing to the correct product definition file. If this link is broken or points to the wrong product, registration fails silently or with cryptic errors. This scenario occurs in practice when custom images are built incorrectly, when snapshots are restored across different SUSE products, or when manual changes corrupt the product configuration.

---

## Optional Advanced Exploration

### Lab 1

- After completing the migration, compare the output of `yum repolist` on EUS vs. standard configurations. How do the repository IDs and base URLs differ?
- Investigate what happens if you attempt to lock the VM back to a specific minor release using `/etc/yum/vars/releasever`. When would this be appropriate?
- Review the RHUI client package contents (`rpm -ql <package>`) to understand which files it installs and how it configures repository access.

### Lab 2

- Compare the three options for providing outbound connectivity to VMs behind Standard Load Balancers: NAT gateway, public IP on the VM, and outbound rules on the LB. What are the trade-offs of each approach?
- Explore using Azure Firewall with a service tag for RHUI to restrict outbound traffic to only the required destinations.
- Investigate `curl -v https://<rhui-ip>/` from the VM to observe the TLS handshake behavior when outbound connectivity is blocked vs. allowed.

### Lab 3

- Examine all `.prod` files in `/etc/products.d/` and compare their contents. How does the registration system use these files to determine which repositories to enable?
- Test what happens when you run `registercloudguest --force-new` with the wrong baseproduct link — observe the error in `/var/log/cloudregister`.
- Explore the difference between `SUSEConnect` and `registercloudguest` — when should each be used on Azure PAYG images?
