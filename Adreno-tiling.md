Like most (all but tegra?) of the embedded/SoC gpu's, adreno is a tiling based architecture.  However the way that it is implemented is a bit simpler.

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

### Performance Notes:

As with most/all tilers, switching render target is expensive and triggers a flush.  Also, at least with freedreno gallium driver, if you are only rendering part of a buffer it is recommended to scissor out parts of the buffer that won't be touched.  The gallium driver can use this information to adjust tile boundaries/sizes and avoid unnecessary restore (pulling in data from system memory to GMEM) or resolve (writing back to system memory from GMEM). 