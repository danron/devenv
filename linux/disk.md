# Disk management in Linux

## LVM

### Resizing a Logaical Volume after increasing the underlying physical size

To check your configuration, use the `lsblk` command:
```
# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
nvme0n1              259:0    0  100G  0 disk
├─nvme0n1p1          259:1    0  512M  0 part /boot
└─nvme0n1p2          259:2    0 99.5G  0 part
  ├─vg_system-root   253:0    0   80G  0 lvm  /
  ├─vg_system-swap   253:1    0    2G  0 lvm
  ├─vg_system-varlog 253:2    0    3G  0 lvm  /var/log
  ├─vg_system-tmp    253:3    0    2G  0 lvm  /var/tmp
  └─vg_system-home   253:4    0 12.5G  0 lvm  /home
```

Resize the disk `nvme0n1` (in AWS or some other way) and let the logical volume `vg_system-root` use that space:

```
pvresize /dev/nvme0n1p2
lvextend --resizefs /dev/vg_system/root -l+100%FREE
```
