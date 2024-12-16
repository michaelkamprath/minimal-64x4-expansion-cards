# Expansion Backplane for Minimal 64x4 Computer

This backplane allows connecting of up to 4 expansion cards to the Minimal 64x4 computer. The [expansion connector](./hardware/expansion-connector/) is a convenient way to connect the backplane to the Minimal 64x4 computer.

### Shared `INH` Line
To have multiple cards connected to the backplane, they need to share the active high `INH` inhibit line. If multiple cards are emitting a LOW signal directly, and one emits a HIGH, contention will occur. To prevent such contention, each card should use some sort of barrier between its `INH` signal source and the `INH` line on the back plane.

The technique generally used in these cards is to use a BS250 MOSFET to act as a high-side switch to the `INH` line. This requires the card to actually drive an active low `/INH` signal to the BS250's gate.

### Power Barrel Jack
The back plane provides a convenient standard 2.1 mm barrel jack you may use to connect to a regulated `5V` power supply to power the expansion cards and additionally power the Minimal 64x4 computer. If this barrel jack is connected to power, be sure to not connect the UART's `+5V` line to the Minimal 64x4 (still connect `GND`)