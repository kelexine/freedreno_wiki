* AT&T: also known as d2att or SGH-I747
* Bell: SGH-I747M
* Cricket: also known as d2cri or SCH-R530C
* MetroPCS: also known as d2mtr or SCH-R530M
* Mobilicity: SGH-T999V
* Rogers: SGH-I747R
* Sprint: also known as d2spr or SPH-L710 (verified to work)
* T-Mobile US: also known as d2tmo or SGH-T999
* Telus: SGH-I747M
* U.S. Cellular: also known as d2usc or SCH-R530
* Verizon: also known as d2vzw or SCH-I535 (verified to work)
* Wind Mobile: SGH-T999V

XDA page: http://forum.xda-developers.com/showthread.php?t=2364048

These instructions can possibly work on other Qualcomm based S III varieties. 

## Hardware Specifications
* SoC: Qualcomm Snapdragon S4 MSM8960
* CPU: 1.5&nbsp;GHz Qualcomm Krait, Dual-core 
* RAM: 2&nbsp;GB
* GPU: Qualcomm Adreno 225
* Display: 1280x720, 4.7&nbsp;inch AMOLED
* Internal Storage: 16&nbsp;GB or 32&nbsp;GB internal flash, accessed as an mmcblk device with partitions
* External microSDXC card slot
* Wi-Fi: Broadcom BCM4334 (SDIO)

## Kernel Support
Various 3.0 and 3.4 kernels are available. The 3.0 kernel hasn't been explored much yet because it seems to hang pre-userspace when kexec'd from a 3.4 kernel (both are the "KT747" flashable kernels in this test.) Others have reported success with the CyanogenMod 10.2 kernel at branch "cm-10.2_kgsl" on the [appropriate repo](https://github.com/CyanogenMod/android_kernel_samsung_d2/tree/cm-10.2_kgsl) at the CM github.

## Development Progress
Currently, these are working:
* Framebuffer console (although it has graphical corruption on d2vzw - **change module_init to late_initcall**)
* USB OTG and ADB server over USB
* Wi-Fi with the Broadcom DHD driver (brcmfmac will reboot the system with nothing useful in dmesg) - _be careful with firmware paths, currently my solution is to mount Android's /system, /data and /cache filesystems on top of Ubuntu from fstab._
* The touch screen is reported to work with "evdev" on the Sprint S3. However, on the Verizon model, the server crashes whenever one taps the screen. You can get it working as a trackpad with "xserver-xorg-input-multitouch".
* X.org with Freedreno (2-D acceleration works, 3-D will cause page faults.) Patches and instructions are available are at the XDA page above.

Untested: Bluetooth, audio and camera. The camera is highly unlikely to work without extra code somewhere. Audio might be doable as it was with the HP Touchpad.