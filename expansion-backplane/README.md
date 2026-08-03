# Expansion Backplane for Minimal 64x4 Computer

This backplane allows connecting of up to 4 expansion cards to the Minimal 64x4 computer. The [expansion connector](./hardware/expansion-connector/) is a convenient way to connect the backplane to the Minimal 64x4 computer.

### Shared `INH` Line
To have multiple cards connected to the backplane, they need to share the active high `INH` inhibit line. If multiple cards are emitting a LOW signal directly, and one emits a HIGH, contention will occur. To prevent such contention, the back plane uses a 74AHCT126 to arbitrate the `INH` signal's progression from a card to the Minimal 64x4's exansion port. Each of the buffers in the 74AHCT126 have their input signal tied high, and the each card's `INH` output us used to control the output enable for it's assigned buffer. In this way, when a card's `INH` output is low (not active), it's corresponding buffer has its output in a high-Z state. So if another card asserts it's `INH` line high, there won't be bus contention back to the card. But each card's active high `INH` signal is passed through to the Minimal 64x4. The downside is that each expansion card cannot monitor whether another card is asserting the `INH` signal. This is not deemed to be a signficant issues since Minimal 64x4 expansion board are generally memory mapped, and thus only one can be actively writing to the bus at a time.


### Card Address Assignment
The data bus has **no arbitration** — the `INH` scheme above protects only the inhibit line. Correctness depends entirely on one card decoding a given address. Because address lines settle at different times during a bus transition, a card can transiently match another card's address and drive the bus against it, so assign base addresses to maximize the **Hamming distance of the decoded address bits**, not the hex separation.

| Card | Base | Decoded `A7..A3` |
| --- | --- | --- |
| Multiplier | `$FED0` | `11010` |
| Timer | `$FE28` | `00101` |

**Failure signature** (seen 2026-08 with the timer at `$FEF0`, one bit from the multiplier): reads from the victim card return wrong values intermittently, writes and RAM are fine, results vary run to run *and* between builds as code shifts, and software-only builds are perfect. Diagnose by moving one card, not by debugging the software.

Two mechanisms produce this and are **not yet distinguished**: address-line skew, or the 74HCT688 comparator glitching its output during input transitions. If it's the latter, bit distance is only probabilistic protection and a gated `/G` or delayed enable is needed. To measure: reinstate the colliding address, run `multiplier/software/test_karatsuba_layers.min64x4` (fails ~30% of 65536 cases, so events are continuous), and trigger a logic analyzer on *the victim card's decode asserting while the address belongs to the other card*.

Note that CLK-gating a card's read output enable is **not** a fix: the address and `RI`/`RO` are set on the falling edge and the selected card must be driving the bus before the rising edge latches it, so the read enable has to be live during the settling window by design.

### Power Barrel Jack
The back plane provides a convenient standard 2.1 mm barrel jack you may use to connect to a regulated `5V` power supply to power the expansion cards and additionally power the Minimal 64x4 computer. If this barrel jack is connected to power, be sure to not connect the UART's `+5V` line to the Minimal 64x4 (still connect `GND`)