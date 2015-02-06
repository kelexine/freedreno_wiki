# Installing Fedora and Freedreno

These are the generic instructions for putting together a fedora userspace with freedreno, to use on some snapdragon board/device.  These instructions were originally on the [[Ifc6410|ifc6410]] wiki page, but apply for any device.  (I use the same filesystem for each of my arm boards, but with a device specific kernel/bootimg).

These instructions are to setup a Fedora filesystem on an external usb or sata disk.  The same basic procedure should work for installation to an micro-sd card.  These instructions are not intended for installation to eMMC.

### Fedora 19

* *`filesystem`*: [Fedora-armhfp-19-Beta-1-sda.raw.xz](https://dl.fedoraproject.org/pub/fedora-secondary/releases/test/19-Beta/Images/armhfp/Fedora-armhfp-19-Beta-1-sda.raw.xz) - see https://fedoraproject.org/wiki/Architectures/ARM/F19/Beta
* *`prebuilt`*: [ifc6410-freedreno-drm.tgz](http://people.freedesktop.org/~robclark/f19/ifc6410-freedreno-drm.tgz)

### Fedora 20

* *`filesystem`*: [Fedora-Desktop-armhfp-20-Beta-5-sda.raw.xz](http://download.fedoraproject.org/pub/fedora/linux/releases/test/20-Beta/Images/armhfp/Fedora-Desktop-armhfp-20-Beta-5-sda.raw.xz)
* *`prebuilt`*: [prebuilt-freedreno-f20.tgz](http://people.freedesktop.org/~robclark/f20/prebuilt-freedreno-f20.tgz)

NOTE: COPR available for F20 as alternative to prebuilt or building from src:

 http://blog.kwizart.fr/post/2014/03/02/163-mesa-10.2-from-git-for-Fedora-20

### Fedora 21

f21 should already have everything you need for mesa/libdrm/ddx.

### Rawhide

* *`filesystem`*: TODO (I used fedup to upgrade from f20)
* *`prebuilt`*: rawhide (as of May 2014) has recent enough mesa (10.2-rc4) and `xf86-video-freedreno` packaged already, so no prebuilt needed.  Just `yum install xorg-x11-drv-freedreno`.

### Instructions

1. Download Fedora filesystem (see links above)
2. If you haven't already, install adb, fastboot, abootimg.  If your host is fedora, you can `sudo yum install android-tools`
2. While you are waiting for the downloads, boot android which came on the ifc6410 board, and save off the firmware:
 * connect to device: `adb shell`
 * on device, `cd /system/etc; tar czvf /data/firmware.tgz firmware` (if you do not have tar, see [[Getting Busybox|FAQ#busybox]])
 * back on host: `adb pull /data/firmware.tgz`
3. on your host, <code>xzcat <em>filesystem</em> > /dev/sdN</code>  (ie, `/dev/sdb`) to extract filesystem to usb or sata disk that you will use 
4. Use gparted or similar tool to resize the 3rd partition (rootfs) to the remaining size of the disk.  Or optionally create a 4th partition for `/home`.
 * NOTE: I had to `sync`, then remove and re-plug my USB disk before using gparted to resize.
4. mount to rootfs partition, /dev/sdN3, and extract the firmware we saved earlier:
 * `cd <mountpath>/lib; tar xzvf path/to/firmware.tgz`
5. unmount rootfs, plug drive to board, and boot fedora:
 * if android still running: `sudo adb reboot-bootloader`
 * once board is booted to fastboot, if you want to prevent android from booting automatically: `sudo fastboot erase boot`
 * see device specific page for boot image and instructions: [[ifc6410|Ifc6410]], [[bStem|bStem]], [[apq8074 dragonboard|apq8074dragonboard]]
6. On board, login as root via serial terminal (initially no password).
7. Install X11 and gnome-desktop: `yum install xorg-x11-server-Xorg xorg-x11-drv-evdev @gnome-desktop`
8. xorg conf file, `/etc/X11/xorg.conf.d/90-freedreno.conf` (listed below)
9. If you don't feel like compiling libddrm/mesa/xf86-video-freedreno, then download the prebuilt and:
 * <code>cd / ; wget <em>prebuilt</em>; tar xzvf <em>prebuilt</em></code>
10. Optionally, other stuff you might want to install:
 * https://github.com/freedreno/nexus4-fedora/raw/master/rootfs/08-es2gears.tar.gz
 * https://github.com/freedreno/nexus4-fedora/raw/master/rootfs/05-term-resize.tar.gz
11. Create a user: `adduser johndoe`
12. Start gdm: `systemctl start gdm`

### Optional Installation
Additional things that I install for building freedreno (not required).

    yum install @development-tools automake autoconf xorg-x11-server-devel libX11-devel libXext-devel \
      libtool libXau-devel libXdamage-devel libXfixes-devel libXxf86vm-devel libxcb-devel pixman-devel \
      xorg-x11-proto-devel xorg-x11-util-macros gcc-c++ vim strace git tig htop xterm
    yum-builddep mesa-libGL mesa-libEGL

Also see [[Fedora-Games|Fedora Games]]

### xorg conf file:

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

### Compiling freedreno
See [[Build Instructions|Git-Trees-&-Branches#build-instructions]]

### Making NetworkManager behave
see [[FAQ|FAQ#wiki-making-networkmanager-behave]]
