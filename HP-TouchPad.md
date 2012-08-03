The HP TouchPad is a Qualcomm Snapdragon-based tablet computer from HP.  Originally shipping with WebOS, community projects have ported both Android and native Linux to the system.  Ubuntu is the most developed native Linux OS on the HP TouchPad and is capable of running the Freedreno driver.

## Hardware Specs
* SoC: Qualcomm APQ8060
* CPU: Qualcomm Snapdragon 1.5GHz dual-core Cortex A8
* RAM: 1GB
* GPU: Adreno 220
* Storage: 16 or 32GB of internal Flash formatted as an LVM volume
* Display: 10" 1024x768 IPS LCD
* WiFi: Atheros ath6kl (SDIO)

## Kernel Support
The HP TouchPad shipped with a modified 2.6.35 kernel.  In addition, there is a community-ported 3.0.8 kernel in development.  Freedreno currently works with both of these kernels.

## General Development and Ubuntu Installation
To install Ubuntu on your TouchPad you will first need to install moboot, the bootloader used to boot multiple OS'es on the TouchPad.  The easiest way to do this is with ACMEInstaller2 for Android installation.  After installing Android (and thus moboot), follow the thread below to install Ubuntu 11.10.
* [XDA Developers Ubuntu 11.10 Development Thread](http://forum.xda-developers.com/showthread.php?t=1304475)