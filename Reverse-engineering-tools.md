The [freedreno](http://github.com/freedreno/freedreno) and [envytools](http://github.com/freedreno/envytools) git trees contains a number of useful tools, and simple "test" programs for collecting cmdstream traces from blob.

## Guided Tour
### freedreno.git
This tree contains mostly target side code.

* tests-2d - simple 2d tests using blob `libC2D2.so`.  Links against android bionic glibc and dynamic loader.
* tests-3d - simple 3d GLES2 tests.  Can also be built for glibc to run against gallium driver for comparison/debugging.  When built by default to link against bionic, rendering happens to a pbuffer so you don't actually need an android UI
* tests-cl - a few simple OpenCL examples
* wrap - contains the src for `libwrap.so` and `libwrapfake.so` which can be `LD_PRELOAD`d to intercept cmdstream for 2d or 3d tests.
* opencl
* fdre-a2xx - simple gl-like API and tests for a2xx.  Not intended to be a complete replacement for GL, but the simple tests and shaders written in assembly provide an easier environment for experimentation.
* fdre-a3xx - same as above but for a3xx.

Normally libwrap and tests-* are built for android using `ndk-build.sh`.  To some extent (not tested much) they can be built for glibc/linux with `make BUILD=glibc tests-3d`.

### envytools.git
This tree contains host side tools, as well as the rnndb register databases.

* afuc - a5xx+ firmware assembler/disassembler
* cffdump/
   * cffdump - cmdstream parsing tool
   * pgmdump/pgmdump2 - parser for `GetProgramBinaryOES()` output (pgmdump2 handles the newer format used by modern blob drivers)
* rnn/
   * headergen2 - generates header files used by gallium driver, kernel, etc
   * librnn - register decoding lib used by cffdump
   * demsm - parsing tool that can be used w/ kernel traces of register read/write (in particular useful for display, but also can be used for gpu).  The `CONFIG_DRM_MSM_REGISTER_LOGGING` option for msm drm/kms driver can be used build in support for register tracing
* rnndb/ - the xml register database files for gpu and display

## Capturing cmdstream traces

The cmdstream from the userspace driver is captured into an `.rd` file, which is a simple binary file consisting of arbitrary number of sections, where each section has a type, length, and value.  The first level cmdstream buffer is captured, and known/important gpu buffers are snapshot'd along with their gpu address.  (cffdump needs this to follow IB's and other pointers in the cmdstream.)

Note that cmdstream traces compress well, and `cffdump`/etc can read `.rd.gz` files.

### downstream kgsl kernel driver:

Either `libwrap.so` (which shim's the kernel interface) or `libwrapfake.so` (which emulates the kernel interface) can be `LD_PRELOAD`d to capture cmdstream from blob android drivers which use the kgsl (`drivers/gpu/msm`) kernel driver.  For `libwrapfake.so` an extra couple of environment variables, `$WRAP_GPU_ID` and `$WRAP_GMEM_SIZE` are used to control which GPU is emulated:

```
export WRAP_GPU_ID=540.0
export WRAP_GMEM_SIZE=0x100000

TESTNUM=0 LD_PRELOAD=`pwd`/libwrapfake.so ./test-quad-flat
```

### upstream drm/msm kernel driver:

For msm drm/kms driver you can simply use:
```
  # cat /sys/kernel/debug/dri/0/rd > logfile.rd
```
The moduleparam `msm.rd_full=1` will capture not just the cmdstream buffers but also ancillary buffers.  This is useful if, for example, you want to see the shaders (which are normally emitted as pointers in the cmdstream to other buffers).

Newer kernels also have a `hangrd` file in debugfs to capture cmdstream from only submits which cause GPU hangs, which is useful when (for example) debugging games that trigger GPU crashes.

## Tools
### cffdump
This tool decodes commandstream captured in .rd file.  It knows how to decode PM4 packets, follow branches in cmdstream (IB's), etc.  And uses librnn to decode register write packets, and disasm to decode shaders.  Useful arguments are:
* `--no-color` - generate boring output
* `--summary` - don't display every register write, but just summary of current register writes on each draw (`DRAW_INDX`) packet.  Typically the order of register writes is not important, the more useful thing is just to know the current values on a draw call.  Other packages, such as const-state write (uniforms, shaders, texture state) are still displayed.
* `--allregs` - display all written registers in summary (don't ignore ones with zero values)
* `-q <reg>` - dump the value of a register (can be specified either by name or offset) at each draw/blit.  This can be specified multiple times to dump multiple registers.
* `--script` - specify a script to be run at each draw/blit

(See `--help` for description of all of the arguments)

Some example .rd files can be found [here](http://people.freedesktop.org/~robclark/a3xx/).

Often a visual diff tool like meld is useful to compare different similar dumps from cffdump, to spot which register(s) have different settings:

![cmdstream-diff](http://freedreno.github.io/images/cmdstream-diff-small.png)

#### scripting

A lua script interface is provided to allow for more advanced analysis.

```lua
io.write("HELLO WORLD\n")

r = rnn.init("a320")

function start_cmdstream(name)
  io.write("START: " .. name .. "\n")
end

function draw(primtype, nindx)
  io.write("DRAW: " .. primtype .. ", " .. nindx .. "\n")
  io.write("GRAS_CL_VPORT_XOFFSET: " .. r.GRAS_CL_VPORT_XOFFSET .. "\n")
  io.write("RB_MRT[0].CONTROL.ROP_CODE: " .. r.RB_MRT[0].CONTROL.ROP_CODE .. "\n")
  io.write("SP_VS_OUT[0].A_COMPMASK: " .. r.SP_VS_OUT[0].A_COMPMASK .. "\n")
  io.write("RB_DEPTH_CONTROL.Z_ENABLE: " .. tostring(r.RB_DEPTH_CONTROL.Z_ENABLE) .. "\n")
  io.write("0x2280: written=" .. regs.written(0x2280) .. ", lastval=" .. regs.lastval(0x2280) .. ", val=" .. regs.val(0x2280) .. "\n")
end

function end_cmdstream()
  io.write("END\n")
end

function finish()
  io.write("FINISH\n")
end
```
The `start_cmdstream()`/`end_cmdstream()` are called at he start an end of each cmdstream file, `finish()` is called at the end of the last cmdstream file, and `draw()` is called for each draw or blit.  The `r` object provides access to parsed registers and bitfields at the time of the draw/blit.

### pgmdump
The tests-3d tests use the "GetProgramBinaryOES" extension to capture the shaders as generated by the compiler, which can be dumped with the pgmdump tool.  

For a2xx the shaders are patched at runtime for vertex/attribute location/type.  This is not needed for a3xx.  But in both cases, alternate versions of the vertex shader are generated for [[binning/tiling|Adreno-Tiling]].

For a3xx, the `--expand` argument will expand out the repeat instructions.
