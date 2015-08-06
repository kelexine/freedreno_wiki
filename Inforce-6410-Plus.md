# Inforce 6410 Plus

Information about freedreno on the [Inforce 6410 Plus](http://www.inforcecomputing.com/products/single-board-computers/6410-plus-single-board-computer-sbc) pico-itx board.  See [[Fedora|Fedora]] for instructions on setting up setting up a fedora filesystem.

NOTE: the below prebuilt kernel below is a bit old, and not getting updated.  It is recommended to see the linaro [kernel branch](https://git.linaro.org/landing-teams/working/qualcomm/kernel.git/shortlog/refs/heads/freedreno/ifc6410-v2.0-drm) and [Inforce 6410 Plus page](https://wiki.linaro.org/Boards/IFC6410).

 * prebuilt kernel: [ifc6410-boot-f20.img](http://people.freedesktop.org/~robclark/f20/ifc6410-boot-f20.img)
 * kernel-msm branch: [ifc6410-drm](https://github.com/freedreno/kernel-msm/commits/ifc6410-drm)
 * defconfig: [ifc6410_rob_defconfig](https://github.com/freedreno/kernel-msm/blob/ifc6410-drm/arch/arm/configs/ifc6410_rob_defconfig)

### Boot Instructions:

1. if android still running: `sudo adb reboot-bootloader`
2. once board is booted to fastboot, if you want to prevent android from booting automatically: `sudo fastboot erase boot`
  * alternatively, see [forcing Inforce 6410 Plus into fastboot](http://mydragonboard.org/2013/forcing-ifc6410-into-fastboot/)
3. boot linux: `sudo fastboot boot ifc6410-boot-f20.img`

note that you can override the kernel commandline with fastboot, for example to boot with filesystem on sd-card:

    sudo fastboot -c "console=ttyHSL0,115200,n8 console=/dev/console lpj=67677 root=/dev/mmcblk1p3" boot ifc6410-boot-f20.img

### Resizing boot partition

To boot from eMMC (without fastboot), a larger boot partition is needed.  The following might work on other devices/boards too, but be very careful to check the partition #'s, and that there is nothing important in the partitions following the boot partition.  On the ifc6410 the boot partition is `mmcblk0p7`.  The following will wipe out the android userspace partitions.

    sgdisk -d 7 /dev/mmcblk0
    sgdisk -n 7:393216:524287 /dev/mmcblk0
    sgdisk -c 7:boot /dev/mmcblk0
    sgdisk -A 7:set:60 /dev/mmcblk0
    sgdisk -t 7:ea00 /dev/mmcblk0

after resizing mmcblk0p7 you can `fastboot flash boot ifc6410-boot-f20.img`

### making a serial cable:
The debug UART (`console=ttyHSL0,115200,n8`) is the three pin header next to the DC power connector. RX/GND/TX are layed out as show below. These can be connected to a standard serial cable, pins 2, 3, and 5 on an [rs232 db9 connector](http://www.arcelect.com/9_PIN_PIN_OUT.GIF).

![Serial layout](http://people.collabora.com/~sjoerd/ifc6410-serial.png)

UPDATE: [UART cable instructions with pictures](http://mydragonboard.org/2013/rs-232-cable-for-ifc6410/)

NOTE: that it seems the RX/TX are swapped on some board revisions.  As far as I know GND is always the center pin.  If the above layout does not work, try plugging in the serial cable the other way around.

### Upstream kernel
Work on getting upstream kernel on ifc6410 is tracked at the [linaro wiki](https://wiki.linaro.org/Boards/IFC6410/LinuxKernel).
