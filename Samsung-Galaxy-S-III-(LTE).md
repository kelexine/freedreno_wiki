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
* Framebuffer console (although it has graphical corruption - **change module_init to late_initcall**)
* USB OTG and ADB server over USB
* Wi-Fi with the Broadcom "dhd" driver (brcmfmac will reboot the system with nothing useful in dmesg) - be careful with firmware paths, the _b2 part of the extension should not be specified. Use a modprobe conf file to set the firmware_path= and nvram_path= module parameters at load.
* The touch screen driver requires a patch to emit single-touch events for X.org. After that, it works.
* X.org with Freedreno (2-D acceleration works, 3-D will cause page faults.) Patches for 2D are no longer required for git versions, as of 09/27/2014.

Untested: Bluetooth, audio and camera. The camera is highly unlikely to work without extra code somewhere. Audio might be doable as it was with the HP Touchpad. Bluetooth should also be possible.

![MATE/Arch Linux ARM on Samsung SGH-T999/d2tmo](http://s13.postimg.org/m7z815xgn/image.png)