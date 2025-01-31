##### Table of Contents
* [Root password](#rootpasswd)
* [Getting Kernel Traces](#kerneltraces)
* [err: request_firmware for vidc_1080p.fw error -2](#vidcfwerr)
* [Making NetworkManager behave](#networkmanager)
* [Getting Busybox (tar, etc)](#busybox)
* [Kernel Debugging Options](#kerneldbgopt)
* [Prebuilt userspace vs system updates](#prebuiltupdate)


<a name="rootpasswd"/>

### Root password
I'm not actually sure, it seems to have changed since earlier fedora arm filesystems.  (If someone knows, please edit this ;-))

What I do after creating a new filesystem is just to mount the disk on my laptop, and edit `$mountpoint/etc/shadow` and change root to:

    root::15921:0:99999:7:::

(ie. remove what is between the first and second `:`)

There may of course be other better ways, but that is pretty simple and works for me.

<a name="kerneltraces"/>

### Getting Kernel Traces
If you don't have a debug UART (for example on a form factor device like phone or tablet), you have a couple options.  If the device does not reboot before you have a chance to dump traces:
```
dmesg > dmesg.txt
```
I frequently configure the kernel for `CONFIG_LOG_BUF_SHIFT=21` to increase the kernel trace buffer size, to avoid loosing traces.

If the device does reboot before you have a chance to collect traces, then the android ram console (`CONFIG_ANDROID_RAM_CONSOLE=y`) is useful.  After reboot, you can get previous boot's traces by:
```
cat /proc/last_kmsg > dmesg.txt
```
(If needed, switch back to a known good kernel to collect traces)

<a name="vidcfwerr"/>

### err: request_firmware for vidc_1080p.fw error -2
The vidc driver (hw accel video encode/decode... nothing to do w/ 2d/3d/freedreno) in msm android kernel is particularly badly behaved when it can't load it's firmware.  If the driver is compiled into the kernel (not built as a module), it needs it's firmware before the root filesystem is mounted.  You either need to put vidc_1080p.fw in the initrd, or disable vidc if you don't need it (`CONFIG_MSM_VIDC=n`) or build vidc as a module (`CONFIG_MSM_VIDC=m`) and load it after the root filesystem is mounted.

<a name="networkmanager"/>

### Making NetworkManager behave

If NetworkManager is eating a lot of CPU, it is probably being confused by the rmnet devices. Ask it to ignore them in /etc/NetworkManager/NetworkManager.conf :

    [keyfile]
    unmanaged-devices=interface-name:rmnet0;interface-name:rmnet1;interface-name:rmnet2;interface-name:rmnet3;interface-name:rmnet4;interface-name:rmnet5;interface-name:rmnet6;interface-name:rmnet7;interface-name:rmnet8;interface-name:rmnet_smux0;interface-name:rev_rmnet0;interface-name:rev_rmnet1;interface-name:rev_rmnet2;interface-name:rev_rmnet3;interface-name:rev_rmnet4;interface-name:rev_rmnet5;interface-name:rev_rmnet6;interface-name:rev_rmnet7;interface-name:rev_rmnet8

<a name="busybox"/>

### Getting Busybox (tar, etc)

If you do not have tar on the board/phone/tablet running android, download it from http://busybox.net/downloads/binaries/latest/busybox-armv7l. Use: 

    adb push busybox-armv7l /system/bin/busybox
    adb shell chmod 0777 /system/bin/busybox

Then either use `busybox tar ...` or create a symbolic link named tar to busybox to use it

<a name="kerneldbgopt"/>

### Kernel Debugging Options

Useful debug kernel options to enable when debugging kernel problems:

    CONFIG_DETECT_HUNG_TASK=y
    CONFIG_DEBUG_SPINLOCK=y
    CONFIG_DEBUG_MUTEXES=y
    CONFIG_DEBUG_ATOMIC_SLEEP=y
    CONFIG_STACKTRACE=y
    CONFIG_DEBUG_BUGVERBOSE=y
    CONFIG_DEBUG_INFO=y

<a name="prebuiltupdate"/>

### Prebuilt userspace vs system updates

If you are using a prebuilt userspace tarball, and system upgrade (yum/dnf/apt-get upgrade, etc) overwrites mesa/libdrm/xf86-video-freedreno, just re-extract latest prebuilt tarball on top.  The most recent prebuilt should be newer version than what is in distro packages.
