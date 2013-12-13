# Freedreno Architecture Overview

This page gives a brief overview of how the various pieces fit together.  For a bit of general background on linux open source graphics driver stack (not specific to any one driver), see [The Linux Graphics Stack](http://blog.mecheye.net/2012/06/the-linux-graphics-stack/).

The userspace components can function in two modes, either utilizing the msm drm/kms kernel driver, or using the msm fbdev + kgsl drivers from downstream msm android tree.  The main purpose of this is to make it easier to use freedreno on android devices, in particular because msm drm/kms driver does not yet have complete DSI panel support for the LCD displays found on phones/tablets.  (And even once the DSI support is in place in msm drm/kms, there would still be a need to write a panel driver for each different model of LCD panel.)

If you have a device that is supported by the msm drm/kms driver, it is recommended to use that, as it has the advantage of supporting page flips, better hw cursor support, etc.  And it also supports wayland compositors, unlike the fbdev/kgsl combo.

### msm drm/kms

When utilizing the drm/kms driver, the graphics stack looks the same as it would for any other open source desktop driver (nouveau, radeon, etc):

<center>
<a href="http://people.freedesktop.org/~robclark/arch-drm.svg">
<img src="http://people.freedesktop.org/~robclark/arch-drm.svg" width="600" height="725"/>
</a>
</center>

### fbdev/kgsl

When using the android fbdev/kgsl driver, the stack is almost the same except that the fbmode_display modesetting code in xf86-video-freedreno and the kgsl backend in libdrm_freedreno are used instead of drmmode_display and msm backend.  There is no need to recompile any userspace component, xf86-video-freedreno and libdrm_freedreno figure out what to use at runtime.

<center>
<a href="http://people.freedesktop.org/~robclark/arch-kgsl.svg">
<img src="http://people.freedesktop.org/~robclark/arch-kgsl.svg" width="600" height="725"/>
</a>
</center>
