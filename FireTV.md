# Amazon Fire TV

Information about freedreno and linux on [Fire TV](http://en.wikipedia.org/wiki/Amazon_Fire_TV)

> NOTE: the firetv comes with no root or unlocked bootloader.  For the time being, you are on your own for that.

> UPDATE: how to get root: http://forum.xda-developers.com/showthread.php?t=2783805

> UPDATE: this is the CVE used to bypass the locked bootloader: https://www.codeaurora.org/projects/security-advisories/fastboot-boot-command-bypasses-signature-verification-cve-2014-4325
> (read the patches, not the description)

For now, I'm just collecting notes here so that I don't forget by the time an unlocker is more widely available.  Installing linux to boot off of the internal storage on firetv requires repartitioning internal storage.  Be careful, you can easily result in a bricked firetv.  If you don't understand what these commands, etc, are doing, you probably don't want to blindly follow them.

Note that I bootstrapped things by booting off of an existing usb disk with filesystem that I use with [[ifc6410|Ifc6410]]/[[bStem|bStem]]/etc.  You can probably get away with switching around the boot partitions when booted from internal disk, but you'll need too boot from usb disk at least once to merge userdata/cache/system partitions, and to extract new rootfs. 

Original partitions:

    [robclark@usbdisk:~]$ sudo gdisk -l /dev/mmcblk0
    GPT fdisk (gdisk) version 0.8.10
    
    Partition table scan:
      MBR: protective
      BSD: not present
      APM: not present
      GPT: present
    
    Found valid GPT with protective MBR; using GPT.
    Disk /dev/mmcblk0: 15269888 sectors, 7.3 GiB
    Logical sector size: 512 bytes
    Disk identifier (GUID): 98101B32-BBE2-4BF2-A06E-2BB33D000C20
    Partition table holds up to 20 entries
    First usable sector is 34, last usable sector is 15269854
    Partitions will be aligned on 2-sector boundaries
    Total free space is 0 sectors (0 bytes)
    
    Number  Start (sector)    End (sector)  Size       Code  Name
       1              34           16417   8.0 MiB     8300  persist
       2           16418           32801   8.0 MiB     8300  goldpersist
       3           32802           65569   16.0 MiB    0700  modem
       4           65570           65825   128.0 KiB   FFFF  sbl1
       5           65826           66337   256.0 KiB   FFFF  sbl2
       6           66338           67361   512.0 KiB   FFFF  sbl3
       7           67362           68385   512.0 KiB   FFFF  tz
       8           68386           69409   512.0 KiB   FFFF  rpm
       9           69410           71457   1024.0 KiB  FFFF  aboot
      10           71458           91937   10.0 MiB    FFFF  boot
      11           91938          112417   10.0 MiB    FFFF  recovery
      12          112418          177953   32.0 MiB    8300  diag
      13          177954          180513   1.2 MiB     8300  splash480p
      14          180514          186017   2.7 MiB     8300  splash720p
      15          186018          198305   6.0 MiB     8300  splash1080p
      16          198306          198321   8.0 KiB     FFFF  ssd
      17          198322          200369   1024.0 KiB  FFFF  misc
      18          200370         1773233   768.0 MiB   8300  system
      19         1773234         3346097   768.0 MiB   8300  cache
      20         3346098        15269854   5.7 GiB     8300  userdata

In case your partitions are different (pay attention to start/end sector, etc) you'll have to adjust the following commands accordingly.

### New boot partition

The default 10MiB boot/recovery partitions are not large enough for real linux, or in particular the initrd size.  For [[Fedora|Fedora]], a stripped down ramdisk (with unneeded kernel modules removed) is still about 12MiB on it's own.

Fortunately [lk](https://www.codeaurora.org/cgit/quic/la/kernel/lk/) seems to respect the partition names.  So I rename 'boot' to 'origboot' and 'diag' to boot.  Now we can use the larger diag partition for boot image.

    sgdisk -c 10:origboot /dev/mmcblk0
    sgdisk -c 12:boot /dev/mmcblk0

### New rootfs partition

If you never care about booting the original android from firetv again, you can delete the three userspace partitions and create a single larger partition (or separate boot/swap/root partitions if you prefer).

For this step, you need to *not* be booted from the partitions you are about to delete/overwrite.  Booting from an external USB disk is the recommended way.  Maybe someone can come up with a ramdisk with enough useful tools to avoid the temporary need for USB disk filesystem.

    sgdisk -d 18 /dev/mmcblk0
    sgdisk -d 19 /dev/mmcblk0
    sgdisk -d 20 /dev/mmcblk0
    sgdisk -n 18:200370:15269854 /dev/mmcblk0
    mkfs.ext4 -L rootfs /dev/mmcblk0p18

### Kernel

 * prebuilt kernel: TODO
 * kernel-msm branch: [firetv-drm](https://github.com/freedreno/kernel-msm/commits/firetv-drm)
 * defconfig: [bueller_rob_defconfig](https://github.com/freedreno/kernel-msm/blob/firetv-drm/arch/arm/configs/bueller_rob_defconfig)

### Audio

Note the q6.* firmware files seem to be different compared to [[ifc6410|Ifc6410]].  Copy the files from `/dev/mmcblk0p3` to `/lib/firmware`.

The needed alsa UCM files: [bueller-snd-card](http://people.freedesktop.org/~robclark/bueller-snd-card/)

### Serial Debug UART

If you have access to sufficient soldering skills/equipment (or know someone who does) then it is possible to solder up the [debug uart](http://imgur.com/JG9jwWC).  Note that it is 1.8V levels, and it is `console=ttyHSL2,115200,n8`.  (not `ttyHSL0` as on [[bStem|bStem]]/[[ifc6410|Ifc6410]].)

 