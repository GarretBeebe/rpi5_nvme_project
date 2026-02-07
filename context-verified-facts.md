# Context: Verified Facts (Do Not Re-Debug)

## SD Card
- SD card is healthy
- SD root boots reliably
- findmnt / → /dev/mmcblk0p2

## NVMe Hardware
- PCIe works (dtparam=pciex1 enabled)
- NVMe appears in lsblk when booted from SD
- NVMe appears in initramfs
- NVMe UUID verified multiple times

## Filesystem
- NVMe partition is ext4
- NVMe filesystem mounts cleanly from SD
- Root filesystem copy is complete and correct

## initramfs
- initramfs-tools installed
- initramfs generated successfully
- Pi 5 uses /boot/firmware/initramfs_2712
- config.txt explicitly references initramfs_2712
- initramfs is confirmed to be active at boot

## initramfs Diagnostics
From initramfs shell:
- nvme0n1 and nvme0n1p1 are present
- blkid shows correct UUID
- /proc/cmdline root UUID matches exactly

## Therefore NOT the problem
- Not bad hardware
- Not wrong UUID
- Not missing PCIe
- Not missing NVMe driver entirely
- Not missing initramfs
- Not wrong partition