# PyOS local package repository

This repository is used by the `pkg` package manager in the PyOS distro prototype.

The repo contains:
- `index.json` with the package catalog
- `packages/` with tar.gz archives for each package
- `rootfs/` with unpacked package installs
- `installed-packages.json` with the installed package database

This is a local, beginner-friendly package repository that simulates a real distro repo without requiring external network access.

## Package-manager features

The current `pkg` flow supports:
- `pkg list` for available packages
- `pkg info <name>` for package metadata
- `pkg search <term>` for searching the catalog
- `pkg install <name>` for dependency-aware installs
- `pkg remove <name>` to uninstall a package
- `pkg update` to refresh repo metadata
- `pkg verify <name>` to check archive checksum metadata
- `pkg upgrade [name]` to upgrade installed packages

This is the base layer for a real distro package system, and it can be extended later into a full APT/DNF-style repository manager.
