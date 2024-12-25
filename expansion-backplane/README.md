# Expansion Backplane for Minimal 64x4 Computer

This backplane allows connecting of up to 4 expansion cards to the Minimal 64x4 computer. The [expansion connector](./hardware/expansion-connector/) is a convenient way to connect the backplane to the Minimal 64x4 computer.

### Shared `INH` Line
To have multiple cards connected to the backplane, they need to share the active high `INH` inhibit line. If multiple cards are emitting a LOW signal directly, and one emits a HIGH, contention will occur. To prevent such contention, the back plane uses a 74HCT126 to arbitrate the `INH` signal's orrigression from a card to the Minimal 64x4's exansion port. Each of the buffers in teh 74HCT126 have their input signal tied high, and the each card's `INH` output us used to control the output enable for it's assigned buffer. In this way, when a card's `INH` output is low (not active), it's corresponding buffer has its output in a high-Z state. In this way if another card assert's it's `INH` line high, there won't be bus contention back to the card. But each card's active high ING signal is passed through to the Minimal 64x4. The downside is that each expansion card cannot monitor whether another card is asserting the `INH` signal. This is not deemed to be a signficant issues since Minimal 64x4 expansion board are generally memory mapped, and thus only one can be actively writing to the bus at a time.


### Power Barrel Jack
The back plane provides a convenient standard 2.1 mm barrel jack you may use to connect to a regulated `5V` power supply to power the expansion cards and additionally power the Minimal 64x4 computer. If this barrel jack is connected to power, be sure to not connect the UART's `+5V` line to the Minimal 64x4 (still connect `GND`)