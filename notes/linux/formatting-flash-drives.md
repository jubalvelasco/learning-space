# find your flash drive

get the name of last leaf of the tree, usually `sdb1`

**WARNING:** double-check the device name (size, model) before formatting, formatting the wrong device (e.g. your main disk) destroys its data.

```shell
lsblk
```

# unmount it

```shell
sudo umount /dev/sdb1
```

# format using FAT 32

```shell
sudo mkfs.vfat -F 32 /dev/sdb1
```

# format using NTFS

```shell
sudo mkfs.ntfs /dev/sdb1
```

# eject after

```shell
sudo eject /dev/sdb1
```
