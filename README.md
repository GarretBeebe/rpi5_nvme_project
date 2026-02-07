# Raspberry Pi 5 NVMe Root Migration

Documentation and troubleshooting notes for migrating a Raspberry Pi 5 root filesystem from SD card to NVMe SSD.

## Goal

Boot the Pi 5 with NVMe as the root filesystem (`/`), using the SD card only for firmware and boot files.

## Current Status

🔴 **NVMe root boot not working** — System drops to initramfs shell

The NVMe is detected and the UUID is correct, but the ext4 mount as root fails. This appears to be an early-boot/initramfs issue, not a hardware or configuration problem.

✅ **SD card boot works reliably** — Always recoverable

## Hardware

- Raspberry Pi 5 (BCM2712)
- PCIe NVMe HAT
- NVMe SSD (healthy, detected consistently)
- SD card (boot + fallback root)

## Documentation

| File | Description |
|------|-------------|
| `context-overview.md` | Goal, current state, status |
| `context-hardware-software.md` | Hardware specs and software versions |
| `context-verified-facts.md` | Confirmed facts — don't re-test these |
| `context-failure-characterization.md` | How the failure manifests |
| `context-current-cause-hypothesis.md` | Working theory on root cause |
| `context-paths-forward.md` | Options for resolution |
| `context-recovery-procedure.md` | How to recover from failed boot |
| `mynotes.txt` | Raw config snippets (UUIDs, fstab, cmdline) |

## Key Finding

The problem is **not**:
- Bad hardware
- Wrong UUID
- Missing PCIe/NVMe drivers
- Missing initramfs
- Wrong partition

The problem **is**:
- initramfs fails to mount NVMe ext4 as root
- Likely ext4 module loading, fsck, or Pi 5-specific edge case

## Quick Reference

```bash
# NVMe UUID
a3c4f52c-1878-4038-bcaf-281cfda51100

# Kernel version
6.12.62+rpt-rpi-2712

# initramfs filename (Pi 5)
/boot/firmware/initramfs_2712
```

## Recovery

If boot fails, the system is always recoverable:

1. Power off
2. Remove NVMe (or edit cmdline.txt from another machine)
3. Boot from SD card

See `context-recovery-procedure.md` for details.
