# Service Troubleshooting Labs

## Title and Scenario

These labs are part of the **Azure Linux Academy — Specialist** track and cover two essential aspects of systemd service management on Azure Linux VMs:

1. **Lab 1 — Service Troubleshooting**: A break-fix exercise on a RHEL 9 VM where a critical service is not functioning correctly. You must diagnose and remediate the issue using only the tools available on the system and through the Azure portal.
2. **Lab 2 — Unit Production**: A productive exercise on a SUSE 15 SP7 VM where you must design and implement a systemd-based scheduling solution without relying on cron.

**Estimated total time:** approximately 90 minutes across both labs.

---

## Deployment

Deploy each lab environment using the corresponding button below. Each button provisions a single Azure VM with the required configuration.

**Lab 1 — Service Troubleshooting (RHEL 9)**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fservice_troubleshooting%2FLabs%2Fservice-lab1.json)

**Lab 2 — Unit Production (SUSE 15 SP7)**

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fspecialist%2Frefs%2Fheads%2Fspecialist2026%2Fservice_troubleshooting%2FLabs%2Fservice-lab2.json)

---

## Skills Required

- Navigating and interpreting systemd service states (`systemctl status`, `systemctl show`)
- Reading and reasoning about unit files (`systemctl cat`)
- Querying and filtering the journal (`journalctl`)
- Understanding service dependencies and ordering
- Identifying environment, permission, and configuration-related failures
- Creating and managing systemd unit files (`.service`, `.timer`)
- Applying safe changes using drop-ins and overrides
- Using the Azure Serial Console for out-of-band access

---

## Recommended Prerequisites

- Familiarity with the **Service Troubleshooting** module content
- Comfortable with Linux shell usage and basic file editing
- Basic understanding of `systemd` service lifecycle (start, stop, enable, mask)
- Ability to use `sudo` and navigate the filesystem
- Familiarity with Azure VM Serial Console access

---

## Objectives

### Lab 1 — Service Troubleshooting

- Diagnose why SSH connectivity to a RHEL 9 VM is failing
- Identify the root cause using systemd tooling (status, logs, unit inspection, dependencies)
- Apply a targeted fix and validate that the service recovers
- Confirm that the fix persists across a reboot

### Lab 2 — Unit Production

- Design a systemd-based solution to run a web server on port 8080 for exactly five minutes every hour
- Implement the solution without using cron
- Validate that the timer activates the service on schedule and that the service stops after five minutes
- Demonstrate understanding of systemd timer and service unit relationships

---

## Environment Overview

### Lab 1 — Service Troubleshooting

| Component        | Detail                      |
|------------------|-----------------------------|
| VM name          | `service-lab1`              |
| OS               | RHEL 9 (raw, Gen2)          |
| VM size          | Standard_B2s                |
| NIC              | Single NIC, static IP `10.1.0.10` |
| Public IP        | Standard SKU, static        |
| NSG rule         | SSH (port 22) from AzureCloud |
| Boot diagnostics | Enabled                     |
| Admin user       | Defined at deployment       |

### Lab 2 — Unit Production

| Component        | Detail                      |
|------------------|-----------------------------|
| VM name          | `service-lab2`              |
| OS               | SUSE 15 SP7 (Gen2)          |
| VM size          | Standard_B2s                |
| NIC              | Single NIC, static IP `10.1.0.10` |
| Public IP        | Standard SKU, static        |
| NSG rule         | SSH (port 22) from AzureCloud |
| Boot diagnostics | Enabled                     |
| Admin user       | Defined at deployment       |

---

## Your Mission

### Lab 1 — Service Troubleshooting

**Duration:** 30–45 minutes. If you do not make significant progress in 15 minutes, please contact your instructor for hints.

1. Deploy one RHEL VM using the link in the **Deployment** section.
2. Confirm you can connect to the VM using Serial Console.
3. Try to connect to the VM using SSH protocol with any client you prefer. You will find it is not working.
4. At this point, the VM is broken. Your objective is to troubleshoot and fix whatever is broken.

### Lab 2 — Unit Production

**Duration:** 30–45 minutes. If you do not finish this exercise in time, please let your instructor know.

1. Deploy one SUSE VM using the link in the **Deployment** section.
2. Using systemd units, a web server needs to be run on port 8080 for only five minutes every hour. This should be set up without using cron.
3. You can use this Python line to run a simple web server:

```bash
/usr/bin/python3 -m http.server 8080
```

4. Share your solution with the class.

---

## Analytical Guidance

### Lab 1 — Service Troubleshooting

Begin by establishing what you can observe and what you cannot:

- Can the VM be reached through the Azure Serial Console? If yes, the VM itself is running.
- What does `systemctl status` tell you about the relevant service? Is it loaded? Active? Failed?
- What do the logs say? Use `journalctl` to build a timeline around the failure.
- What does the unit file itself say? Use `systemctl cat` to review the effective configuration.
- Are there dependency or ordering issues? Use `systemctl list-dependencies` to inspect relationships.

Work through the standard troubleshooting flow:

```bash
systemctl status <service>
journalctl -u <service> -b
systemctl cat <service>
systemctl list-dependencies <service>
```

Cross-reference what you find across these views. The root cause may not appear in a single command output.

### Lab 2 — Unit Production

Consider the following when designing your solution:

- Which systemd unit types are relevant for scheduling recurring tasks? How do `.timer` and `.service` units relate?
- How do you express "run for five minutes" in a systemd context? Consider `RuntimeMaxSec`, `Type`, and timer accuracy.
- How do you express "every hour" without cron? What timer options control periodicity?
- How do you validate that the timer is registered and active?

Useful inspection commands:

```bash
systemctl list-timers --all
systemctl status <your-timer>
systemctl status <your-service>
journalctl -u <your-service> -b
```

---

## Validation Criteria

### Lab 1 — Service Troubleshooting

- [ ] SSH connectivity to the VM is restored and functional
- [ ] The relevant service is in an `active (running)` state
- [ ] The fix persists after a full VM reboot (`sudo reboot`)
- [ ] You can explain what was broken and why the fix resolves it

### Lab 2 — Unit Production

- [ ] A `.service` unit exists that starts the Python web server on port 8080
- [ ] A `.timer` unit exists that triggers the service every hour
- [ ] The service runs for exactly five minutes and then stops
- [ ] The timer is enabled and active (`systemctl list-timers` confirms scheduling)
- [ ] The solution does not use cron
- [ ] The solution survives a reboot (timer remains enabled)

---

## Documentation Expectations

For each lab, document:

- The sequence of commands you ran and what each revealed
- The specific findings at each diagnostic step
- The root cause or design rationale (Lab 1: what was broken and why; Lab 2: why you chose a particular approach)
- The exact fix or implementation applied
- Validation results (before and after state)
- Any side observations or unexpected behavior encountered

---

## What Not To Do

- Do not skip the Serial Console verification step — it establishes that the VM is operational at the OS level
- Do not guess at fixes without first completing the diagnostic flow (status → logs → unit file → dependencies)
- Do not edit vendor-provided unit files directly — use `systemctl edit` to create overrides
- Do not rely solely on `systemctl status` — the embedded log tail is often insufficient
- Do not use cron for Lab 2 — the exercise explicitly requires a systemd-native solution
- Do not assume network readiness guarantees — verify that services depending on the network account for ordering

---

## Real-World Context

### Lab 1 — Service Troubleshooting

SSH connectivity failures are among the most common support cases for Linux VMs in Azure. In production, the inability to SSH into a VM triggers immediate escalation. The Azure Serial Console provides an out-of-band access path that bypasses network-layer issues entirely. Building instinct for when to pivot from network debugging to service debugging — and how to use systemd tooling to rapidly isolate the failure — is a core SRE skill.

### Lab 2 — Unit Production

Systemd timers are the modern replacement for cron on systemd-managed systems. They offer advantages including dependency awareness, structured logging via the journal, resource control integration, and calendar or monotonic scheduling. Understanding how to compose `.timer` and `.service` units is essential for production workloads where scheduled tasks must be observable, auditable, and integrated with the service manager.

---

## Optional Advanced Exploration

### Lab 1

- After restoring connectivity, review the full boot journal (`journalctl -b`) and identify all services that started before and after the broken service. Consider what would happen if other services depended on it.
- Investigate whether `systemctl edit` can be used to add safeguards (e.g., `Restart=on-failure`) to make the service more resilient to future misconfigurations.

### Lab 2

- Extend your solution to also log the start and stop times of the web server to a file under `/var/log/`.
- Experiment with `OnCalendar` versus `OnUnitActiveSec` and observe the behavioral differences.
- Add a `[Install]` section to your timer and verify that `systemctl enable` creates the correct symlinks.
- Explore `AccuracySec` and `RandomizedDelaySec` to understand how systemd manages timer precision.
