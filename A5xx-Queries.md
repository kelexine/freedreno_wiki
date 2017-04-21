Occlusion and time-elapsed queries can be accumulated on the command-stream.

Occlusion Query
---------------

### Start:
```
    write RB_SAMPLE_COUNT_CONTROL (e1d1)
        RB_SAMPLE_COUNT_CONTROL: { COPY }
    write RB_SAMPLE_COUNT_ADDR_LO (e267)
        RB_SAMPLE_COUNT_ADDR_LO: 0xc0456000
        RB_SAMPLE_COUNT_ADDR_HI: 0x00000000
    opcode: CP_EVENT_WRITE (46) (2 dwords)
        { EVENT = ZPASS_DONE }
```
### During:
```
    GRAS_SC_CNTL |= SAMPLES_PASSED
    RB_RENDER_CNTL |= SAMPLES_PASSED
```
### Stop:
```
    opcode: CP_MEM_WRITE (3d) (5 dwords)
        gpuaddr:00000000c0456010
        ffffffff ffffffff
    opcode: CP_WAIT_MEM_WRITES (12) (1 dwords)
    write RB_SAMPLE_COUNT_CONTROL (e1d1)
        RB_SAMPLE_COUNT_CONTROL: { COPY }
    write RB_SAMPLE_COUNT_ADDR_LO (e267)
        RB_SAMPLE_COUNT_ADDR_LO: 0xc0456010
        RB_SAMPLE_COUNT_ADDR_HI: 0x00000000
    opcode: CP_EVENT_WRITE (46) (2 dwords)
        { EVENT = ZPASS_DONE }
    opcode: CP_WAIT_REG_MEM (3c) (7 dwords)
        0000: 70bc8006 00000014 c0456014 00000000 ffffffff ffffffff 00000010
        XXX why c0456014 instead of c0456010 ??
    opcode: CP_MEM_TO_MEM (73) (10 dwords)
        { NEG_C | DOUBLE }
        dst:  c0456008 00000000
        srcA: c0456008 00000000   <== result
        srcB: c0456010 00000000   <== stop-value
        srcC: c0456000 00000000   <== start-value
```
The `CP_MEM_TO_MEM` packet does `dst = srcA + srcB - srcC`, aka `result += stop-value - start-value`

Timestamp Query
---------------

### Start:
```
    opcode: CP_EVENT_WRITE (46) (5 dwords)
        { EVENT = CACHE_FLUSH_AND_INV_EVENT | 0x40000000 }
        { ADDR_0_LO = 0xc0456000 }
        { ADDR_0_HI = 0 }
        0xdead
            0000: 70460004 40000016 c0456000 00000000 0000dead
    opcode: CP_WAIT_FOR_IDLE (26) (1 dwords)
    opcode: CP_MEM_TO_MEM (73) (10 dwords)
        { NEG_C | DOUBLE | 0xc0000000 }
        dst:  c0456000 00000000
        srcA: c0456000 00000000
        srcB: c021c008 00000000
        srcC: c021c000 00000000
```

### Stop:
```
    opcode: CP_EVENT_WRITE (46) (5 dwords)
        { EVENT = CACHE_FLUSH_AND_INV_EVENT | 0x40000000 }
        { ADDR_0_LO = 0xc0456008 }
        { ADDR_0_HI = 0 }
        0xdead
            0000: 70460004 40000016 c0456008 00000000 0000dead
    opcode: CP_WAIT_FOR_IDLE (26) (1 dwords)
    opcode: CP_MEM_TO_MEM (73) (10 dwords)
        { NEG_C | DOUBLE | 0xc0000000 }
        dst:  c0456008 00000000
        srcA: c0456008 00000000
        srcB: c021c008 00000000
        srcc: c021c000 00000000
```
