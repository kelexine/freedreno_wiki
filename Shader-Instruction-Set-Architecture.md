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

...


## FETCH instructions
The FETCH instruction is also 96 bit, but can fetch one vec4 vertex value or one vec4 texture sample value.

...

