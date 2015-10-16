Adreno A2xx: Requires `kgsl` (no upstream support in kernel), works "barely", enough for gnome-shell/etc
 * OpenGL 1.4
 * OpenGL ES 2.0

Adreno A3xx:
 * LVDS / HDMI / DSI
 * OpenGL 2.1
 * OpenGL ES 3.0
 * Hardware binning

Adreno A4xx:
 * HDMI / eDP / DSI
 * OpenGL 2.1
 * OpenGL ES 3.0

Missing:
* MSAA (technically required for OpenGL ES 3.0)
* Texture tiling
* OpenGL 3.0 / 3.1: RGTC, conditional rendering
* OpenGL 3.2 / 4.0: Geometry/Tessellation shader capabilities (a4xx only)
* OpenGL ES 3.1: Compute, SSBO, counters, images (a3xx may not have enough in hardware for this)

For detailed GL extension information in mesa, see http://people.freedesktop.org/~imirkin/glxinfo/glxinfo.html#p=compat