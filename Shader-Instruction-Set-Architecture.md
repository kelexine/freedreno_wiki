# Instruction Set Architecture (ISA) Overview
The adreno GPU has a unified shader architecture, so the same instruction set and shader resources are used by both vertex (VS) and fragment/pixel (PS) shaders.

It bears some resemblances to the [r600](http://www.x.org/docs/AMD/r600isa.pdf) ISA, in that it is a VLIW architecture, with separation of control flow (CF) program and arithmetic and logic (ALU) / FETCH instructions.  But while the r600 ALU instructions consist of up to 5 scalar operations, the adreno ALU instruction consists of one vec4 operation and/or one scalar operation.

## Assembler syntax
The assembler syntax is loosely based on the [r600 assembler syntax](http://www.x.org/docs/AMD/R600-R700-Evergreen_Assembly_Language_Format.pdf), and also a single screenshot in [optimize-adreno.pdf](https://developer.qualcomm.com/download/optimize-adreno.pdf) (pg 6).  As with the r600 assembler syntax, the CF and ALU/FETCH instructions are interleaved for easier reading:
```
EXEC ADDR(0x3) CNT(0x1)
   (S)FETCH:	VERTEX	R1.xyz1 = R0.x FMT_32_32_32_FLOAT UNSIGNED STRIDE(12) CONST(10)
ALLOC COORD SIZE(0x0)
EXEC ADDR(0x4) CNT(0x1)
      ALU:	MAXv	export62 = R1, R1	; gl_Position
ALLOC PARAM/PIXEL SIZE(0x0)
EXEC_END ADDR(0x5) CNT(0x0)
```
While in memory, the actual layout is:
```
EXEC ADDR(0x3) CNT(0x1)
ALLOC COORD SIZE(0x0)
EXEC ADDR(0x4) CNT(0x1)
ALLOC PARAM/PIXEL SIZE(0x0)
EXEC_END ADDR(0x5) CNT(0x0)
   (S)FETCH:	VERTEX	R1.xyz1 = R0.x FMT_32_32_32_FLOAT UNSIGNED STRIDE(12) CONST(10)
      ALU:	MAXv	export62 = R1, R1	; gl_Position
```
The `ADDR` and `CNT` fields for `EXEC` and `EXEC_END` CF clauses refer to the offset (in multiples of 96bits) and instruction counts of the corresponding ALU/FETCH instructions.

## CF instructions
Each 96bit (3 dwords) CF instruction consists of two CF clauses.  The instruction format is:

<table>
  <tr><th>dword</th><th>bit position</th><th>description</th></tr>
  <tr><td rowspan=3>dword0</td>
      <td> 0..11</td><td>addr/size 1</td></tr>
  <tr><td>12..15</td><td>count 1</td></tr>
  <tr><td>16..31</td><td>sequence 1 - 2 bits per instruction in the <code>EXEC</code> clause, the low bit seems to control <code>FETCH</code> vs <code>ALU</code> instruction type, the high bit seems to be <code>(S)</code> modifier on instruction (which might make the name <code>SERIALIZE()</code> in optimize-for-adreno.pdf screenshot make sense.. although I don't quite understand the meaning yet)</td></tr>
  <tr><td rowspan=4>dword1</td>
      <td> 0..7 </td><td>UNKNOWN</td></tr>
  <tr><td>8..15?</td><td>CF opcode 1</td></tr>
  <tr><td>16..27</td><td>addr/size 2</td></tr>
  <tr><td>28..31</td><td>count 2</td></tr>
  <tr><td rowspan=3>dword2</td>
      <td> 0..15</td><td>sequence 2 - same as sequence 1 but for the 2nd CF clause</td></tr>
  <tr><td>16..23</td><td>UNKNOWN</td></tr>
  <tr><td>24..31</td><td>CF opcode 2</td></tr>
</table>

## ALU instructions
Each 96 bit ALU instruction can execute one vec4 operation, and/or one scalar operation.  Some instructions are only available as scalar or vector instructions.

An example of a combined vec4+scalar instruction in assembler syntax:
```
ALU:	MULv	R2.xyz_ = R3.zzzw, C10
  	RCP	R4.x___ = R0
```
To perform only a scalar operation, the vector operation should mask each channel in the vector dest (ie. `R0.____`)

Each ALU instruction can have up to 3 src registers.  The 3rd src register is either used for a 3 op vector instruction like `MULADDv` or for the paired scalar instruction.

<table>
  <tr><th>dword</th><th>bit position</th><th>description</th></tr>
  <tr><td rowspan=9>dword0</td>
      <td> 0..5? </td><td>vector dest register</td></tr>
  <tr><td>6?..7  </td><td>UNKNOWN</td></tr>
  <tr><td>8..13? </td><td>scalar dest register</td></tr>
  <tr><td> 14    </td><td>UNKNOWN</td></tr>
  <tr><td> 15    </td><td>export flag</td></tr>
  <tr><td>16..19 </td><td>vector dest write mask (wxyz, 1 bit per channel)</td></tr>
  <tr><td>20..23 </td><td>scalar dest write mask (same as above)</td></tr>
  <tr><td>24..26 </td><td>UNKNOWN</td></tr>
  <tr><td>27..31 </td><td>scalar operation</td></tr>
  <tr><td rowspan=9>dword1</td>
      <td> 0..7  </td><td>src3 swizzle</td></tr>
  <tr><td> 8..15 </td><td>src2 swizzle</td></tr>
  <tr><td>16..23 </td><td>src1 swizzle</td></tr>
  <tr><td>  24   </td><td>src3 negate</td></tr>
  <tr><td>  25   </td><td>src2 negate</td></tr>
  <tr><td>  26   </td><td>src1 negate</td></tr>
  <tr><td>  27   </td><td>predicate case (1 - execute if true, 0 - execute if false)</td></tr>
  <tr><td>  28   </td><td>predicate (conditional execution)</td></tr>
  <tr><td>29..31 </td><td>UNKNOWN</td></tr>
  <tr><td rowspan=13>dword2</td>
      <td> 0..5? </td><td>src3 register</td></tr>
  <tr><td>  6    </td><td>UNKNOWN</td></tr>
  <tr><td>  7    </td><td>src3 abs (assumed)</td></tr>
  <tr><td> 8..13?</td><td>src2 register</td></tr>
  <tr><td>  14   </td><td>UNKNOWN</td></tr>
  <tr><td>  15   </td><td>src2 abs</td></tr>
  <tr><td>16..21?</td><td>src1 register</td></tr>
  <tr><td>  22   </td><td>UNKNOWN</td></tr>
  <tr><td>  23   </td><td>src1 abs</td></tr>
  <tr><td>24..28 </td><td>vector operation</td></tr>
  <tr><td>  29   </td><td>src3 type/bank (1 - Register bank (R), varyings and locals; 0 - Constant bank (C), uniforms and consts</td></tr>
  <tr><td>  30   </td><td>vector src2 type/bank (same as above)</td></tr>
  <tr><td>  31   </td><td>vector src1 type/bank (same as above)</td></tr>
</table>

Interpretation of ALU src swizzle fields

<table>
  <tr><td>1..0</td><td colspan=2>chan[0] (x) swizzle</td></tr>
  <tr><td></td><td>00</td><td>x</td></tr>
  <tr><td></td><td>01</td><td>y</td></tr>
  <tr><td></td><td>10</td><td>z</td></tr>
  <tr><td></td><td>11</td><td>w</td></tr>
  <tr><td>3..2</td><td colspan=2>chan[1] (y) swizzle</td></tr>
  <tr><td></td><td>11</td><td>x</td></tr>
  <tr><td></td><td>00</td><td>y</td></tr>
  <tr><td></td><td>01</td><td>z</td></tr>
  <tr><td></td><td>10</td><td>w</td></tr>
  <tr><td>5..4</td><td colspan=2>chan[2] (z) swizzle</td></tr>
  <tr><td></td><td>10</td><td>x</td></tr>
  <tr><td></td><td>11</td><td>y</td></tr>
  <tr><td></td><td>00</td><td>z</td></tr>
  <tr><td></td><td>01</td><td>w</td></tr>
  <tr><td>7..6</td><td colspan=2>chan[3] (w) swizzle</td></tr>
  <tr><td></td><td>01</td><td>x</td></tr>
  <tr><td></td><td>10</td><td>y</td></tr>
  <tr><td></td><td>11</td><td>z</td></tr>
  <tr><td></td><td>00</td><td>w</td></tr>
</table>

...


## FETCH instructions
The FETCH instruction is also 96 bit, but can fetch one vec4 vertex value or one vec4 texture sample value.

...

## Example
Here is a more complete example.

GLSL format:
```c
uniform mat4 modelviewMatrix;
uniform mat4 modelviewprojectionMatrix;
uniform mat3 normalMatrix;

attribute vec4 in_position;
attribute vec3 in_normal;
attribute vec4 in_color;

vec4 lightSource = vec4(2.0, 2.0, 20.0, 0.0);

varying vec4 vVaryingColor;

void main()
{
    gl_Position = modelviewprojectionMatrix * in_position;
    vec3 vEyeNormal = normalMatrix * in_normal;
    vec4 vPosition4 = modelviewMatrix * in_position;
    vec3 vPosition3 = vPosition4.xyz / vPosition4.w;
    vec3 vLightDir = normalize(lightSource.xyz - vPosition3);
    float diff = max(0.0, dot(vEyeNormal, vLightDir));
    vVaryingColor = vec4(diff * in_color.rgb, 1.0);
}
```
and commented shader assembly:
```
 ;;;; const/register assignment:
 ; R0: vVaryingColor
 ; R1, CONST(1): in_color
 ; R3, CONST(2): in_normal
 ; R2, CONST(3): in_position
 ; C0+: modelviewMatrix
 ; C4+: modelviewprojectionMatrix
 ; C8+: normalMatrix
 ; C11: 2.000000, 2.000000, 20.000000, 0.000000
 ; C12: 1.000000, 0.000000, 0.000000, 0.000000
EXEC
      FETCH:  VERTEX  R1.xyz_ = R0.z FMT_32_32_32_FLOAT SIGNED STRIDE(12) CONST(4)
      FETCH:  VERTEX  R2.xyz1 = R0.x FMT_32_32_32_FLOAT SIGNED STRIDE(12) CONST(4)
      FETCH:  VERTEX  R3.xyz_ = R0.y FMT_32_32_32_FLOAT SIGNED STRIDE(12) CONST(4)
   (S)ALU:    MULv    R0 = R2.wwww, C7           ; -> modelviewprojectionMatrix * in_position
      ALU:    MULADDv R0 = R0, R2.zzzz, C6       ; -> modelviewprojectionMatrix * in_position
      ALU:    MULADDv R0 = R0, R2.yyyy, C5       ; -> modelviewprojectionMatrix * in_position
ALLOC COORD SIZE(0x0)
EXEC
      ALU:    MULADDv export62 = R0, R2.xxxx, C4 ; gl_Position = modelviewprojectionMatrix * in_position
      ALU:    MULv    R0 = R2.wwww, C3           ; -> modelviewMatrix * in_position
      ALU:    MULADDv R0 = R0, R2.zzzz, C2       ; -> modelviewMatrix * in_position
      ALU:    MULADDv R0 = R0, R2.yyyy, C1       ; -> modelviewMatrix * in_position
      ALU:    MULADDv R0 = R0, R2.xxxx, C0       ; vec4 vPosition4 = modelviewMatrix * in_position
      ALU:    MULv    R2.xyz_ = R3.zzzw, C10     ; -> normalMatrix * in_normal
              RCP     R4.x___ = R0               ; -> 1 / vPosition4.w
EXEC
      ALU:    MULADDv R0.xyz_ = C11.xxzw, -R0, R4.xxxw ; -> lightSource - (vPosition4.xyz / vPosition4.w
      ALU:    DOT3v   R4.x___ = R0, R0                 ; -> normalize(...)
      ALU:    MULADDv R2.xyz_ = R2, R3.yyyw, C9        ; -> normalMatrix * in_normal
      ALU:    MULADDv R2.xyz_ = R2, R3.xxxw, C8        ; vec3 vEyeNormal = normalMatrix * in_normal
ALLOC PARAM/PIXEL SIZE(0x0)
EXEC_END
      ALU:    MAXv    R0.____ = R0, R0
              RSQ     R0.___w = R4.xyzx          ; -> normalize(...)  (1 / sqrt(dot(..))
      ALU:    MULv    R0.xyz_ = R0, R0.wwww      ; -> vec3 vLightDir = normalize(lightSource.xyz - vPosition3)
      ALU:    DOT3v   R0.x___ = R2, R0           ; -> dot(vEyeNormal, vLightDir)
      ALU:    MAXv    R0.x___ = R0, C11.wyzw     ; float diff = max(0.0, dot(vEyeNormal, vLightDir))
      ALU:    MULv    export0.xyz_ = R1, R0.xxxw ; vVaryingColor.xyz_= diff * in_color.rgb
              MOV     export0.___w = C12.xyzx    ; vVaryingColor.___w  = 1.0
```