# supportconfig — Quick Reference Cheatsheet

> **Applies to:** SUSE Linux Enterprise Server (SLES), openSUSE

---

## Installation

```bash
zypper install supportutils
```

---

## Basic Usage

| Action | Command |
|---|---|
| Generate a full default report | `supportconfig` |
| Display help / all options | `supportconfig -h` |

> **Note:** Run as root. The default archive is saved to `/var/log` in the format `scc_<HOST>_<DATE>_<TIME>.txz` (tar + xz). Older SUSE versions may use the legacy `nts_` prefix with `.tbz` (tar + bzip2) compression.

---

## Controlling Report Scope

| Action | Command |
|---|---|
| Minimal report (reduced data) | `supportconfig -m` |
| Limit to a specific topic | `supportconfig -i <KEYWORD>` |
| List all available topic keywords | `supportconfig -F` |

### Topic Examples

```bash
# Collect only boot-related information
supportconfig -i BOOT

# Collect only LVM-related information
supportconfig -i LVM

# Collect only network-related information
supportconfig -i NET

# List all available feature keywords
supportconfig -F
```

---

## Output Control

| Action | Command |
|---|---|
| Custom file name prefix | `supportconfig -B <prefix>` |
| Specify alternate output directory | `supportconfig -R <path>` |
| Quiet mode (less console output) | `supportconfig -Q` |

Default output: `/var/log/scc_<HOST>_<DATE>_<TIME>.txz`

Compressed with tar + xz.

---

## Working with the Archive

```bash
# List archive contents without extracting
tar tJf /var/log/scc_*.txz

# Extract the archive
tar xJf /var/log/scc_*.txz

# Check archive size
ls -lh /var/log/scc_*.txz
```

---

## Archive Structure

- All files are saved into a single directory
- Files end in `*.txt` for cross-platform compatibility
- Files are arranged by topic
- Each file contains:
  - Header
  - Command executed
  - Command output

---

## Plugins

- Plugin executables are located in `/usr/lib/supportconfig/plugins`
- Plugins dump output to stdout and stderr
- Plugin output is captured in `plugin-<name>.txt` files inside the archive

---

## Common Options — Quick Table

| Flag | Description |
|---|---|
| `-h` | Display help and all options |
| `-m` | Minimal report (reduced size) |
| `-i <KEYWORD>` | Limit report to a specific topic |
| `-F` | List all available feature keywords |
| `-B <prefix>` | Custom file name prefix |
| `-R <path>` | Alternate output directory |
| `-Q` | Quiet mode |
| `-l` | Log command output to a file |

---

## Size Comparison Quick Test

```bash
# Full report
supportconfig
ls -lh /var/log/scc_*.txz

# Minimal report
supportconfig -m
ls -lh /var/log/scc_*.txz

# Topic-limited report
supportconfig -i BOOT
ls -lh /var/log/scc_*.txz
```
