# Chapter 14 — Package Management

Package management is the method of installing and maintaining software on the system. Most Linux software is distributed through **repositories** — curated collections of packages built and maintained for a specific distribution.

## Two Packaging Families

| Family        | Format | Distributions                                      |
| ------------- | ------ | -------------------------------------------------- |
| Debian-style  | `.deb` | Debian, Ubuntu, Linux Mint, Raspbian               |
| Red Hat-style | `.rpm` | Fedora, CentOS, Red Hat Enterprise Linux, OpenSUSE |

> \*openSUSE uses `.rpm` but uses `zypper` instead of dnf.

A package from one family is **not compatible** with the other.

## Key Concepts

- **Package file** — a compressed archive containing program files, metadata, and optional pre/post-install scripts
- **Repository** — a central server hosting thousands of packages for a distribution
- **Dependency** — a package that another package requires to function; high-level tools resolve these automatically
- **Package maintainer** — the person who compiles upstream source code, packages it, and keeps it updated for a distribution

## Tools: High-Level vs Low-Level

| | Debian | Red Hat |
| - | ------ | ------- |
| **Low-level** (install/remove files, no dependency resolution) | `dpkg` | `rpm` |
| **High-level** (search, resolve dependencies, talk to repos) | `apt-get`, `apt` | `yum`, `dnf` |

> Always prefer high-level tools. Low-level tools skip dependency resolution and will error if something is missing. `yum` is legacy. On modern systems it is usually a wrapper around `dnf`

## Common Tasks

### Search for a Package

```bash
apt search search_string # Debian  
dnf search search_string # Fedora/RHEL
```

### Install from a Repository

```bash
apt update && apt install package_name    # Debian
dnf install package_name                  # Fedora
```

### Install from a Local Package File

```bash
dpkg -i package.deb     # Debian
dnf install package.rpm # Fedora (preferred)
rpm -i package.rpm      # Low-level
```

> On Fedora prefer `dnf install file.rpm` instead of `rpm -i`  because dnf resolves dependencies automatically
### Remove a Package

```bash
apt remove package_name    # Debian
dnf remove package_name    # Fedora
```

### Update All Packages

```bash
apt update && apt upgrade    # Debian
dnf upgrade                  # Fedora
```

### Upgrade from a Local Package File

```bash
dpkg -i package.deb     # Debian
dnf upgrade package.rpm # Fedora
```

### List All Installed Packages

```bash
dpkg -l     # Debian
dnf list installed   # Fedora (cleaner)
rpm -qa     # Low-level Fedora
```

### Check if a Specific Package is Installed

```bash
dpkg -s package_name     # Debian
dnf list installed package_name  # Fedora
rpm -q package_name      # Low-level Fedora
```

### Show Package Information

```bash
apt show package_name     # Debian
dnf info package_name     # Fedora
```

### Find Which Package Owns a File

```bash
dpkg -S /path/to/file     # Debian
dnf provides /path/to/file # Fedora (modern)
rpm -qf /path/to/file     # Low-level Fedora
```
