For something a bit more current, see the trello page: https://trello.com/b/VC0IXzrq/freedreno

Here is a list of things needed, in no particular order.

### gallium:
* misc:
 * [ ] MSAA
 * [X] tiled textures
 * [X] Emulate unsupported texture wrap modes in shader - a3xx does not seem to support mirror-clamp, mirror-clamp-to-border, and mirror-clamp-to-edge seems to only work for power-of-two textures.
   * emulation for GL_CLAMP done, for others we have stopped advertising the extensions for now
* compiler:
 * [X] add relative addressing support to new-compiler (and remove legacy compiler)
 * [X] integer support
 * [X] loops / switch / subroutines
  * note gallium will unroll loops with constant # of iterations, and inline functions, if the gallium driver does not support.. sufficient for simple shaders and gles2.
 * [x] derivative support (`TGSI_OPCODE_DDX`, `TGSI_OPCODE_DDY`)
* gl2 support:
 * [x] occlusion query
 * [x] additionally sRGB texture support will bring us up to gl2.1
* gles3
 * [x] GLSL version level 130 (integer support, loops, and ??)
  * note non-unrollable loops (which are less common) still missing, but we advertise glsl 130 all the same
 * [x] packed float (`PIPE_FORMAT_R11G11B10_FLOAT`)
 * [x] [EXT_texture_shared_exponent](http://developer.download.nvidia.com/opengl/specs/GL_EXT_texture_shared_exponent.txt) (`PIPE_FORMAT_R9G9B9E5_FLOAT`)
 * [x] transform feedback
 * [x] sRGB framebuffer support
 * [x] UBO's
 * [x] MRT
* gl3 support:
 * [x] GLSL version level 130 (integer support and ??)
 * [X] texture compression (`PIPE_FORMAT_RGTC{1,2}_{U,S}NORM`)
 * [x] packed float (`PIPE_FORMAT_R11G11B10_FLOAT`)
 * [x] [EXT_texture_shared_exponent](http://developer.download.nvidia.com/opengl/specs/GL_EXT_texture_shared_exponent.txt) (`PIPE_FORMAT_R9G9B9E5_FLOAT`)
 * [x] transform feedback
 * [ ] MSAA >= 4
 * [X] 32b float depth (`PIPE_FORMAT_Z32_FLOAT` and `PIPE_FORMAT_Z32_FLOAT_S8X24_UINT`)
 * [X] [NV_conditional_render](http://www.opengl.org/registry/specs/NV/conditional_render.txt) (`PIPE_CAP_CONDITIONAL_RENDER`)
* gl3.1 support:
 * [ ] GLSL version level 140 (needs ??)
 * [x] UBO's
 * [x] [ARB_draw_instanced](https://www.opengl.org/registry/specs/ARB/draw_instanced.txt) (`PIPE_CAP_TGSI_INSTANCEID`)
 * [X] [ARB_texture_buffer_object](https://www.opengl.org/registry/specs/ARB/texture_buffer_object.txt) (`PIPE_CAP_TEXTURE_BUFFER_OBJECTS`)
 * [x] [EXT_texture_snorm](https://www.opengl.org/registry/specs/EXT/texture_snorm.txt)

### mesa/gallium:
* [ ] add support for [EGL_ARM_pixmap_multisample_discard](http://www.khronos.org/registry/egl/extensions/ARM/EGL_ARM_pixmap_multisample_discard.txt) and/or [GL_EXT_multisampled_render_to_texture](https://www.khronos.org/registry/gles/extensions/EXT/EXT_multisampled_render_to_texture.txt).. either or both of these would be a big win for MSAA on tiler gpu's.  Also would be possibly worthwhile to implement a GL/GLX version of the same extensions.  Extension needs to be implemented in mesa and multisample-discard attribute passed through to gallium driver.

### drm/msm (kernel):
* mdp4 (apq8060, apq8064, etc):
 * [X] DSI support
 * [x] LVDS support
 * [x] YUV plane (overlay)
 * [ ] plane scaling
* mdp5 (apq8074, etc):
 * [x] DSI support
 * [x] displayport support
 * [x] YUV plane (overlay)
 * [x] plane scaling
 * [x] hw cursor
