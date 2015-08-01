Overview
--------

A4xx introduces geometry shader support. Unlike other GPU families, it doesn't have actual support for emitting multiple vertices, splitting primitives, etc. It functions by invoking the geometry shader multiple times until it executes a kill instruction, at which point that stage is considered to be over.

Sadly this doesn't map nicely onto how OpenGL expresses geometry shaders, so significant shenanigans have to occur in order to rewrite them efficiently. However an inefficient implementation might just count the number of emitted vertices, and run through all the code until the right one is hit.

Inputs
------
* `r0.x` -- Presumably this is configurable via some register, but I've been unable to get the RA to assign it to anything else, so no idea which bitfield to look in. This value contains a bitfield:
  * Bits 0:4 : ???
  * Bits 10:14 : `gl_InvocationID`
  * Bits 15:24 : The vertex whose outputs we are currently computing
* `r0.y` -- `gl_PrimitiveIDIn`

Incoming vertex values are accessed using the `ldlw` instruction. The exact address is computed with, for example,

    0005[62018003x_10309024x] mad.u24 r0.w, c9.x, (r)r0.w, c12.x
    0013[c286000bx_0480c001x] ldlw.u32 r2.w, l[r0.w], 4

`c9.x` contains `0xc0`, `r0.w` is the bits 0:4 value, and `c12.x` is `0x80`. This particular example was reading all 4 components of the 3rd vertex's `gl_Position`. It appears that the vertices are 0x40 away from each other, but that is probably a function of the number of VS outputs.

Outputs
-------
A single geometry shader runthrough only produces a single vertex's worth of data, so it works very much like vertex shaders do. In addition to the general data, it can also output `gl_PrimitiveID`, `gl_Layer`, and `gl_ViewportIndex` (???) values, the latter 2 select which renderbuffer layer and viewport settings are used.

TODO: How is the mapping done... VPC? Is there a GPC somewhere?