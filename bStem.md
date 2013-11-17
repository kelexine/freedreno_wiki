see: http://www.braincorporation.com/bstem-faq/

Same basic instructions as for [[ifc6410|Ifc6410]] should work.  I use the same filesystem for both boards.

A work in progress kernel is [here](http://people.freedesktop.org/~robclark/f20/bstem-boot-f20-2.img) (or on kernel-msm [bstem-drm](https://github.com/freedreno/kernel-msm/commits/bstem-drm) branch).  Note that as of 16-Nov the kernel is still work in progress.. memory protection for the GPU is missing, so don't use this kernel for production purposes.  Hopefully this should be solved soon.
