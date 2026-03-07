
# Timer - Clock Counter
This Timer card provides a convenient way to capture precise timing on the Minimal 64x4 by counting clock cycles. The counter is resettable to 0, allowing specific operations to be timed. The counter is 48 bits wide, which allows time ranges up to about 1.1 years at the Minimal 64x4's 8 MHz clock frequency.

## Hardware Design
[This card](./hardware/) uses six 74HC590 8-bit counters arranged in a cascading carry chain to create a 48-bit counter. The 74HC590 has tri-state outputs, so each counter IC is connected directly to the data bus and remains in high-Z unless selected to drive its latched value. Card control is handled by two 74HC688 comparators (address decode) and an ATF22V10C programmable logic device (control logic for read byte select, latch, and reset). The combinational logic is captured in [the PLD code](./pld-files/).

## Memory Map

Define:

`base = $FE00 + (x << 4) + y`

Where:
- `y` is controlled by jumper `J1` (address bit A3): `J1` open => `y = 0`, `J1` bridged => `y = 8`.
- `x` is controlled by jumpers `J2`-`J5` (address bits A4-A7): `J2` is A4 (LSB of `x`) and `J5` is A7 (MSB of `x`).
- Jumpers are named from the PCB silkscreen (`J1`-`J5`).

| Offset From `base` | R/W   | Function       | Description |
| ------------------ | ----- | -------------- | ----------- |
| `+0`               | Read  | Latched Byte 0 | Read latched counter value, byte 0 (LSB) |
| `+1`               | Read  | Latched Byte 1 | Read latched counter value, byte 1 |
| `+2`               | Read  | Latched Byte 2 | Read latched counter value, byte 2 |
| `+3`               | Read  | Latched Byte 3 | Read latched counter value, byte 3 |
| `+4`               | Read  | Latched Byte 4 | Read latched counter value, byte 4 |
| `+5`               | Read  | Latched Byte 5 | Read latched counter value, byte 5 (MSB) |
| `+6`               | Write | Latch Counter  | Latch (snapshot) the current 48-bit counter into the output registers. Any value may be written to trigger this action. |
| `+7`               | Write | Reset Counter  | Reset the counter value to `0`. Any value may be written to trigger this action. |

Concrete ranges for a selected `x`:
- `J1` open (`y = 0`): registers are at `$FEx0`-`$FEx7`.
- `J1` bridged (`y = 8`): registers are at `$FEx8`-`$FExF`.

Notes:
- Bridging all jumpers `J2` through `J5` sets `x` to `$F`; leaving them open sets `x` to `$0` (default).
- Reading `$FEx0`-`$FEx5` (or `$FEx8`-`$FExD` when `J1` is bridged) returns the last value latched by writing to the latch register, not necessarily the current counter value.
- Resetting the counter does not latch a `0` value.

## Time Range
This card has a 48-bit counter that is read one byte at a time. Not all bytes need to be read, but reading fewer bytes reduces the effective time range captured. At the **Minimal 64x4**'s 8 MHz clock, the maximum time range by number of bytes read is shown in this table:

| Bytes | Bits | Max Seconds (s) | Approx Time                      |
| ----: | ---: | --------------: | -------------------------------- |
|     6 |   48 | 35184372.088832 | 407.22652996 days (~1.115 years) |
|     5 |   40 |   137438.953472 | 38.17748708 hours (~1.5907 days) |
|     4 |   32 |      536.870912 | 8.94784853 minutes               |
|     3 |   24 |        2.097152 | 2.097152 seconds                 |
|     2 |   16 |        0.008192 | 8.192 ms                         |
|     1 |    8 |        0.000032 | 32 µs                            |

The latched bytes are arranged in little-endian order: `base + 0` is the LSB and `base + 5` is the MSB.
