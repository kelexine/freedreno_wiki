# Instruction Set Architecture (ISA) Overview
The adreno GPU has a unified shader architecture, so the same instruction set and shader resources are used by both vertex (VS) and fragment/pixel (PS) shaders.

It bears some resemblances to the [r600](http://www.x.org/docs/AMD/r600isa.pdf) ISA, in that it is a VLIW architecture, with separation of control flow (CF) program and arithmetic and logic (ALU) instructions.  But while the r600 ALU instructions consist of up to 5 scalar operations, the adreno ALU instruction consists of one vec4 operation and/or one scalar operation.

## Assembler syntax
The assembler syntax is loosely based on the r600 [assembler syntax](http://www.x.org/docs/AMD/R600-R700-Evergreen_Assembly_Language_Format.pdf), and also a single screenshot in [optimize-adreno.pdf](https://developer.qualcomm.com/download/optimize-adreno.pdf) (pg 6).  As with the r600 assembler syntax, the CF and ALU instructions are interleaved for easier reading:
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
The `ADDR` and `CNT` fields for `EXEC` and `EXEC_END` CF clauses refer to the offset (in multiples of 96bits) and instruction counts of the corresponding ALU instructions.

## CF instructions

## ALU instructions
