# Installing Arch Linux on Ifc6410

(*these instructions are untested on the Ifc6410 and were modified from the nexus7 instructions.)

Since Arch Linux Arm does not distribute an Ifc6410 release a slight work
around is needed. Download for example
[[ArchLinuxArm|http://archlinuxarm.org/os/ArchLinuxARM-trimslice-latest.tar.gz]] (or you can
download another armv7 one, it's just that this one only has 1 partition and is
easier to use) and extract to a usb or sata (/dev/sdX1)

* Download a working kernel from
[[ifc6410-boot-f20.img|http://people.freedesktop.org/~robclark/f20/ifc6410-boot-f20.img]] or build your
own from source and make it into an andoid boot.img (some information about
Arch boot.img can be found here [[build-initramfs-mkinitcpio|https://github.com/crondog/arch-flo#build-initramfs-mkinitcpio]])

* Plug the usb/sata drive into the Ifc6410 and connect to your computer

* Boot the kernel via fastboot with "fastboot boot ifc6410-boot-f20.img"

* Once booted, you will need to remove device specific packages such as

  * linux-headers-* 
  * linux-*
  * nvidia-trimslice etc

### Graphics
* xf86-video-freedreno can be found at [[xf86-video-freedreno-git|https://aur.archlinux.org/packages/xf86-video-freedreno-git/]]
* libdrm and be installed via pacman -S libdrm
* Mesa will need to be built. A PKGBUILD can be found here
[[PKGBUILD|https://github.com/crondog/arch-flo/blob/master/PKGBUILD.mesa]] (I would
recommend running distcc on the board and 
[[distcc-alarm|https://github.com/WarheadsSE/PKGs/tree/master/distccd-alarm]] on the host for 
faster builds)

### Xorg.conf

    Section "Device"
            Identifier      "Video Device"
            Driver          "freedreno"
            # Uncomment for addition debug traces in xorg log:
            #Option          "Debug"           "true"
            # The below two options are not needed if you are using the
            # msm drm/kms driver:
            #Option          "fb"              "/dev/fb0"
            #Option           "SWCursor"        "true"
    EndSection
    Section "Screen"
            Identifier      "Screen"
            Monitor         "Monitor"
            Device          "Video Device"
    EndSection