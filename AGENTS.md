# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This repository documents troubleshooting efforts to migrate a Raspberry Pi 5 root filesystem from SD card to NVMe SSD. The goal is to boot from NVMe while using the SD card only for firmware/boot.

## Documentation Structure

Context is split across multiple `context-*.md` files to avoid information overload:

- `context-overview.md` — Goal, current state, and status
- `context-hardware-software.md` — Hardware specs (Pi 5, NVMe HAT) and software versions
- `context-verified-facts.md` — Facts that have been confirmed and should NOT be re-debugged
- `context-failure-characterization.md` — How the failure manifests (initramfs drop, not detection failure)
- `context-current-cause-hypothesis.md` — Working theory on root cause
- `context-paths-forward.md` — Options A/B/C for resolution
- `context-recovery-procedure.md` — How to recover if boot fails
- `mynotes.txt` — Raw configuration snippets (UUIDs, fstab entries, cmdline.txt)

## Key Technical Context

### RPi 5 Boot Architecture
- Pi 5 uses kernel suffix `_2712` (BCM2712 SoC)
- initramfs filename: `/boot/firmware/initramfs_2712`
- Firmware lives on SD at `/boot/firmware`

### The Problem
The NVMe device IS detected in initramfs (blkid shows correct UUID), but mounting it as ext4 root fails. This is a **mount failure**, not a detection failure.

### What Has Been Ruled Out
- Hardware issues
- UUID mismatches
- Missing PCIe/NVMe drivers
- Missing initramfs
- Wrong partition

## Working With This Repository

When assisting with this troubleshooting:

1. **Read `context-verified-facts.md` first** — Avoid suggesting things already tested
2. **Check `context-current-cause-hypothesis.md`** for the working theory
3. **Refer to `context-paths-forward.md`** for the decision tree on next steps
4. Keep suggestions focused on early-boot/initramfs/ext4 mount issues
5. The system is always recoverable via SD card — see `context-recovery-procedure.md`

## Useful Commands (Run on the Pi)

```bash
# Check current root device
findmnt /

# List block devices
lsblk

# Show UUIDs
blkid

# Check kernel version
uname -r

# Regenerate initramfs (Pi 5)
sudo update-initramfs -u -k all

# View boot config
cat /boot/firmware/config.txt
cat /boot/firmware/cmdline.txt
```
