Note: see [instr-a3xx.h](https://github.com/freedreno/freedreno/blob/master/includes/instr-a3xx.h) for all the known instruction opcodes and encodings.

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
* `sam (f32)(xyzw)Rdst, Rsrc, s#0, t#0`

These are the only instructions that write to and read from more than one scalar register.  In particular they write to up to four (as controlled by writemask) successive scalar registers.  The texture coordinate (`Rsrc`) should contain two or three successive coordinates (for normal 2d texture fetch, vs 3d texture fetch, etc)

Other non-cat5 instructions which read from a register written by a cat5 instruction must have the `(sy)` bit set to sync with the texture-fetch pipeline.

### Category 6
Load/Store instructions to private/local/global memory, atomic add/sub/exchange/etc, and other misc instructions.  Mostly useful for opencl.

## Assembly Syntax

### Special Registers
* `a0` - address register, numerically 61
* `p0` - predicate register, numerically 62

## Scheduling

## Const register file and src register constraints

## Relative Addressing
