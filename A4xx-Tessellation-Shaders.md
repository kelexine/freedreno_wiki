Overview
--------

A4xx tessellation shaders map fairly reasonably onto the GL concepts, unlike geometry shaders. Hull (aka tessellation control) shaders run as vertex shaders, and domain (aka tessellation evaluation) shaders run as geometry shaders.

Tessellation enables custom tessellation of arbitrary geometry. In order to achieve this, data is passed around in patches, which are just groups of vertices, up to 32. This is configurable via `glPatchParameter`. A hull shader receives an input patch, and is meant to produce an output patch. The number of vertices between input and output patches need not match (and a single hull shader may be used with different sizes of input patches). A single hull invocation is run for each output vertex patch. Additionally, hull shaders can *read* other invocations' outputs, synchronized via the GLSL `barrier()` call. However they can only write their own outputs.

Each patch is then fed into a hardware tessellation unit. Additionally, the tessellator consumes tessellation factors, determined by the hull shader, as well as more global configuration, like the domain mode, whether the output primitives should be connected, and if they are to be connected, their faceness (CW vs CCW). Based on the tessellation factors and mode, the tessellator invokes the domain shader for some number of points in the "generic" domain, with its position determined via `gl_TessCoord` (which has different meanings for different modes). The domain shader has access to the full patch and generates a single vertex's worth of data, which is then assembled into primitives by the tessellator.

In order to trigger a draw with tessellation, the draw packet's `TESS_MODE` needs to be set to `0xc` for quads, `0xd` for triangles, and `0xe` for isolines. If there's a geometry shader, add 4 to `TESS_MODE` (verified on A5xx only). The `PRIM_TYPE` should be `PATCH_VERTICES - 1` in addition to the `0x20` bit (so for the default `PATCH_VERTICES = 3`, we would use `0x22`). Additionally the `PC_HS_PARAM` register contains settings for the number of output vertices from HS, as well as tessellator parameters (spacing, connectivity, faceness). These are derived from both hull and domain shader properties.

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
  * Bits 0:4 : primitive offset in buffer
  * Bits 10:14 : `gl_InvocationID`
* `r0.y` -- `gl_PrimitiveID`
* `r0.z` -- primitive number in patch "buffer". Used to compute the offset in the tessfactor patch outputs, as well as regular patch outputs.

See [[A4xx Geometry Shaders]] for how inputs are read in with `ldlw`. The same logic follows here. The input primitive size is based on `gl_PatchVerticesIn`, which comes in via a driver-supplied const.

### Outputs
There are no regid-based outputs from a hull shader. It writes all of its data into gmem. In order to interact with the other hull shaders, it first stages all of its outputs (including per-vertex) outputs into "private" memory, accessible via `stp` and `ldp`, and then invocation 0 writes all of them out into global memory. The layout of the tess factors is fixed since the hardware has to read it in, but the regular patch outputs and per-vertex outputs are free-form and up to the driver -- just have to match to what the domain shader expects to read in.

The address where factors should be written is determined with

```
0005[62010007x_10241038x] mad.u24 r1.w, c14.x, r0.z, c9.x
```

where c14.x contains the stride as outlined above, and c9.x contains the tessfactor base address. Unfortunately since the stride differs by tessellation mode, it also means that the output code has to be customized. Either it needs to predicate based on the stride, or it needs to just be different based on the mode.

_Question: How big does the patch buffer need to get? Does it have to be big enough for the whole draw? Or only up to N patches at a time? Hopefully the latter._

A barrier is simply implemented as a (ss) flag on the next instruction, which forces all loads/stores to complete.

```
opcode: CP_LOAD_STATE (30) (3 dwords)
        { DST_OFF = 0 | STATE_SRC = SS_INDIRECT_STM | STATE_BLOCK = SB_VERT_SHADER | NUM_UNIT = 5 }
        { STATE_TYPE = ST_SHADER | EXT_SRC_ADDR = 0xc0101000 }
```


Tessellation Evaluation Shaders
-------------------------------

### Inputs
Presumably the specific regid's are configurable via some register, but I've been unable to get the RA to assign it to anything else, so no idea which bitfield to look in.
* `r0.x` --  ?
* `r0.y` -- `gl_PrimitiveID`
* `r0.z` -- primitive number in patch "buffer". This matches up to the value used in the hull shader, and can be used to retrieve either tessfactors or patch/vertex values stored by the hull shader.

The `gl_TessCoord.xy` values are supplied via
```
VFD_CONTROL_3: { REGID_VTXCNT = r63.x | REGID_TESSX = r0.w | REGID_TESSY = r1.x }
```
and the `.z` value is computed as `1 - x - y` for triangle domains (these are barycentric coordinates), 0 otherwise.

The remaining inputs, which are written by the hull shader, are read out of global memory with `ldg`, using the `r0.z` value + patch stride to index into a shared global buffer.

### Outputs

Tessellation evaluation outputs are handled the same way as vertex shader outputs, with `SP_DS_PARAM_REG` and `SP_DS_OUT[].REG` specifying the outputs.

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