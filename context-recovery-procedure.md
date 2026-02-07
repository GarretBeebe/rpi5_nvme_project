# Context: Guaranteed Recovery Procedure

If the system drops to initramfs:

1. Power off
2. Remove NVMe
3. Boot from SD
4. If needed, fix SD offline:
   - Set root=/dev/mmcblk0p2 in cmdline.txt
   - Remove NVMe entries from /etc/fstab
5. System always returns to SD boot

No data loss has occurred during any failure.