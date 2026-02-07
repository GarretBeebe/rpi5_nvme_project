# Context: Current Root Cause Hypothesis

The most likely cause is that the initramfs environment
cannot successfully mount the NVMe ext4 filesystem as root.

Possible reasons include:
- ext4 support not loaded early enough in initramfs
- fsck blocking or failing during root mount
- initramfs module ordering issues
- Raspberry Pi 5–specific NVMe root edge case
- Kernel/initramfs bug with NVMe root on Pi 5

Evidence strongly suggests this is an early-boot filesystem mount issue,
not configuration or hardware.