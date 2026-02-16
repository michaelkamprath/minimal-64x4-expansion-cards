
# Clock Counter
This Clock Counter card provides a convenient way to capture precise timing on the Minimal 64x4 by counting the number of clock cycles. The counter is resetable to 0, thus allowing the timing of specific operations. The counter is 48-bits wide, thus allowing captureing time ranges up to 1.1 years at the Minimal 64x4's 8MHz clock frequency.

## Memory Map

| Address | R/W   | Function       | Description |
| ------- | ----- | -------------- | ------------|
| `$FEx0` + `y` | Read  | Latched Byte 0 | Read latched counter value, byte 0 (LSB) |
| `$FEx1` + `y` | Read  | Latched Byte 1 | Read latched counter value, byte 1 |
| `$FEx2` + `y` | Read  | Latched Byte 2 | Read latched counter value, byte 2 |
| `$FEx3` + `y` | Read  | Latched Byte 3 | Read latched counter value, byte 3 |
| `$FEx4` + `y` | Read  | Latched Byte 4 | Read latched counter value, byte 4 |
| `$FEx5` + `y` | Read  | Latched Byte 5 | Read latched counter value, byte 5 (MSB) |
| `$FEx6` + `y` | Write | Latch Counter  | Latch (snapshot) the current 48-bit counter into the output registers. Any value may be written. |
| `$FEx7` + `y` | Write | Reset Counter  | Reset the counter value to `0`. Any value may be written. |

Notes:
* `x` is the jumper-selectable high nibble of the address low byte (selects the 16-byte slot). Set by Jumpers `J2`-`J5`, where `J2` is the least signficant bit and `J5` is the most sigficant bit. Bidging all jumpers `J2` through `J5` would set `x` to `$F`. Altogether, `$FEx` is the address prefix.
* `y` is the position offsett within the selected address prefix. Set by Jumper `J1`. With `J1` open, `y` is equal to 0. With `J1` bridged, `y` is equal to 8.
* Reading $FEx0–$FEx5 (+`y`) returns the last value latched by writing to `$FEx6` (+`y`), not necessarily the current counter value.
* Resetting the counter by writing to `$FEx7` (+`y`) does not latch a `0` value.

## Time Range
This card has a 48-bit counter which is read 1 byte at a time. Not all bytyes need to be read, but in doing so the effective time range captured is limitted. At the **Minimal 64x4**\'s 8 Mhz clock, the maximum time ranges are showing in this table:

| Bytes | Bits | Max Seconds (s) | Approx Time                      |
| ----: | ---: | --------------: | -------------------------------- |
|     6 |   48 | 35184372.088832 | 407.22652996 days (~1.115 years) |
|     5 |   40 |   137438.953472 | 38.17748708 hours (~1.5907 days) |
|     4 |   32 |      536.870912 | 8.94784853 minutes               |
|     3 |   24 |        2.097152 | 2.097152 seconds                 |
|     2 |   16 |        0.008192 | 8.192 ms                         |
|     1 |    8 |        0.000032 | 32 µs                            |


The latched bytes are arranged in little-endian order starting at `$FEx0` for the LSB to `$FEx5` for the MSB (`x` is determined by the address jumper settings on the card).