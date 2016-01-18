**Adreno A2xx**: Requires `kgsl` (no upstream support in kernel), works "barely", enough for gnome-shell/etc
 * OpenGL 1.4
 * OpenGL ES 2.0

**Adreno A3xx**:
 * LVDS / HDMI / DSI
 * OpenGL 3.1
 * OpenGL ES 3.0
 * Hardware binning

**Adreno A4xx**:
 * HDMI / eDP / DSI
 * OpenGL 3.1
 * OpenGL ES 3.0

For detailed GL extension information in mesa, see http://people.freedesktop.org/~imirkin/glxinfo/glxinfo.html#p=compat

***

**Missing**:
* MSAA (technically required for OpenGL ES 3.0)
* Texture tiling (perf boost)
* HW binning (on a4xx)
* OpenGL 3.2: Geometry shaders (a4xx only), MS textures
* OpenGL 3.3: RGB10_A2UI textures/vertices (a3xx), dual-source blending (a4xx), timer query
* OpenGL 4.0: Tessellation shaders, sample shading, ARB_gpu_shader5 features, indirect draws, lots more
* OpenGL ES 3.1: Compute, SSBO, counters, images, indirect draws (a3xx may not have enough in hardware for all this)