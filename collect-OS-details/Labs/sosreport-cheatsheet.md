# sosreport — Quick Reference Cheatsheet

> **Applies to:** RHEL, CentOS, Oracle Linux, Ubuntu

---

## Installation

| Distribution | Command |
|---|---|
| RHEL / CentOS / Oracle Linux | `yum install sos` |
| Ubuntu | `apt install sosreport` |

Verify installation:

```bash
# RHEL / CentOS / Oracle Linux
rpm -qa | grep sos

# Ubuntu / Debian
dpkg -l | grep sosreport
```

---

## Basic Usage

| Action | Command |
|---|---|
| Generate a default report | `sosreport` |
| Generate a report (newer syntax) | `sos report` |
| Clean up old reports | `sos clean` |

> **Note:** On recent distributions, `sos report` replaces the legacy `sosreport` command. Both are included here for reference.

---

## Plugin Management

| Action | Command |
|---|---|
| List all plugins (enabled/disabled) | `sosreport -l` |
| List all plugins (newer syntax) | `sos report -l` |
| Enable a specific plugin | `sosreport -e <plugin>` |
| Disable (skip) specific plugins | `sosreport -n <plugin1>,<plugin2>` |
| Run only a specific plugin | `sosreport -o <plugin>` |
| Pass an option to a plugin | `sosreport -k <plugin>.<option>` |

### Examples

```bash
# Enable the Azure plugin
sosreport -e azure
sos report -e azure

# Disable the kvm and zfs plugins
sosreport -n kvm,zfs

# Run only the Azure plugin
sosreport -o azure

# Include yum repository list via plugin option
sos report -k yum.yumlist
```

---

## Configuration File

Edit `/etc/sos/sos.conf` to set persistent defaults:

```bash
vi /etc/sos/sos.conf
```

The configuration file is segmented by section for easy modification of plugin options, enabled/disabled plugins, and general settings.

### Example Configuration Snippets

```ini
[global]
# Change default temporary directory for report output
tmp-dir = /mnt/data/sosreports

# Run in non-interactive (batch) mode by default
batch = yes

[plugins]
# Disable specific plugins (comma-separated)
disable = kvm, zfs, ovirt

# Enable specific plugins that are off by default
enable = azure, docker

[plugin_options]
# Include full yum repository listing
yum.yumlist = on

# Collect all logs for the networking plugin
networking.traceroute = on
```

---

## Output and Size Control

| Action | Command |
|---|---|
| Generate report with all logs (larger archive) | `sosreport --all-logs` |
| Specify alternate output directory | `sosreport --tmp-dir /path/to/dir` |
| Set a custom case ID | `sosreport --case-id <ID>` |
| Limit report to a batch (non-interactive) | `sosreport --batch` |

> **Tip:** The `--all-logs` option removes size limits on log collection and can significantly increase archive size. Use `--tmp-dir` if `/var/tmp` has limited space.

---

## Working with the Archive

Default output location: `/var/tmp/sosreport-<hostname>-<date>-<hash>.tar.xz`

```bash
# List archive contents without extracting
tar tf /var/tmp/sosreport-*.tar.xz

# Extract the archive
tar xvf /var/tmp/sosreport-*.tar.xz

# Check archive size
ls -lh /var/tmp/sosreport-*.tar.xz
```

---

## Azure-Specific

- RHEL 7.4+ ships with the Azure plugin module
- The Azure plugin collects WALinuxAgent log and configuration details
- Update the sos package on RHEL 7.2/7.3 to add Azure plugin support

```bash
# Generate a report with the Azure plugin enabled
sosreport -e azure

# Generate a report collecting only Azure plugin data
sosreport -o azure
```

---

## Common Options — Quick Table

| Flag | Description |
|---|---|
| `-l` | List all plugins and their status |
| `-e <plugin>` | Enable a plugin |
| `-n <plugin>` | Skip (disable) a plugin |
| `-o <plugin>` | Run only the specified plugin |
| `-k <plugin>.<opt>` | Pass a plugin-specific option |
| `--all-logs` | Collect all logs (no size limits) |
| `--tmp-dir <path>` | Alternate output directory |
| `--batch` | Non-interactive mode (no prompts) |
| `--case-id <ID>` | Tag the report with a case/ticket ID |
| `-h` | Display help |
