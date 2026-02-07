# Context: Failure Characterization

## Symptom
When root=UUID=<NVMe> is set:
- Boot drops to initramfs shell
- Happens even though NVMe device and UUID are visible

## Interpretation
initramfs is running and working,
but fails when attempting to mount NVMe ext4 as root.

## Key Distinction
- Device discovery succeeds
- Filesystem mount as root fails

This is an early-boot mount failure, not a detection failure.