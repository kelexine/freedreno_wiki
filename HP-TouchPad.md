The HP TouchPad is a Qualcomm Snapdragon-based tablet computer from HP.  Originally shipping with WebOS, community projects have ported both Android and native Linux to the system.  Ubuntu is the most developed native Linux OS on the HP TouchPad and is capable of running the Freedreno driver.

## Hardware Specs
* SoC: Qualcomm APQ8060
* CPU: Qualcomm Snapdragon S3 1.2GHz dual-core Cortex A8 "Scorpion" (the 64GB version runs at 1.5GHz max)
* RAM: 1GB
* GPU: Adreno 220
* Storage: 16, 32, or 64GB of internal Flash formatted as an LVM volume
* Display: 9.7" 1024x768 IPS LCD
* WiFi: Atheros ath6kl (SDIO)

## Kernel Support
The HP TouchPad shipped with a modified 2.6.35 kernel.  In addition, there is a community-ported 3.0.8 kernel in development.  Freedreno currently works with both of these kernels.

* For userland which still supports it, use of the 2.6 kernel is recommended. The 3.0.8 kgsl drivers have been stably backported to it, and the result is available at willcast/kernel_tenderloin branch 2.6.35_desktop.
* There is now a 3.4.94 kernel at willcast/kernel_tenderloin, branch desktop_3.4.
* There is also a 3.0.101 kernel at branch desktop_3.0, but it hasn't been touched and is completely untested.
* The old 3.0.8 kernel is available at freedreno/kernel-msm.

## Fedora installation
There is a port of Fedora 18 laying around. It has functional graphics.

## General Development and Ubuntu Installation
Ubuntu 11.10 through 13.04 are all avaliable.

### Ubuntu 13.04, 13.10-dev
Available from XDA Developers. (The final version of) 13.04 requires a kexec bootloader such as kexecboot or nsboot. These are all available under the Other TouchPad Development forum. (nsboot is definitely recommended over kexecboot.) 13.10 porting is unfinished.
* [XDA Developers 13.x Development Thread](http://forum.xda-developers.com/showthread.php?t=2225462)

### Ubuntu 11.10, 12.04, 12.10
To install Ubuntu on your TouchPad you will first need to install moboot, the bootloader used to boot multiple OS'es on the TouchPad.  The easiest way to do this is with ACMEInstaller2 for Android installation.  After installing Android (and thus moboot), follow the thread below to install Ubuntu 11.10. 12.04 is available by upgrading 11.10. The 12.10 rootfs is in the same thread, around page 65.

* [XDA Developers Ubuntu 11.10 Development Thread](http://forum.xda-developers.com/showthread.php?t=1304475)