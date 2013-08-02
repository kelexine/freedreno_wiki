# Verizon Galaxy S III
### (Also known as d2vzw, or SCH-I535)

These instructions probably work with the Sprint version as well (using its own kernel), and possibly other Qualcomm based S III varieties. 

## Hardware Specifications
* SoC: Qualcomm MSM8960
* CPU: Qualcomm Snapdragon S4 1.5 GHz dual-core Cortex A9 "Krait"
* RAM: 2GB
* GPU: Adreno 225
* Internal Storage: 16GB or 32GB internal flash, accessed as an mmcblk device with partitions
* External microSDXC card slot_
* Wi-Fi: Broadcom BCM4334 (SDIO)

## Kernel Support
Various 3.0 and 3.4 kernels are available. The 3.0 kernel hasn't been explored much yet because it seems to hang pre-userspace when kexec'd from a 3.4 kernel (both are the "KT747" flashable kernels in this test.)

## Development Progress
Currently, these are working:
* Framebuffer console (although it has graphical corruption) - _change module_init to late_initcall_
* USB OTG and ADB server
* Wi-Fi with the Broadcom DHD driver (brcmfmac will reboot the system with nothing useful in dmesg) - _be careful with firmware paths, currently my solution is to mount Android's /system, /data and /cache filesystems on top of Ubuntu from fstab._

**Freedreno** itself is still having issues. With the latest libdrm_freedreno, an assertion will fail and the X server will terminate before displaying anything on the screen. This is currently the focus for bug fixing.

With earlier versions  of libdrm, the cursor will show up, but an IOMMU page fault will occur, and the handler will try to write to a struct curr_context, which turns out to be NULL, which panics the kernel. When a check is added to prevent this, one can get past the login screen, but the system will still eventually reboot. In that case, from the point that the server is started to the point of reboot, the IOMMU IRQ handler will show up in the output of "top" with nearly 100 percent CPU usage, indicating that the IOMMU is repeatedly dealing with pagefaults from Freedreno. 

THe touch screen does not work under X with tslib, and is not found by evdev's catchall rules. When added to th the server config as an evdev device, the server will crash on any touch event.

Untested: audio and camera. Both are highly unlikely to work without extra code somewhere.