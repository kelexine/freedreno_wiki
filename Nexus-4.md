### Mainline kernel
This is the way to go, as robclark said:
> tbh, with the exception of perhaps making phone calls (modem), upstream support for 8064 should be decent (like the work that john stultz did on n7 tablet).. so I probably wouldn't try too hard on ancient 3.4 stuff 

You might find some helpful pointers about mainlining the device [here](https://wiki.postmarketos.org/wiki/The_Mainline_Kernel).

### 3.4 backport
There's a backport from 2013 of the freedreno to a 3.4 kernel in the [kernel-msm](https://github.com/freedreno/kernel-msm) repository, with two branches.
* [mako-kernel](https://github.com/freedreno/kernel-msm/tree/mako-kernel)
  * This one compiles.
  * The display is only "working" by using qcom's customized fbdev driver according to robclark, so it is a bit non-standard. 
  * Notably the USB networking was disabled in the kernel config (in favor for serial debugging?), but it should be possible to activate it again.
  * [package build recipe with GCC6 patches](https://github.com/ollieparanoid/pmbootstrap/tree/beb8b0a4ec65d68681eebe9ab5c5223842716cf8/aports/device/linux-lg-mako)
* [mako-kernel-drm](https://github.com/freedreno/kernel-msm/tree/mako-kernel-drm)
  * This one is said to have the display "kind of almost" working.
  * But it's abandoned work in progress, and doesn't even compile. After [patching some includes](https://github.com/ollieparanoid/pmbootstrap/blob/6d5fb90b41abeee20c375f91f842148656586161/aports/device/linux-lg-mako/02_freedreno_includes.patch) it fails with [these errors](https://gist.github.com/ollieparanoid/9b4ac94b6ba1318c9df730fcea4e6489).

### Links
* [postmarketOS: Nexus 4](https://wiki.postmarketos.org/wiki/Google_Nexus_4_(lg-mako)) (doesn't run freedreno yet as of 2017-12, but the goal is to have it working with the mainline kernel)
* [Fedora installer from 2013](https://github.com/freedreno/nexus4-fedora) (includes a pre-built boot.img file)
