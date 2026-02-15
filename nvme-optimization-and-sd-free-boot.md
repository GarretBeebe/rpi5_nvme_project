# NVMe Optimization & SD-Free Boot Guide

This document provides optimization steps for the NVMe-based system and a path to eliminate the SD card from the boot process entirely.

---

## Part 1: NVMe Performance Optimization

Now that root is on NVMe, these optimizations will maximize performance and longevity.

### 1.1 Verify Current Mount Options

**Checkpoint:** Confirm NVMe is root and check current mount options.

```bash
# Verify NVMe is root
findmnt /

# Expected output shows /dev/nvme0n1p1 or UUID as source
```

### 1.2 Optimize fstab Mount Options

Update `/etc/fstab` for optimal NVMe performance:

```bash
# Backup current fstab
sudo cp /etc/fstab /etc/fstab.backup.$(date +%Y%m%d)

# View current entry
grep nvme /etc/fstab
```

**Recommended mount options for NVMe root:**

```
UUID=a3c4f52c-1878-4038-bcaf-281cfda51100  /  ext4  defaults,noatime,discard,commit=120  0  1
```

| Option | Benefit |
|--------|---------|
| `noatime` | Reduces writes by not updating access times |
| `discard` | Enables TRIM for SSD longevity |
| `commit=120` | Reduces journal commits (less write amplification) |

**Apply changes:**

```bash
# Edit fstab
sudo nano /etc/fstab

# Test fstab syntax (CRITICAL before reboot)
sudo findmnt --verify

# Remount to apply without reboot
sudo mount -o remount /
```

**Rollback:** If issues occur, boot from SD and restore:
```bash
sudo cp /mnt/nvme/etc/fstab.backup.* /mnt/nvme/etc/fstab
```

### 1.3 Enable TRIM Support

Verify TRIM is working:

```bash
# Check if TRIM is supported
sudo fstrim -v /

# Expected: reports bytes trimmed
```

Set up periodic TRIM (weekly):

```bash
# Enable fstrim timer
sudo systemctl enable fstrim.timer
sudo systemctl start fstrim.timer

# Verify it's active
systemctl status fstrim.timer
```

### 1.4 Optimize I/O Scheduler

NVMe works best with `none` or `mq-deadline` scheduler:

```bash
# Check current scheduler
cat /sys/block/nvme0n1/queue/scheduler

# Expected: [none] or [mq-deadline]
```

If not optimal, make it permanent:

```bash
# Create udev rule
echo 'ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="none"' | \
  sudo tee /etc/udev/rules.d/60-nvme-scheduler.rules

# Reload udev
sudo udevadm control --reload-rules
sudo udevadm trigger
```

### 1.5 Move High-I/O Directories to NVMe

If using hybrid boot (SD firmware + NVMe root), ensure these are on NVMe:

```bash
# Verify these are on NVMe (should show nvme as device)
df -h /var/log
df -h /tmp
df -h /var/lib/docker  # if using Docker
```

### 1.6 Reduce SD Card Wear (Hybrid Setup)

If SD card still holds `/boot/firmware`:

```bash
# Make boot partition read-only after config changes complete
# Add 'ro' to /boot/firmware mount in fstab (optional)
```

---

## Part 2: Eliminate SD Card from Boot (Full NVMe Boot)

The Raspberry Pi 5 can boot entirely from NVMe using the onboard EEPROM. This removes the SD card requirement.

### Prerequisites

- Working NVMe root filesystem (verified in Part 1)
- NVMe contains complete bootable system
- Current EEPROM firmware (2024 or later recommended)

### 2.1 Check EEPROM Version

```bash
# Check current EEPROM version
sudo rpi-eeprom-update

# Update if needed (requires reboot)
sudo rpi-eeprom-update -a
sudo reboot
```

**Checkpoint:** EEPROM should be 2024-xx-xx or later for reliable NVMe boot.

### 2.2 Create Boot Partition on NVMe

The NVMe needs a FAT32 boot partition for firmware. Two approaches:

#### Option A: Add Boot Partition to Existing NVMe (Recommended)

This requires resizing the existing partition. **BACKUP FIRST.**

```bash
# Backup critical data
sudo rsync -aAXv / /mnt/backup/ --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*"}
```

If your NVMe has unallocated space:

```bash
# Check partition layout
sudo fdisk -l /dev/nvme0n1

# Use fdisk/parted to create a ~512MB FAT32 partition as partition 1
# Move root to partition 2 if needed
```

#### Option B: Fresh NVMe Setup with Proper Partitioning

Use Raspberry Pi Imager to write a fresh image directly to NVMe, then restore data.

**Recommended partition layout:**

| Partition | Size | Type | Mount |
|-----------|------|------|-------|
| nvme0n1p1 | 512 MB | FAT32 | /boot/firmware |
| nvme0n1p2 | Remainder | ext4 | / |

### 2.3 Copy Boot Firmware to NVMe

Once NVMe has a FAT32 boot partition:

```bash
# Mount NVMe boot partition
sudo mkdir -p /mnt/nvme-boot
sudo mount /dev/nvme0n1p1 /mnt/nvme-boot

# Copy boot firmware from SD
sudo rsync -av /boot/firmware/ /mnt/nvme-boot/

# Unmount
sudo umount /mnt/nvme-boot
```

### 2.4 Update cmdline.txt on NVMe Boot Partition

Ensure cmdline.txt points to NVMe root:

```bash
# Mount NVMe boot
sudo mount /dev/nvme0n1p1 /mnt/nvme-boot

# Edit cmdline.txt
sudo nano /mnt/nvme-boot/cmdline.txt
```

**Content should be:**

```
console=serial0,115200 console=tty1 root=PARTUUID=<nvme-root-partuuid> rootfstype=ext4 fsck.repair=yes rootwait
```

Get the PARTUUID:

```bash
blkid /dev/nvme0n1p2 | grep -oP 'PARTUUID="\K[^"]+'
```

### 2.5 Update fstab on NVMe Root

Ensure `/etc/fstab` on the NVMe root partition references NVMe boot:

```bash
# Mount NVMe root if not already
sudo mount /dev/nvme0n1p2 /mnt/nvme

# Edit fstab
sudo nano /mnt/nvme/etc/fstab
```

**Required entries:**

```
PARTUUID=<nvme-boot-partuuid>  /boot/firmware  vfat  defaults  0  2
PARTUUID=<nvme-root-partuuid>  /               ext4  defaults,noatime,discard  0  1
```

### 2.6 Configure EEPROM Boot Order

**CRITICAL STEP** — Tell the Pi 5 to boot from NVMe first.

```bash
# Edit EEPROM config
sudo rpi-eeprom-config --edit
```

**Change BOOT_ORDER:**

```
[all]
BOOT_UART=1
BOOT_ORDER=0xf416
```

| Code | Device |
|------|--------|
| 6 | NVMe |
| 4 | USB |
| 1 | SD Card |
| f | Loop/retry |

`0xf416` means: Try NVMe → USB → SD → retry.

**Apply the change:**

```bash
# Write new config (happens on next reboot)
sudo reboot
```

### 2.7 Test NVMe-Only Boot

**Checkpoint sequence:**

1. **First test WITH SD card inserted:**
   ```bash
   # After reboot, verify booting from NVMe
   findmnt /
   findmnt /boot/firmware
   
   # Both should show nvme0n1p* devices
   ```

2. **Second test WITHOUT SD card:**
   - Power off
   - Remove SD card
   - Power on
   - System should boot from NVMe

3. **Verify boot source:**
   ```bash
   # Check boot device
   cat /proc/cmdline
   
   # Should show root=PARTUUID pointing to NVMe
   ```

### 2.8 Rollback Procedure

If NVMe boot fails:

#### Immediate Recovery (with SD Card)

1. Power off
2. Insert SD card
3. Power on — Pi will fall back to SD boot
4. Fix NVMe configuration from SD environment

#### Reset EEPROM Boot Order to SD-First

```bash
# Boot from SD, then:
sudo rpi-eeprom-config --edit

# Change to SD-first
BOOT_ORDER=0xf41

sudo reboot
```

#### Factory Reset EEPROM (Nuclear Option)

```bash
# Download recovery image from raspberrypi.com
# Write to SD card
# Boot Pi with recovery SD
# EEPROM will be reset to factory defaults
```

---

## Part 3: Post-Migration Verification Checklist

Run these checks after completing migration:

```bash
# System is booting from NVMe
echo "=== Boot Source ==="
findmnt /
findmnt /boot/firmware

# NVMe performance is reasonable
echo "=== NVMe Performance ==="
sudo hdparm -t /dev/nvme0n1

# TRIM is working
echo "=== TRIM Status ==="
sudo fstrim -v /

# No critical errors in journal
echo "=== Recent Boot Errors ==="
journalctl -b -p err

# Disk health
echo "=== NVMe Health ==="
sudo smartctl -a /dev/nvme0n1 | grep -E "(Critical|Temperature|Percentage)"
```

---

## Quick Reference: Key Files

| File | Purpose |
|------|---------|
| `/etc/fstab` | Filesystem mount configuration |
| `/boot/firmware/cmdline.txt` | Kernel boot parameters |
| `/boot/firmware/config.txt` | Pi hardware configuration |
| EEPROM config | Boot device order |

---

## Decision Matrix

| Goal | Recommended Path |
|------|------------------|
| Maximum performance, keep SD as fallback | Part 1 only |
| Eliminate SD completely | Part 1 + Part 2 |
| Safest approach | Part 1, skip Part 2, keep SD |

---

## Summary of Commands by Phase

### Phase 1: Optimize Performance
```bash
sudo cp /etc/fstab /etc/fstab.backup.$(date +%Y%m%d)
sudo systemctl enable --now fstrim.timer
sudo findmnt --verify
```

### Phase 2: Check EEPROM
```bash
sudo rpi-eeprom-update
sudo rpi-eeprom-config --edit
```

### Phase 3: Full NVMe Boot
```bash
# Get PARTUUIDs
blkid /dev/nvme0n1p1
blkid /dev/nvme0n1p2

# Update BOOT_ORDER in EEPROM
sudo rpi-eeprom-config --edit
# Set BOOT_ORDER=0xf416
```

### Emergency Recovery
```bash
# From SD boot, reset EEPROM
sudo rpi-eeprom-config --edit
# Set BOOT_ORDER=0xf41
```
