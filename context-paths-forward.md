# Context: Paths Forward

## Option A — Continue NVMe Root Debugging
- Force ext4 + NVMe modules into initramfs
- fsck NVMe filesystem
- Recreate filesystem with conservative options
- Retry NVMe root one final time

Pros: Achieves original goal  
Cons: High complexity, diminishing returns

## Option B — Hybrid (Most Practical)
- Keep root on SD
- Move Docker, Nextcloud, databases to NVMe
- SD handles boot + low I/O only

Pros: Huge performance win, very stable  
Cons: Root technically still on SD

## Option C — Fresh NVMe OS Install
- Install Raspberry Pi OS directly to NVMe
- SD used only for firmware

Pros: Cleanest NVMe-root outcome  
Cons: Requires reinstall / migration