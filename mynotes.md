# /etc/fstab entry for mounting the nvvme
```
    UUID=a3c4f52c-1878-4038-bcaf-281cfda51100  /mnt/nvme  ext4  noatime,commit=60  0  2
```
# /etc/fstab entry for mounting root from the nvme
```    
    UUID=a3c4f52c-1878-4038-bcaf-281cfda51100  /  ext4  noatime,commit=120  0  1
```
# cmdline.txt entry pointing at the uuid for the nvme
```
    console=serial0,115200 console=tty1 root=UUID=a3c4f52c-1878-4038-bcaf-281cfda51100 rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles cfg80211.ieee80211_regdom=US
```
# OS
```
    6.12.62+rpt-rpi-2712
```
# Simplified cmdline.txt pointing at nvme
```    
    root=/dev/nvme0n1p1 rootfstype=ext4 rootwait rootdelay=20 fsck.repair=yes
```    