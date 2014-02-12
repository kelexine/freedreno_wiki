# Status:
apps/games/features/etc that are known to work or not

### Features
Implemented/Supported:
* OpenGL 1.4 - on best-effort basis
* OpenGLES 1 and OpenGLES 2
* textures: 2D, mipmap, cubemap, 3D
* 16b or 24b depth buffer
* 8b stencil buffer
* 8b/16b/32b index buffer

Missing:
* OpenGL 2 - at least some could be supported on a3xx with a bit of work in compiler, but a2xx cannot support all necessary GLSL
* OpenGLES 3 - a3xx can support GLES3 features in hw, but missing support in gallium driver
* non-unwindable loops in shaders - should not be too hard to support in a3xx (we already use branching for if/else), is supported by the hw on a2xx but needs a bit of work to figure out how to use it.
* MSAA
* discard/kill in fragment shaders
* not all TGSI opcode's are implemented, so you might run into a missing one with some weird shaders.. but all the common ones should be implemented

### Adreno 3xx
|       App       | Status |
|:---------------:|:-------|
| **gnome-shell** | no known issues |
| **compiz**      | should work.. not all plugins tested |
| **xbmc**        | no known issues, good performance at 1080p, sw decode h264 seems fast enough up to 720p content |
| **xonotic-glx** | no known issues (15-20fps @720p) |
| **vdrift**      | no known rendering issues.. but slow |
| **darkplaces**, **openarena**,<br>(and others based on q3 engine)| no known issues (60fps, vsync limited @720p) |
| **etuxracer**   | works except msaa (~20-30fps w/ bo-cache) |
| **supertuxkart**| works except msaa (~20-30fps w/ bo-cache) |
| **alienarena**  | crashes in game (in `Mod_LoadLeafs()`.. does not appear to be freedreno related) |
| **bzflag**      | appears to work |
| **maniadrive**  | crashes at startup (`unknown VS semantic name: BCOLOR`) |
| **neverball**   | appears to work |
| **tremulous**   | appears to work (~20fps at 1280x720) |
NOTES:
* fps figures are with msm drm/kms driver (otherwise we can't pageflip and have gpu stall on presentation blit)
* the games with lower framerates tend to suffer due to heavy vertex shader workload because binning-pass is not implemented yet.  So vertex shaders are executed for each (approx) 256x256 tile.  This matters less for apps like window managers or xbmc, which would benefit more from compiler optimizations.

### Adreno 2xx
TODO
