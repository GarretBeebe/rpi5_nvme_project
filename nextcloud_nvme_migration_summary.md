# Raspberry Pi 5 NVMe Migration & Nextcloud Performance Optimization Summary

## Overview

This document summarizes the complete migration from SD-card boot to
NVMe-only boot on a Raspberry Pi 5, followed by full-stack performance
optimization of a Docker-based Nextcloud deployment.

The system now operates entirely from NVMe storage with production-grade
tuning applied across:

-   Bootloader & partition layout
-   Filesystem configuration
-   Linux VM tuning
-   Docker stack
-   Postgres database
-   Redis caching
-   PHP Opcache
-   Nextcloud background jobs

------------------------------------------------------------------------

# Phase 1 --- NVMe-Only Boot Migration

## Initial State

-   Root filesystem on NVMe
-   Boot partition on SD card
-   Mixed boot dependency
-   Risk of SD-related instability

## Actions Performed

### 1. Shrunk NVMe root partition safely

-   Ran filesystem check (`e2fsck`)
-   Resized ext4 filesystem
-   Adjusted GPT partition boundary
-   Freed \~1GB for dedicated boot partition

### 2. Created NVMe Boot Partition

-   Added FAT32 partition (`nvme0n1p2`)
-   Formatted with `mkfs.vfat`
-   Copied boot firmware from SD to NVMe
-   Verified kernel, initramfs, overlays copied correctly

### 3. Updated Configuration to Use UUIDs

-   `/etc/fstab` updated:
    -   Root mounted via UUID
    -   `/boot/firmware` mounted via NVMe UUID
-   `cmdline.txt` updated to:
    -   `root=UUID=<nvme-root-uuid>`

### 4. Verified Boot Chain

-   Confirmed:
    -   `/` → nvme0n1p1
    -   `/boot/firmware` → nvme0n1p2
-   Removed SD card
-   Confirmed successful NVMe-only boot

## Result

✔ Fully SD-independent system\
✔ Clean GPT layout\
✔ Stable UUID-based mounting\
✔ Production-safe boot configuration

------------------------------------------------------------------------

# Phase 2 --- System-Level NVMe Performance Tuning

## Filesystem & IO

-   Confirmed NVMe scheduler: `none`
-   Enabled weekly TRIM (`fstrim.timer`)
-   Root mounted with `noatime`

## Linux VM Tuning

Applied:

    vm.swappiness = 10
    vm.dirty_background_ratio = 5
    vm.dirty_ratio = 15

Benefits: - Reduced swap usage - Smoother writeback behavior - Reduced
IO spikes - Improved database stability

Persisted in:

    /etc/sysctl.d/99-nvme-tuning.conf

------------------------------------------------------------------------

# Phase 3 --- Nextcloud Stack Optimization

Docker-based services:

-   nextcloud:32.0.6
-   postgres:15-bookworm
-   redis:latest
-   portainer
-   watchtower

------------------------------------------------------------------------

## Redis Configuration

Verified: - Redis used for locking - Redis used for distributed cache -
APCu used for local cache

Confirmed via `occ` commands.

------------------------------------------------------------------------

## Background Jobs

-   Switched from AJAX mode to CRON mode
-   Host cron executes every 5 minutes:

```{=html}
<!-- -->
```
    docker exec -u www-data nextcloud php /var/www/html/cron.php

Result: ✔ Reliable background task execution\
✔ Faster file scanning\
✔ Improved preview generation\
✔ Better cleanup behavior

------------------------------------------------------------------------

## Postgres Performance Tuning

Original defaults were conservative.

Applied via `ALTER SYSTEM`:

    shared_buffers = 1GB
    effective_cache_size = 5GB
    work_mem = 16MB
    maintenance_work_mem = 256MB

Restarted container to activate `shared_buffers`.

Result: ✔ Faster query planning\
✔ Reduced disk IO\
✔ Better memory utilization\
✔ Improved Nextcloud DB responsiveness

------------------------------------------------------------------------

## PHP Opcache Optimization

Created persistent override file:

    zzz-opcache-tuning.ini

Applied:

    opcache.memory_consumption = 256
    opcache.interned_strings_buffer = 64
    opcache.max_accelerated_files = 20000

Mounted via docker-compose bind mount.

Result: ✔ Reduced opcode cache eviction\
✔ Faster app load times\
✔ Lower CPU churn under concurrency

------------------------------------------------------------------------

# Final System State

### Storage

-   NVMe-only boot
-   Dedicated boot partition
-   TRIM enabled
-   Clean mount structure
-   No SD dependency

### Memory & IO

-   Optimized writeback behavior
-   Reduced swap pressure
-   Stable NVMe scheduling

### Database

-   Postgres tuned for 8GB system
-   Increased buffer efficiency
-   Better vacuum & maintenance behavior

### Caching

-   Redis active
-   APCu active
-   Cron active
-   Opcache expanded

------------------------------------------------------------------------

# Outcome

The Raspberry Pi 5 now runs a production-grade, NVMe-optimized Nextcloud
deployment with:

-   High stability
-   Improved IO consistency
-   Reduced latency
-   Better concurrency handling
-   Clean boot architecture
-   No SD bottleneck

This configuration exceeds many default cloud VPS deployments in tuning
quality.

------------------------------------------------------------------------

End of summary.
