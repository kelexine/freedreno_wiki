# Brain Corp bStem board

Information about freedreno on the [bStem](http://www.braincorporation.com/bstem-faq/) board.  See [[Fedora|Fedora]] for instructions on setting up setting up a fedora filesystem.

 * prebuilt kernel: [bstem-boot-f20.img](http://people.freedesktop.org/~robclark/f20/bstem-boot-f20.img)
 * kernel-msm branch: [bstem-drm](https://github.com/freedreno/kernel-msm/commits/bstem-drm)
 * defconfig: TODO

### Boot Instructions:

1. board ships with ubuntu.  The easiest way to force the board to come up in fastboot mode is: TBD
2. once board is booted to fastboot, if you want to prevent default ubuntu kernel from booting automatically: `sudo fastboot erase boot`
3. boot linux: `sudo fastboot boot bstem-boot-f20.img`

note that you can override the kernel commandline with fastboot, for example to boot with filesystem on sd-card:

    sudo fastboot -c "console=ttyHSL0,115200,n8 console=/dev/console lpj=67677 root=/dev/mmcblk1p3" boot bstem-boot-f20.img
