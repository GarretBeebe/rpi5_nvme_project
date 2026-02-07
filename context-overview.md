# Context: Raspberry Pi 5 NVMe Root Troubleshooting

## Goal
Migrate Raspberry Pi 5 root filesystem from SD card to NVMe for performance and reliability.
Use SD card only for firmware/boot.

## Current State
- SD card boots reliably
- NVMe hardware works and is detected
- NVMe filesystem is valid and mountable when booted from SD
- NVMe root boot consistently fails, dropping to initramfs

## Key Insight
The system can see the NVMe device and correct UUID in initramfs,
but fails when attempting to mount NVMe ext4 as root.

This points to an early-boot / initramfs / ext4 mount issue,
not hardware, UUIDs, or configuration mistakes.

## Status
System is always recoverable.
NVMe root attempts are currently paused.