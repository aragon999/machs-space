---
title: "Ext Btrfs Conversion"
date: 2023-xx-xxT00:00:00+02:00
draft: true
---

Recently I again needed (or rather wanted) to convert a root `ext4` file system to a root `btrfs` file system and replace the SSD with a bigger and faster one. Since I have done this several times in the past but always needed to lookup the specifics, this time I wrote some notes for myself. I added some more explanations and links and now have this as first "real" blog post. Note that I kept my boot partition on the old hard drive, if that does not work for you, copy that partition to the new hard drive as well and make sure that it is on the first blocks of the new device.

I will not go into why I use `btrfs`, this blog post will only be about the conversion, it may become a future blog post though. The guide is written for and using [Arch Linux](https://archlinux.org/). Probably all other distributions, which provide similar tools and a kernel which supports all needed features, could follow this guide, maybe some details differ. Furthermore I only tested this guide with `ext4`, but it should work for `ext{2,3}` as well.

Throughout the guide I will assume that the source and target devices are available on the same machine. In this guide the source device is `/dev/sdY` and the target device `/dev/sdX`. The source partition is `/dev/sdYj` and the target partition `/dev/sdXi`.

# Preparations
So let us start with some preparations, which still can be done when booted into the source hard drive, but the target hard drive should already be available.

For the sake of completeness and better be save than sorry, [make sure](https://man7.org/linux/man-pages/man8/e2fsck.8.html) that the source partition has no problems
```
# e2fsck -f /dev/sdYj
```

Next we want to prepare the target device by creating a partition on. This partition should be larger (if you want to be save make it twice as large) as the partition you want to convert from. You can shrink the partition later if you want. In order to do so we will use the appropriate partitioning tool
```
# [f,g]disk /dev/sdX
```
i.e. `fdisk` for the old [MBR](https://en.wikipedia.org/wiki/Master_boot_record) and `gdisk` for the newer [GUID](https://en.wikipedia.org/wiki/GUID_Partition_Table) partition tables.

To create a new partition table use `g` in `fdisk` or `o` in `gdisk` to create a new GPT-partition table and create a Linux partition using `n`. Finally write the new partition table onto the device `w` and quit `q` in case of `fdisk`.

The last thing we need is a live bootable Arch Linux USB stick. When copying the data, the source partition should not be mounted. So grab an USB stick and [create a USB installation medium](https://wiki.archlinux.org/title/USB_flash_installation_medium#In_GNU/Linux) if you do not have it anyway: https://archlinux.org/download/

# Clone existing partition
Now reboot into the live Linux and copy partition block-wise using [`dd(5)`](https://man7.org/linux/man-pages/man1/dd.1.html)
```
# dd if=/dev/sdYj of=/dev/sdXi bs=1M conv=fsync oflag=direct status=progress
```
Note that after this any changes you perform on the old partition, will not be mirrored to the new one.

Now make sure that everything copied is still correct in the sense of `ext`
```
# e2fsck -f /dev/sdXi
```

And correct the file system alignments to the created partition
```
# resize2fs /dev/sdXi
```

# Converting to `btrfs`
This section can be done either when booted into the old linux, or from the live USB medium. But now we actually do the conversion of the partition (might take a while, but you can use your system normally), note if you are in the live USB stick, make sure to run
```

```
if you are in EFI mode.

Next convert the partition using [`btrfs-convert(8)`](https://man7.org/linux/man-pages/man8/btrfs-convert.8.html)
```
# btrfs-convert /dev/sdXi
```
and [check](https://man7.org/linux/man-pages/man8/btrfs-check.8.html) that there were no errors
```
# btrfs check /dev/sdXi
```
and that all checksums of the data are actually correct:
```
# btrfs check --check-data-csum /dev/sdXi
```

Create a temporary directory, and save it in a variable
```
$ TMP_NEW_ROOT="$(mktemp -d)"
```

Mount btrfs file system
```
# mount -t btrfs /dev/sdXi "${TMP_NEW_ROOT}"
```

Chroot into the new system, by using [arch-chroot(8)](https://man.archlinux.org/man/arch-chroot.8) which mounts all required (virtual) filesystems in the `chroot` environment
```
# arch-chroot "${TMP_NEW_ROOT}"
```

Inside the chroot environment, regenerate the initramfs
```
# sudo mkinitcpio -p linux
```
and finally change the "/etc/fstab" file to mount your new partition (make sure to change the last field, `fs_passno` to 0).

`exit` the chroot environment finally change the bootloader to the new partition. In my case this is `systemd-boot`, where I needed to change the `PARTUUID` in `/boot/loader/entries/arch.conf`. Get the `PARTUUID` or the `UUID` using `$ blkid`.

Reboot and you should be booting from you `btrfs` file system.

# Cleanup
If everything worked, cleanup the created subvolume and rebalance the btrfs filesystem:
```
# btrfs subvolume delete /ext2_saved
# btrfs balance start --bg /
```
where the status of the balance operation can be checked using
```
# btrfs balance status /
```

# Resources and further reading:
- https://wiki.archlinux.org/title/btrfs
- https://wiki.archlinux.org/title/systemd-boot
- `man 5 btrfs`, `man 8 btrfs` and `man 8 btrfs-*`
