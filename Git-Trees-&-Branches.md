# Kernel
The [kernel-msm](https://github.com/freedreno/kernel-msm.git) github tree has branches for various devices.  The interesting ones are:
* __hp-tenderloin-3.0__ - [[HP TouchPad|HP-TouchPad]] kernel based on msm-3.0 (kgsl+fbdev)
* __mako-kernel__ - [[nexus4|Nexus-4]] kernel based on msm-3.4 (kgsl+fbdev)
  * note __mako-kernel-drm__ experimental branch exists, but don't use it yet unless you want to hack on DSI panel support in msm drm/kms driver.
* __ifc6410-drm__ - msm-3.4 based kernel for [[ifc6410|Ifc6410]] board (msm drm/kms)
  * note older __ifc6410__ branch still exists for historical purposes (kgsl+fbdev), but probably no reason to use it anymore
* __bstem-drm__ - msm-3.4 based kernel for [[bSTem|bStem]] board

Other branches are mostly likely just random work-in-progress things, and might get deleted at some point.

# Userspace
The upstream trees are on freedesktop.org:
* __mesa__ - `git://anongit.freedesktop.org/mesa/mesa` ([gitweb](http://cgit.freedesktop.org/mesa/mesa/))
* __libdrm__ - `git://anongit.freedesktop.org/mesa/drm` ([gitweb](http://cgit.freedesktop.org/mesa/drm/))
* __xf86-video-freedreno__ - `git://anongit.freedesktop.org/xorg/driver/xf86-video-freedreno` ([gitweb](http://cgit.freedesktop.org/xorg/driver/xf86-video-freedreno/))

The corresponding github trees master branches are periodically kept in sync with the freedesktop.org trees.  And from time to time will have branches with various work-in-progress stuff that is not ready to push to the upstream freedesktop.org trees yet.

### Build instructions:
These are the configure flags I use (and the order to build things):

**libdrm:**

    ./autogen.sh --prefix=/usr --enable-freedreno

**mesa:**

    ./autogen.sh --prefix=/usr --with-dri-drivers= --with-gallium-drivers=freedreno,swrast --with-egl-platforms=x11 --enable-gles2 --enable-gles1 --enable-debug --enable-gallium-egl --disable-gallium-llvm --enable-xa --disable-dri3

**xf86-video-freedreno:**

    ./autogen.sh --prefix=/usr

Mesa in particular has a lot of build options to play with..  `--enable-debug` does have performance overhead, so while useful for debugging there will be an fps drop, especially for games that are CPU limited.
