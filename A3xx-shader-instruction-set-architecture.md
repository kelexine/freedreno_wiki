Note: see [instr-a3xx.h](https://github.com/freedreno/freedreno/blob/master/includes/instr-a3xx.h) for all the known instruction opcodes and encodings.

Note: this page applies to a4xx and a5xx with some very minor differences.  In the mesa code, the instruction set and shader compiler for a3xx/a4xx/a5xx (and beyond?) is referred to as "ir3".

## Instruction Set Architecture (ISA) Overview
Unlike the [[a2xx shader ISA|A2XX-Shader-Instruction-Set-Architecture]], the a3xx uses a "simple" scalar instruction set, but with some tricks.  And the compiler needs to be a bit more aware about scheduling and some other constraints.

Each instruction is 64bits (qword), and there are 7 basic instruction encodings or "categories" (which in some cases have multiple sub-encodings):

There is no separate CF vs FETCH/ALU program, as with [[a2xx|A2XX-Shader-Instruction-Set-Architecture]].  But some categories of instructions run asynchronously and need special synchronization to deal with read-after-write (or write-after-read?).  And unlike a2xx, there are now both full and half (16bit) registers which are *not* aliased.  And instructions not just for floating point, but also integer.

### Category 0
Generally flow control instructions that take zero arguments (sometimes with an embedded constant).  For example: 
* `nop`
* `jump` - unconditional jump to immediate (encoded in instruction) offset
* `br`anch - predicated (on predicate register, `p0`) jump to immediate offset

### Category 1
Variants of move/convert (single src register).  There are no op-codes for cat1 instructions.  Although the shader mnemonics differ depending on the src and destination type.  If the src and destination type are the same, it is called a move:
* `mov.f16f16 Rdst, Rsrc` - move from same type src and dst
* `cov.f32u16 Rdst, Rsrc` - move/convert from f32 src to u16 dst
* `mova` -  is a `mov.f16f16` to the address register (`a0`), see section on relative addressing for more.
(in all cases, the src register can be const)

### Category 2
Normal ALU instructions, typically with 2 src registers, although there are a handful of cases where the 2nd src encoded is ignored:
* `add.f Rdst, Rsrc0, Rsrc1`
* `and.b Rdst, Rsrc0, Rsrc1`
* `floor.f Rdst, Rsrc0` - an example of cat2 which ignores the 2nd src.

The full set of cat2 instructions which use only the 1st src register is:
* `absneg.f`, `absneg.s`, `clz.b`, `cls.s`, `sign.f`, `floor.f`, `ceil.f`, `rndne.f`, `rndaz.f`, `trunc.f`, `not.b`, `bfrev.b`, `setrm`, and `cbits.b`

Interestingly, cat2 also includes the varying fetch/interpolate instructions (`bary.f`).  Although I guess the latency for these should not be high compared to texture fetch from external memory.

### Category 3
Three src register operations, such as:
* `mad.f16` - src0 * src1 + src2
* `sel.f32` - src1 ? src0 : src2

### Category 4
Complex single src operations, which take more cycles (potentially unpredictable number) and/or are more asynchronous compared to cat1-cat3:
* `rcp Rdst, Rsrc`
* `log2 Rdst, Rsrc`

Other non-cat4 instructions which read from a register written by a cat4 instruction must have the `(ss)` bit set to sync to the complex-alu pipeline.
### Category 5
Generally texture sample related instructions:
* `sam (f32)(xyzw)Rdst, Rsrc0, Rsrc1, s#0, t#0`
* `isam (f32)(xyzw)Rdst, Rsrc0, Rsrc1, s#0, t#0`
* `samgq (f32)(xyzw)Rdst, Rsrc0, Rsrc1, s#0, t#0`
* `isam (f32)(xyzw)Rdst, Rsrc0, Rsrc1, Rsrc2`

These are the only instructions that write to and read from more than one scalar register.  In particular they write to up to four (as controlled by writemask) successive scalar registers.

Each instruction can also have different flags:
* `.l` - explicit LOD
* `.b` - LOD bias
* `.a` - array texture
* `.3d` - 3d/cube texture
* `.o` - offset
* `.s` - shadow comparator
* `.p` - projector

The `isam` variant expects integer coordinates, while the `sam` variant expects float coordinates. (Note that `isam` will use unnormalized coordinates even if the sampler says otherwise.) Since `isam` wants integer coordinates, they must be provided wrt LOD 0 size, even if specifying a specific LOD. Lastly, if sampling from an integer texture, `(f32)` will cause that integer to be converted to a float, so to obtain correct results, `(u32)` must be used.

`Rsrc0` points to the first register of the sequence: x, y, (`.3d`?) z, (`.s`?) shadow, (`.a`?) array, (`.p`?) projector, and for `samgq`, starting at offset 4 (i.e. padding if not enough flags to force it), dpdx.xy, dpdy.xy

`Rsrc1` points to the first register of the sequence: (`.o`?) offset.xy, (`.3d.o`?) offset.z, (`.l`?) lod, (`.b`?) bias. Note that if not using a `sam.l` or `sam.b` variant, and offset isn't set, `Rsrc1` is omitted entirely.

`Rsrc2` points to the first register of the sequence: texture#, (untested) sampler#. s2en bit enables Rsrc2.

Other non-cat5 instructions which read from a register written by a cat5 instruction must have the `(sy)` bit set to sync with the texture-fetch pipeline.

### Category 6
Load/Store instructions to private/local/global memory, atomic add/sub/exchange/etc, and other misc instructions.  Mostly useful for opencl.

## Assembly Syntax
General purpose registers (GPRs) are denoted in vec4 syntax, ie. `r1.y` refers to the y'th component of 2nd vec4 register.  But for the most part there are not any constraints about treating the registers as vec4.  You could just as easily think of `r1.y` as `r5` ((4 * 1) + 1).  Inputs and outputs to shaders do not need to be vec4 aligned, for example a varying output of a vertex shader can occupy `r2.w` through `r3.z`.  But assignment of registers to a shader thread happen on granularity of vec4.

A half-width register is denoted as `hr2.z`.  And similarly a full or half-width const file register (uniforms/immediates) is, for example, `c3.z` or `hc5.x`.

An example fragment shader from the disassembler:
```
0000[57305b00x_00002000x] (sy)(ss)(rpt3)bary.f hr0.x, (r)0, r0.x
0001[47304b04x_00002004x] (rpt3)bary.f hr1.x, (r)4, r0.x
0002[40080b04x_00044000x] (rpt3)add.f hr1.x, (neg)(r)hr0.x, (r)hr1.x
0003[4730c808x_00002008x] bary.f (ei)hr2.x, (r)8, r0.x
0004[00000200x_00000000x] (rpt2)nop
0005[63020300x_20008008x] (rpt3)mad.f16 hr0.x, hr2.x, (r)hr1.x, (r)hr0.x
0006[03000000x_00000000x] end
```
The first column shows the instruction index and hex encoding.

The `(rptN)` syntax sets the repeat field, in which case the instruction is executed N more times with successive destination registers.  Src registers with the `(r)`epeat flag set are also incremented.  This serves mainly to increase code density, so in typical cases a vec4 instruction can be encoded in a single 64bit instruction.  The above shader with repeat instructions expanded (the `--expand` argument to pgmdump) to show what the shader actually executes:
```
0000[57305b00x_00002000x] (sy)(ss)(rpt3)bary.f hr0.x, (r)0, r0.x
0000[                   ] bary.f hr0.y, (r)1, r0.x
0000[                   ] bary.f hr0.z, (r)2, r0.x
0000[                   ] bary.f hr0.w, (r)3, r0.x
0001[47304b04x_00002004x] (rpt3)bary.f hr1.x, (r)4, r0.x
0001[                   ] bary.f hr1.y, (r)5, r0.x
0001[                   ] bary.f hr1.z, (r)6, r0.x
0001[                   ] bary.f hr1.w, (r)7, r0.x
0002[40080b04x_00044000x] (rpt3)add.f hr1.x, (neg)(r)hr0.x, (r)hr1.x
0002[                   ] add.f hr1.y, (neg)(r)hr0.y, (r)hr1.y
0002[                   ] add.f hr1.z, (neg)(r)hr0.z, (r)hr1.z
0002[                   ] add.f hr1.w, (neg)(r)hr0.w, (r)hr1.w
0003[4730c808x_00002008x] bary.f (ei)hr2.x, (r)8, r0.x
0004[00000200x_00000000x] (rpt2)nop
0004[                   ] nop
0004[                   ] nop
0005[63020300x_20008008x] (rpt3)mad.f16 hr0.x, hr2.x, (r)hr1.x, (r)hr0.x
0005[                   ] mad.f16 hr0.y, hr2.x, (r)hr1.y, (r)hr0.y
0005[                   ] mad.f16 hr0.z, hr2.x, (r)hr1.z, (r)hr0.z
0005[                   ] mad.f16 hr0.w, hr2.x, (r)hr1.w, (r)hr0.w
0006[03000000x_00000000x] end
```

### Special Registers
* `a0` - address register, numerically 61
* `p0` - predicate register, numerically 62

## Scheduling
The compiler is responsible for taking into account the number of (instruction dispatch) cycles before a destination register of a previous instruction is ready.  For cat1-cat3 instructions, the destination register is available three instructions later.  The compiler is responsible for inserting `nop` instructions if needed.

For a destination register written by a cat4 or cat5 instruction, the `(ss)` or `(sy)` bit can be set to synchronize.  This is because, unlike cat1-cat3, the number of cycles needed to complete is not predictable.

In particular, for cat3 instructions, the 3rd src register is not needed until the 2nd cycle, so for example a DP4 (dot product) instruction can be implemented as:
```
; DP4 r0.x, r2.xyzw, r3.xyzw:
mul.f r0.x, r2.x, r3.x
nop
mad.f32 r0.x, r2.y, r3.y, r0.x
nop
mad.f32 r0.x, r2.z, r3.z, r0.x
nop
mad.f32 r0.x, r2.w, r3.w, r0.x
```
rather than needing two nop's for the result of the previous instruction to be available.  The compiler can of course schedule unrelated instructions in those `nop` slots if possible.

An instruction writing a register which was previously a src for texture sample instruction (WAR hazard) needs the `(ss)` bit set.

## Branches / Flow Control
Typically simple if/else constructs will be flattened out, with all legs of the branch executed, and then `sel` instructions used to conditionally write back the results from the leg of the branch that was "taken" from a flow control point of view.  Divergent flow control amonst threads is typically expensive (ie. hardware ends up having to execute one thread at a time within the thread-group), and so a compiler will normally try to avoid it.

(Currently if/else is implemented with branches in gallium driver, simply because the compiler is not clever enough to figure out how to flatten it.)

When branches are needed, they can be implemented with cat0 instructions, for example an if/else could naively be achieved by:
```
cmps.f.eq p0.x, hr1.x, hc2.x
br p0.x, #6
mov.f16f16 hr1.x, hc2.x
mov.f16f16 hr1.y, hc2.y
mov.f16f16 hr1.z, hc2.x
mov.f16f16 hr1.w, hc2.y
jump #6
(jp)nop
mov.f16f16 hr1.x, hc2.y
mov.f16f16 hr1.y, hc2.x
mov.f16f16 hr1.z, hc2.x
mov.f16f16 hr1.w, hc2.y
(jp)nop
```
Note that the branch target instructions have a `(jp)` (jump-target) flag set on them.  Presumably this helps the thread scheduler figure out the potential convergence points.  The jump target need not be a `nop`.  Branches can be forward (positive) or backwards (negative) immediate offset.

## Const register file and src register constraints
There are limitations about use of const src arguments for instructions, and in some cases the compiler will need to move a const into a GPR.  Known limitations are:
* cat2 can take at most one const src (but can be in either position)
* cat3 cannot take a const src as 2nd argument (src1)
* cat4 cannot take a const src

## Relative Addressing
For cat1 instructions, there is a bit indicating relative src, and a second bit to indicate const or gpr register file.  A write to the address register seems to need 5 instruction slots before it can be used.  Seems like the `(ul)` flag indicates the last relative instruction.  For example, the following shader:
```
precision mediump float;
precision mediump int;
uniform int idx;
uniform mat4 m[4];

void main()
{
	gl_FragColor = m[1][idx];
}
```
will result in the following snippet for the address relative load:
```
mova a0.x, hr0.x
; need 5 cycles 
(rpt5)nop 
mov.f32f32 r0.x, c<a0.x + 16>
mov.f32f32 r0.y, c<a0.x + 17>
(ul)mov.f32f32 r0.z, c<a0.x + 18>
mova a0.x, hr0.x
cov.f32f16 hr0.x, r0.x
cov.f32f16 hr0.y, r0.y
cov.f32f16 hr0.z, r0.z
(rpt2)nop
(ul)mov.f32f32 r0.x, c<a0.x + 19>
```
