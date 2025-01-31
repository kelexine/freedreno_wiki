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

also see: https://bug707361.bugzilla-attachments.gnome.org/attachment.cgi?id=259762 .. this seems to be the tiled format on apq8064 and before. This Gst buffer format can be used to render  the output file generated by VIDC. The format is described here:

http://linuxtv.org/downloads/v4l-dvb-apis/re31.html

and support in Gst was added recently:

http://cgit.freedesktop.org/gstreamer/gst-plugins-base/commit/?id=f8d3b9b4fcc5e08b771314fa95e9ed8f750b54e6
http://cgit.freedesktop.org/gstreamer/gst-plugins-base/commit/?id=d899e6df5ac162709f8d2cf53c23d0fa8ce3f09f

To render the following Gstreamer pipeline should work:

`gst-launch-1.0 filesrc location=output4.yuv ! videoparse`
`format=nv12-64z32 width=1280 height=720  ! videoconvert ! ximagesink`

The support was added recently in Gstreamer, it will be in Gstreamer 1.4, until then you need to recompile gstreamer from git.

It seems that QCOM h/w specs requires that both Luma and Chroma planes are aligned on 8kb. So when using some resolution such as 640x480 or 1080p (when the number of pixels is not a multiple of 8096) the videoparse based Gstreamer pipeline won't work fine, but it won't be a problem once there is a proper plugin for the video decoder.

Note: as of 7/17/2014, Gst v1.3.91 packages have been uploaded in the Linaro PPA (https://launchpad.net/~linaro-maintainers/+archive/ubuntu/qcom-overlay/). So if you run a Linaro/ubuntu image and upgrade the following pipeline works:

`gst-launch-1.0 videotestsrc !  video/x-raw,format=NV12_64Z32,width=1280,height=720 ! videoconvert ! eglglessink`

It does NV12_64Z32 (which is the Gst format name for the QCOM NV12 tiled format) to YUV/I420 in s/w and renders with freedreno through eglglessink...

It works with the output of the VIDC decoder too, if you have output.yuv generated by VIDC:

`gst-launch-1.0 filesrc location=output.yuv ! videoparse format=nv12-64z32 width=1280 height=720   ! videoconvert ! eglglessink`

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
