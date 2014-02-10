##### Table of Contents  
* [Getting Kernel Traces](#kerneltraces)
* [err: request_firmware for vidc_1080p.fw error -2](#vidcfwerr)
* [Making NetworkManager behave](#networkmanager)

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
    unmanaged-devices=interface-name:rmnet0;interface-name:rmnet1;interface-name:rmnet2;interface-name:rmnet3;interface-name:rmnet4;interface-name:rmnet5;interface-name:rmnet6;interface-name:rmnet7;interface-name:rmnet_smux0

