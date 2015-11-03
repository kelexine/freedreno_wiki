Overview
--------

A4xx introduces geometry shader support. As a brief reminder, an OpenGL geometry shader receives fully assembled primitives (up to 6 vertices for the adjacency variants), and outputs any number of primitives (including 0), with a limit of up to N vertices (implementation-dependent). The output primitives can be points, line strips, or triangle strips. Cutting a primitive will effectively restart it (using `EndPrimitive()`). The shader can supply any number of per-vertex varyings, as well as some per-primitive varyings like `gl_PrimitiveID`, `gl_Layer`, and `gl_ViewportIndex`.

More advanced geometry shaders, available with [[ARB_gpu_shader5|https://www.opengl.org/registry/specs/ARB/gpu_shader5.txt]] and in part its ES cousin, [[OES_gpu_shader5|https://www.khronos.org/registry/gles/extensions/OES/OES_gpu_shader5.txt]] allow instancing, whereby a single geometry shader will have up to N parallel executions, differentiated with `gl_InvocationID`, as well as multi-stream transform feedback, whereby a particular vertex and its associated varyings are fed into one or another stream. Note that only stream 0 is passed on for rasterization, and this (multi-stream TF) can only be done with points primitives.

Unlike other GPU families, A4xx doesn't have actual support for emitting multiple vertices, splitting primitives, etc from within the shader. It functions by invoking the geometry shader multiple times until it executes a kill instruction, at which point that "invocation" is considered to be over.

Sadly this doesn't map nicely onto how OpenGL expresses geometry shaders, so significant shenanigans have to occur in order to rewrite them efficiently. However an inefficient implementation might just count the number of emitted vertices, and run through all the code until the right one is hit. In other words, one might implement `EmitVertex()` and `EndPrimitive()` like

    int n = 0;
    int prim_info = 0;
    void EmitVertex() {
       prim_info |= gl_Layer << 7; // and maybe |= gl_ViewportIndex << 3
       if (n == TheVertexForThisExecution)
          exit();
       n++;
       prim_info = 0;
    }
    void EndPrimitive() { prim_info = 4; }

Inputs
------
* `r0.x` -- Presumably this is configurable via some register, but I've been unable to get the RA to assign it to anything else, so no idea which bitfield to look in. This value contains a bitfield:
  * Bits 0:4 : primitive offset in buffer
  * Bits 10:14 : `gl_InvocationID`
  * Bits 15:24 : The vertex whose outputs we are currently computing
* `r0.y` -- `gl_PrimitiveIDIn`

Incoming vertex values are accessed using the `ldlw` instruction. The exact address is computed with, for example,

    0005[62018003x_10309024x] mad.u24 r0.w, c9.x, (r)r0.w, c12.x
    0013[c286000bx_0480c001x] ldlw.u32 r2.w, l[r0.w], 4

`c9.x` contains `0xc0`, `r0.w` is the bits 0:4 value, and `c12.x` is `0x80`. This particular example was reading all 4 components of the 3rd vertex's `gl_Position`.

Vertex data is laid out in the space accessed by ldlw sequentially. A particular vertex attribute forms a group of the N vertices in the input primitive, and it appears that the baseline quantity is `0x40` for "no" attributes (i.e. just position and probably others). So the layout is as follows for triangle inputs:

    0x00: V1 pos, V2 pos, V3 pos
    ...
    0xc0: V1 attr0, V2 attr0, V3 attr0
    0xf0: V1 attr1, V2 attr1, V3 attr1
    ...

So as a result, in order to access a non-system attribute `N` in a primitive with `V` vertices for vertex `v`, you have to access it at `0x40 * V + N * 0x10 * V + v * 0x10`. System vertices are laid out in per-vertex groups as:

    0x00: gl_Position
    0x10: gl_PointSize, ???
    0x20: ???
    0x30: ???

So the gl_Position of vertex `v` would be at `0x40 * v`, while the gl_PointSize would be at `0x40 * v + 0x10`. And of course the whole thing is offset by `r0.x & 31 * primitive stride`.

Outputs
-------
A single geometry shader runthrough only produces a single vertex's worth of data, so it works very much like vertex shaders do. In addition to the general data, it can also output `gl_PrimitiveID`, `gl_Layer`, and `gl_ViewportIndex` (???) values, the latter 2 select which renderbuffer layer and viewport settings are used.

There is an output map for these, same as for VPC. In addition there are explicit registers specifying the position of the vertex as well as a primitive information register. This contains whether a new primitive should be started, the target layer, and probably viewport index. The layer starts at bit 7, the new primitive request value is 4. Speculation is the viewport index is at bit 3. In order for the fragment shader to receive any of these special values, they must be passed as a regular varying.

It also appears that the point size goes into the register following the primitive info register. However there's no additional indication that the point size is being provided.

Optimization
------------
By using the `EmitVertex` implementation as above, each shader will execute `O(n^2)` times. This may be undesirable. However if the computation for data of vertex n+1 does not depend on the computation for data of vertex n, it should be possible to rewrite the shader with a `switch` statement. For example a shader that looks like

    for (int i = 0; i < 3; i++) {
      gl_out[i].gl_Position = gl_in[i].gl_Position;
      EmitVertex();
    }

it should be possible to rewrite it as

    switch (TheVertexForThisExecution) {
    case 0: gl_Position = gl_in[0].gl_Position; break;
    case 1: gl_Position = gl_in[1].gl_Position; break;
    case 2: gl_Position = gl_in[2].gl_Position; break;
    }
    if (TheVertexForThisExecution == 2) kill;

or even better as

    gl_Position = gl_in[TheVertexForThisExecution].gl_Position;
    if (TheVertexForThisExecution == 2) kill;