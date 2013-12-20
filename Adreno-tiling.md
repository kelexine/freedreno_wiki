Like many of the embedded/SoC gpu's, adreno is a tiling based architecture.  However the way that it is implemented is a bit simpler.

Most tilers render small (32x32 and/or 64x64) tiles, with the hw somehow sorting geometry per tile.  Usually non-visible surfaces for a given tile are ignored.  (IMG/sgx does per-pixel hidden surface removal, unlike the others which are per primitive.  The value of this for general use is debatable.)

Adreno, on the other hand, has a (relatively) large in-core (GMEM) or on-chip (OCMEM) tile buffer of anywhere from 256KB to 1MB.  The buffer being rendered is divided into "tiles" or "bins" for which the color buffer(s) and (if enabled) depth/stencil buffers can fit within the tile buffer.  The driver is completely responsible for per tile/bin drawing, as well as restore (moving data from system memory to GMEM) and resolve (moving from GMEM to system memory).

* note that it seems that the resolve step can be used for multi-sample resolve by doing multiple passes.

### Naive Approach:

The naive approach is to build up cmds to setup state and perform clears/draws ignoring tiling.  And then in the top-level cmdstream buffer (what is submitted to the kernel), do the (optional) resolve, per-tile setup, IB (branch) to the clear/draw commands, and then resolve:

<center>
![adreno binning](http://freedreno.github.io/images/adreno-binning.png)
</center>
* Rendering within each tile works like traditional IMR
* The per-tile commands:
  * restore - optional transfer contents from system memory to tile buffer
  * setup window-offset and screen scissor
  * IB to clear/draw rendering commands
  * resolve - transfer tile buffer to system memory
* Notes:
  * the order of cmdstream building is not the same order that GPU executes, and restore/resolve dirties some GPU state registers that are also used in clear/draw, so some care needs to be taken in the driver about marking certain state objects as dirty before the first clear/draw.

This is what is currently implemented in the gallium driver.  It is not necessarily a huge disadvantage when there is small amount of geometry and/or inexpensive vertex shader.

### Optimized Approach:

The naive approach has the disadvantage that the vertex shader runs for each vertex for each bin.  But this can be split into two passes to reduce the overhead.  In the first pass (the "binning" pass), the vertices are split into per-tile bins.  This information is used in the second pass to limit the vertices processed for each bin.  There is no requirement to use the same vertex shader on both passes, the binning pass can use a simplified shader which only computes `gl_Position`/`gl_PointSize`.

> NOTE the below notes are for a3xx, but a2xx should be roughly similar

The blob driver assigns each tile a `VSC_PIPE` (Visibility Stream C<something>? Pipe).  There are eight pipes, and more than one tile can be assigned to a pipe, ie, it could use four pipes for an arrangement of 4x4 tiles, like so:

    #  X, Y = upper-left tile coord of group of tiles mapped to pipe
    #  W, H = size of group in tiles, so below each pipe is mapped to
    #         a 2x2 group of tiles
    VSC_PIPE[0].CONFIG: { X = 0 | Y = 0 | W = 2 | H = 2 }
    VSC_PIPE[0x1].CONFIG: { X = 0 | Y = 2 | W = 2 | H = 2 }
    VSC_PIPE[0x2].CONFIG: { X = 2 | Y = 0 | W = 2 | H = 2 }
    VSC_PIPE[0x3].CONFIG: { X = 2 | Y = 2 | W = 2 | H = 2 }

For each pipe the driver configures pipe buffer address/size (`VSC_PIPE[n].DATA_ADDRESS` and `VSC_PIPE[n].DATA_LENGTH`), which gives the gpu a location to store the visibility stream data.  And the size buffer (`VSC_SIZE_ADDRESS`), a 4 byte * 8 pipes buffer, where the gpu stores amount of data written to each pipe buffer.  This is used during the binning pass to control which vertices are stored to which pipe.

During the rendering pass, at the start of each tile, the driver configures the gpu to use the data from pipe `p` (ie. appropriate buffer size/address) via `CP_SET_BIN_DATA` packet:

	OUT_PKT3(ring, CP_SET_BIN_DATA, 2);
	OUT_RELOC(ring, pipe[p].bo, 0);  /* same value as VSC_PIPE[p].DATA_ADDRESS */
	OUT_RELOC(ring, size_addr_bo, (p * 4));  /* same value as VSC_SIZE_ADDRESS + (p * 4) */

	OUT_PKT0(ring, REG_A3XX_PC_VSTREAM_CONTROL, 1);
	OUT_RING(ring, A3XX_PC_VSTREAM_CONTROL_SIZE(pipe[p].config.w * pipe[p].config.h) |
			A3XX_PC_VSTREAM_CONTROL_N(n));  /* N is 0..(SIZE-1) */

	OUT_PKT3(ring, CP_SET_BIN, 3);
	OUT_RING(ring, 0x00000000);
	OUT_RING(ring, CP_SET_BIN_1_X1(x1) | CP_SET_BIN_1_Y1(y1));
	OUT_RING(ring, CP_SET_BIN_2_X2(x2) | CP_SET_BIN_2_Y2(y2));

### Performance Notes:

As with most/all tilers, switching render target is expensive and triggers a flush.  Also, at least with freedreno gallium driver, if you are only rendering part of a buffer it is recommended to scissor out parts of the buffer that won't be touched.  The gallium driver can use this information to adjust tile boundaries/sizes and avoid unnecessary restore (pulling in data from system memory to GMEM) or resolve (writing back to system memory from GMEM). 