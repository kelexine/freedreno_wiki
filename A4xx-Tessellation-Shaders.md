Overview
--------

A4xx tessellation shaders map fairly reasonably onto the GL concepts, unlike geometry shaders. Hull (aka tessellation control) shaders run as vertex shaders, and domain (aka tessellation evaluation) shaders run as geometry shaders.

Tessellation enables custom tessellation of arbitrary geometry. In order to achieve this, data is passed around in patches, which are just groups of vertices, up to 32. This is configurable via `glPatchParameter`. A hull shader receives an input patch, and is meant to produce an output patch. The number of vertices between input and output patches need not match (and a single hull shader may be used with different sizes of input patches). A single hull invocation is run for each output vertex patch. Additionally, hull shaders can *read* other invocations' outputs, synchronized via the GLSL `barrier()` call. However they can only write their own outputs.

Each patch is then fed into a hardware tessellation unit. Additionally, the tessellator consumes tessellation factors, determined by the hull shader, as well as more global configuration, like the domain mode, whether the output primitives should be connected, and if they are to be connected, their faceness (CW vs CCW). Based on the tessellation factors and mode, the tessellator invokes the domain shader for some number of points in the "generic" domain, with its position determined via `gl_TessCoord` (which has different meanings for different modes). The domain shader has access to the full patch and generates a single vertex's worth of data, which is then assembled into primitives by the tessellator.

In order to trigger a draw with tessellation, the draw packet's `TESS_MODE` needs to be set to `0xc` for quads, `0xd` for triangles, and `0xe` for isolines. The `PRIM_TYPE` should be `0x1f + PATCH_VERTICES` (so for the default `PATCH_VERTICES = 3`, we would use `0x22`). Additionally the `PC_HS_PARAM` register contains settings for the number of output vertices from HS, as well as tessellator parameters (spacing, connectivity, faceness). These are derived from both hull and domain shader properties.

```
PC_HS_PARAM: { VERTICES_OUT = 32 | SPACING = EVEN_SPACING | CW | CONNECTED }
```

Note that for isolines, `CW` actually represents the `CONNECTED` setting. (Similarly to NVIDIA hw, curiously enough.)

The tessellator will expect tessellation parameters packed in at `PC_TESSFACTOR_ADDR`, with each factor group taking up 12, 20, 28 bytes for isolines, triangles, and quads respectively. The first 4 bytes should be the primitive ID (but only set conditionally based on a const, which means that it's only needed ... when? maybe when gs/fs consume it?). The next bytes are the outer factors, followed by the inner factors. For isolines, that's just 2 outer factors, for triangles it's 3 outer factors followed by 1 inner factor, and quads have all 6.

Tessellation Control Shaders
----------------------------

### Inputs
Presumably the specific regid's are configurable via some register, but I've been unable to get the RA to assign it to anything else, so no idea which bitfield to look in.
* `r0.x` --  This value contains a bitfield:
  * Bits 10:14 : `gl_InvocationID`
* `r0.y` -- `gl_PrimitiveID`
* `r0.z` -- primitive number in patch "buffer". Used to compute the offset in the tessfactor patch outputs, as well as regular patch outputs.

```
opcode: CP_LOAD_STATE (30) (3 dwords)
        { DST_OFF = 0 | STATE_SRC = SS_INDIRECT_STM | STATE_BLOCK = SB_VERT_SHADER | NUM_UNIT = 5 }
        { STATE_TYPE = ST_SHADER | EXT_SRC_ADDR = 0xc0101000 }
```


Tessellation Evaluation Shaders
-------------------------------

```
opcode: CP_LOAD_STATE (30) (3 dwords)
        { DST_OFF = 0 | STATE_SRC = SS_INVALID_ALL_IC | STATE_BLOCK = SB_GEOM_SHADER | NUM_UNIT = 1 }
        { STATE_TYPE = ST_SHADER | EXT_SRC_ADDR = 0xc00ea000 }
```


Notes:

These are likely related to HS register inputs or ... something:

```
t0                      write 0x230c
                                0x230c: 00010000
c00812e4:                       0000230c 00010000
t0                      write 0x2318
                                0x2318: 00200c00
c00812ec:                       00002318 00200c00
t0                      write 0x2319
                                0x2319: 02000000
c00812f4:                       00002319 02000000
t0                      write 0x2340
                                0x2340: 00000000
c00812fc:                       00002340 00000000
```