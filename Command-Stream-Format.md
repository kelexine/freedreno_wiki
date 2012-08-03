# Overview
The adreno GPU traces it's lineage back to ATI's [Imageon](http://en.wikipedia.org/wiki/Imageon) line of mobile GPUs intended for SoC application.  As such, it shares some similarities with the [Radeon](http://en.wikipedia.org/wiki/Radeon) GPUs.  In particular, the CP (command processor) appears to be identical.  However, none of the op-codes or register addresses appear to the same.

## Command Processor (CP)
This is the easiest part, as it is identical to radeon.  The CP is the block that reads a series of rendering commands from a ringbuffer (the PM4 command stream) and either sets some register values or triggers some rendering action.  It consists primarily of Type-0 (PKT0) and Type-3 (PKT3) commands

### Type-0 (PKT0)
Writes N consecutive (32bit) dwords to N registers, starting with the register specified by `BASE_INDEX`.

<table>
 <tr><th width="6.25%">31..32</th><th width="43.75%">29..16</th><th width="50%">15..0</th></tr>
 <tr><td align="center">`00`</td><td align="center">COUNT</td><td align="center">BASE_INDEX</td></tr>
 <tr><td colspan="3" align="center">REG_DATA_1</td></tr>
 <tr><td colspan="3" align="center">REG_DATA_2</td></tr>
 <tr><td colspan="3" align="center">...</td></tr>
 <tr><td colspan="3" align="center">REG_DATA_n</td></tr>
</table>

### Type-3 (PKT3)
Perform the operation specified by `IT_OPCODE`.

<table>
 <tr><th width="6.25%">31..32</th><th width="43.75%">29..16</th><th width="25%">15..8</th><th width="21.875%">7..1</th><th width="3.125%">0</th></tr>
 <tr><td align="center">`11`</td><td align="center">COUNT</td><td align="center">IT_OPCODE</td><td align="center">Reserved</td><td align="center">P</td></tr>
 <tr><td colspan="5" align="center">DATA_1</td></tr>
 <tr><td colspan="5" align="center">DATA_2</td></tr>
 <tr><td colspan="5" align="center">...</td></tr>
 <tr><td colspan="5" align="center">DATA_n</td></tr>
</table>

A few common actions are:
* `CP_SET_CONSTANT:0x2d` - performs basically the same thing as a `PKT0`.. `DATA_1` is `0x00040000 | (BASE_INDEX - 0x2000)`, and `DATA_2` through `DATA_n` are N-1 register values
* `CP_INDIRECT_BUFFER:0x3f` - branch to a 2nd cmdstream buffer
* `CP_DRAW_INDX:0x22` - trigger drawing

