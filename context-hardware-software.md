# Context: Hardware and Software Environment

## Hardware
- Raspberry Pi 5 (BCM2712)
- SD card (boot + fallback root)
- PCIe NVMe HAT
- NVMe SSD (healthy, detected consistently)

## Software
- Raspberry Pi OS
- Kernel: 6.12.62+rpt-rpi-2712
- initramfs-tools installed manually
- Firmware uses /boot/firmware

## Boot Architecture
- SD card contains firmware + boot config
- NVMe intended to be root (/)
- initramfs filename on Pi 5: initramfs_2712