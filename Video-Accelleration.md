For now, just a place to stash some notes about qcom hw video encode/decode.

NOTE: the video encode/decode acceleration is not actually part of the GPU.  However, we certainly want to be able to use the output of decoder as textures (tiled YUV formats and all).  And it may make sense to integrate video accel in the freedreno gallium driver for a couple of reasons:
1. support for all the gallium video state trackers (VDPAU, XvMC, and OpenMAX)
2. Easier to deal with multiplanar and custom formats if it is all hidden within the gallium driver.  In some sense, multi-planar buffers are a bit similar to multi-slice buffers (mipmap, cube textures, etc).  It at least sidesteps having to punch extensions all the way through egl and mesa state-tracker layers.

***

Code: https://android.googlesource.com/platform/hardware/qcom/media/

The kernel interface is some sort of (custom?) v4l device.

***

tiled YUV format: seems libc2d2 (2d api implemented on top of z180 in older a2xx devices, and on gpu with newer devices) supports `C2D_FORMAT_MACROTILED` format flag, which should be enough to figure out texture format.

see: https://android.googlesource.com/platform/hardware/qcom/media/+/master/libc2dcolorconvert/

***

seems qcom omx implementation has it's own h264, etc, parsers:
https://android.googlesource.com/platform/hardware/qcom/media/+/master/mm-video-v4l2/vidc/vdec/src/h264_utils.cpp

gallium provides this sort of parsing for us (see src/gallium/auxillary/vl)

***

WTF? https://android.googlesource.com/platform/hardware/qcom/media/+/master/mm-video-v4l2/vidc/vdec/src/power_module.cpp

***

There seems to be some copy/paste/modify going on for different hw generations.  Note entirely sure which SoC's use which of these:

https://android.googlesource.com/platform/hardware/qcom/media/+/master/mm-video-v4l2/vidc/vdec/src/omx_vdec.cpp
https://android.googlesource.com/platform/hardware/qcom/media/+/master/mm-video-v4l2/vidc/vdec/src/omx_vdec_hevc.cpp
https://android.googlesource.com/platform/hardware/qcom/media/+/master/mm-video-v4l2/vidc/vdec/src/omx_vdec_msm8974.cpp

Probably for a gallium implementation, we'll need to come up with a cleaner way to manage different hw generations.
