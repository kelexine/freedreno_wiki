# Status:
apps/games/features/etc that are known to work or not

### Features
Implemented/Supported:
* OpenGL 2.1 (a3xx) / 1.4 (a2xx) - on best-effort basis
* OpenGLES 1 and OpenGLES 2
* textures: 2D, mipmap, cubemap, 3D
* 16b or 24b depth buffer
* 8b stencil buffer
* 8b/16b/32b index buffer

Missing:
* OpenGLES 3 - a3xx can support GLES3 features in hw, but missing support in gallium driver
* non-unwindable loops in shaders - should not be too hard to support in a3xx (we already use branching for if/else), is supported by the hw on a2xx but needs a bit of work to figure out how to use it.
* MSAA
* not all TGSI opcode's are implemented, so you might run into a missing one with some weird shaders.. but all the common ones should be implemented

### Adreno 3xx
|       App       | Status |
|:---------------:|:-------|
| **gnome-shell** | no known issues |
| **es2gears**    | works, 900fps (1200fps on apq8074/a330) |
| **glmark2-es2** | works other than `loop` and `terrain`.. glmark2 score 343 |
| **compiz**      | should work.. not all plugins tested |
| **xbmc**        | no known issues, good performance at 1080p, sw decode h264 seems fast enough up to 720p content |
| **xonotic-glx** | no known issues (35fps @720p on firetv) |
| **vdrift**      | no known rendering issues.. but slow |
| **darkplaces**, **openarena**,<br>(and others based on q3 engine)| no known issues (60fps, vsync limited @720p) |
| **etuxracer**   | works except msaa (??fps) |
| **supertuxkart**| works except msaa (30-60fps depending on level, appears to be CPU limited in most slower spots) |
| **alienarena**  | crashes in game (in `Mod_LoadLeafs()`.. does not appear to be freedreno related) |
| **bzflag**      | appears to work |
| **maniadrive**  | appears to work |
| **neverball**   | appears to work |
| **tremulous**   | appears to work (??fps at 1280x720) |
| **0ad**         | needs later than alpha 16 (z-fighting issue with earlier versions); generally works, problematic effects/options: *Post Processing*, *Real Water Depth*,  |
NOTES:
* fps figures are with msm drm/kms driver (otherwise we can't pageflip and have gpu stall on presentation blit); on a320 (unless otherwise noted), 1280x720 (if fullscreen, otherwise results are with XA enabled)

### Adreno 2xx
TODO
