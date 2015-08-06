# Inforce 6540

Information about freedreno on the [inforce 6540](http://www.inforcecomputing.com/products/single-board-computers/6540-single-board-computer-sbc) pico-itx board.

NOTE: this page is still a bit of a placeholder.  See also [linaro ifc6540 wiki page](https://wiki.linaro.org/Boards/IFC6540)

## kernel

As of now (Nov 2014) the gdsc regulators, and maybe a few other things are missing upstream.  So for kernel I have been using latest drm backported to the android 3.10 kernel.  See the [inforce 6540-drm branch](https://github.com/freedreno/kernel-msm/commits/ifc6540-drm)

## userspace

Very preliminary gallium support for a4xx has been pushed to upstream mesa.

