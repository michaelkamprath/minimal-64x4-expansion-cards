# Minimal 64x4 Breadboard Prototyping Adapter: Design Prompt

I need help designing a breadboard prototyping adapter for the Minimal 64x4 expansion bus. It is bench tooling: it lets me prototype expansion cards on a solderless breadboard before designing their PCBs. It is not a product and will not get its own video. Produce a complete design: schematics for two PCBs, a cable specification, a parts list, and a validation plan. Work through the open questions with me before producing final deliverables.

## The host machine

The Minimal 64x4 is a discrete 74HC TTL home computer by Carsten Herting (slu4). 8-bit data bus, 16-bit address bus, 8 MHz CPU clock, 125 ns cycle. The CPU reads the data bus on the cycle when its RAM output strobe is active and writes on the cycle when its RAM input strobe is active.

The expansion port is 32 pins, unbuffered, and the manual warns not to add capacitance to it:

| Pin | Direction | Signal |
|---|---|---|
| 1 | | GND |
| 2 | | +5 V |
| 3 | in | INH, active high; when high the system RAM and FLASH are disabled so a card can drive the bus |
| 4 | | user line (unused by the Minimal; my cards share it as a signal the CPU cannot read) |
| 5 | in/out | CPU reset, active low |
| 6 | out | buffered 8 MHz system clock |
| 7..14 | in/out | BUS0..BUS7 |
| 15..30 | out | A0..A15 |
| 31 | out | RAM output strobe, active low (CPU is reading) |
| 32 | out | RAM input strobe, active low (CPU is writing) |

Page 0xfe00..0xfeff is reserved for expansion cards. Each card decodes its own addresses and asserts INH only on cycles where it is actually driving the data bus for a CPU read. I have a stackable backplane that merges INH from several cards through a 74AHCT126 (each slot's INH drives the output enable of one buffer whose input is tied high) and passes the port signals to each slot, so the adapter normally plugs into a backplane slot. It must also work plugged directly into the Minimal's expansion port, where pin 4 is unconnected and the PLD's INH drives the Minimal's INH line (4.7 kΩ pulldown) directly.

## My conventions (github.com/michaelkamprath/minimal-64x4-expansion-cards)

- Address decode with 74HC688 comparators against the 0xfe page plus switch- or jumper-selected base bits.
- 74AHCT logic on timing-critical paths; 74HC elsewhere. ATF16V8C or ATF22V10C PLDs for control logic.
- INH timing on my existing cards is proven; the adapter must not add enough delay to change that.

## What the adapter is

Two PCBs and a ribbon cable:

**Board A, backplane adapter.** Plugs into a backplane slot. Buffers every signal between the backplane and the cable except reset and the user line, decodes the 0xfe page plus a switch-selected 16-byte base window, and sends a base-match signal down the cable. Protects the Minimal: anything the breadboard does wrong should damage a buffer on this board, never the computer.

**Board B, breadboard breakout.** Receives the cable, provides bulk decoupling and the power entry, and brings every signal to labeled 0.1 inch header pins that plug into a solderless breadboard.

**Cable.** Ribbon, as short as the bench allows (target 15 to 20 cm). Every signal conductor must have a ground conductor adjacent to it: the pattern is signal, ground, signal, ground along the whole cable. This is a hard requirement, not a preference. The ground next to each signal is its return path; with 74AHCT edges of a few nanoseconds, a signal whose return current has to find its way to a distant ground conductor forms a loop that rings, radiates, and couples into its neighbors. Adjacent grounds also halve crosstalk between signals. The cost is half the conductors, which is why the signal list below is kept to 19 and a 40-way cable is specified rather than a 26-way. The clock and the two strobes are the most sensitive and each must sit between two grounds. All cable grounds tie to the ground plane at both ends. No logic power rides on the ribbon. +5 V to the breadboard goes over a separate twisted pair (+5 V and ground, 20 or 22 AWG) from Board A's J2 to Board B, with its own connector on each board. The pair's ground is not there for current capacity: the ribbon's 21 grounds (about 1.7 mm² of 28 AWG copper together) carry more than a 20 AWG wire, and because both paths connect at both ends, most of the supply's DC return flows in the ribbon anyway, at well under a millivolt of drop for a prototype's current. The pair's ground gives the +5 V wire a return right beside it, so twisting the two keeps the loop that carries the breadboard's supply transients small instead of spreading it across the ribbon.

## Signals on the cable, breadboard side

Only what a prototype at a decoded base address needs:

- A0..A3 (register select within the 16-byte window)
- ~MATCH, active low (adapter decoded the 0xfe page and the selected base bits; active for the whole 16-byte window)
- BUS0..BUS7, bidirectional
- RAM output strobe, RAM input strobe
- 8 MHz clock
- reset, active low, one conductor, unbuffered pass-through (see Board A requirement 5)
- user line (bidirectional, unbuffered pass-through)
- INH request, active high, from the breadboard to the adapter

That is 19 signals. Odd pins are ground and the signals sit on even pins 2 through 38, so along the ribbon every signal has a ground on both sides; pin 40 is an extra ground. The pin assignment is in [`README.md`](README.md), which is the single source of truth for the cable. Do not drop the interleaved grounds to fit a smaller cable; if more signals are ever needed, the answer is a second cable.

## Board A requirements

1. **Address decode.** Two 74HC688s, working in parallel rather than chained: one compares A8..A15 to 0xfe (~PAGE), the other compares A4..A7 to a 4-position DIP switch (ON = 1, 10k pulldowns) (~NIB). The PLD (requirement 3) combines them into ~MATCH, which goes to the cable. There is no option to export the raw page match: A4..A7 are not on the cable, so a prototype could not use it.
2. **Outbound buffers** (backplane to cable): A0..A3, ~MATCH, both strobes, and the clock, through one 74AHCT245 with its direction wired A to B and its output enable wired low (eight channels, all used). Using the same part as the data transceiver keeps the parts list to one transceiver type; the 541 was no faster (TI datasheets: 9.5 ns max for both at 50 pF, -40 to 85 °C). All eight lines, the clock included, are terminated by the same 8-resistor pack (RN1). Reset and the user line are not buffered (see requirement 5).
3. **Data bus transceiver.** One 74AHCT245, A side toward the cable, B side toward the backplane. Direction is toward the backplane only when INH request is high, the RAM output strobe is low, the RAM input strobe is high, the adapter's own window match is active, and the adapter is enabled; toward the breadboard otherwise, so the breadboard sees every CPU write in its window. Qualifying with the strobes, as the timer card does, keeps a breadboard that raises INH request at the wrong time from fighting the CPU on the bus. Output enable whenever ~MATCH is active, with no jumper, and never while the adapter is disabled. INH is only asserted inside the window, so this also covers every cycle where INH is asserted. The direction, enable, INH, and match logic is in one ATF16V8C-7PU (`pld-files/backplane-adapter.pld`): 6 inputs and 4 outputs fit easily, at the same 7.5 ns as the ATF22V10C-7 in a smaller DIP-20. The PLD reads the strobes directly from the backplane, not through the outbound buffer, to keep the INH path short. The backplane clock also goes to the PLD's pin 1, unused today, so registered outputs can be added later without rework; the clock to the cable stays on the outbound buffer.
4. **INH.** The backplane already buffers INH to the computer, so the adapter does not need a bus driver. INH comes from a PLD output with the same equation as the 245's direction: INH request AND RAM output strobe low AND RAM input strobe high AND the local window match AND enabled. The local match makes INH fall as soon as the address leaves the window instead of after a round trip through the cable and the breadboard. That matters because INH drives the Minimal's SRAM ~CE directly and every read step is followed by an opcode fetch from RAM, so a late INH release eats that fetch's access time. That matches the timer card, whose INH is also qualified by the strobes. INH and DIR use separate PLD pins so the off-board INH trace does not share a driver with the local DIR pin. The PLD output also gives the backplane's merge a clean local edge instead of whatever the cable delivered. This is the one timing-critical path the adapter adds: INH request, cable, and one PLD delay. Its release is no slower than a card with a 74HC688 and a PLD.
5. **Reset.** One active-low line on the cable, wired straight between the backplane and Board B with no gate on Board A, so the breadboard circuit sees the Minimal's reset and resets its own state the way my other cards do. The push button on Board B pulls this line low directly. Breadboard logic must never drive this line high. No reset button on Board A; the Minimal's own is nearby. Reset is slow, so no series termination or edge restoration is needed on it.
6. **Enable switch.** A single switch (SW2, closed = enabled, open = disabled) that turns the 245 fully off and holds INH low, so a misbehaving breadboard can be cut off without unplugging anything. While disabled the breadboard also stops seeing CPU writes.
7. **Series termination.** 33 ohm in series with every line driven onto the cable from Board A: RN1 for the outbound buffer (U3, clock included) and RN2 for the data transceiver's cable side (U4). If the scope shows ringing or slow edges on the breadboard, the fix is to swap the pack for a different value.
7a. **Breadboard power jumper and fuse.** JP1, a 2-pin header with a shunt, in J2's +5 V lead, followed by F1, a 0.5 A hold resettable PTC fuse (Bourns MF-RHT050, about 0.92 A trip) so a breadboard fault cannot pull more than about 1 A through the Minimal's or the backplane's 5 V path. Shunt fitted: the breadboard is powered from the backplane. Shunt removed: J2's +5 V is disconnected and the breadboard needs its own supply. Ground stays common either way, through J2 and the ribbon.
8. **Scope header.** J3, a 1x6 right-angle pin header on the card's forward-facing side edge, close to the backplane connector and reachable with cards packed on the backplane (exposed test pads are not). Pins 1 to 5 carry the backplane side of INH, the RAM output strobe, the RAM input strobe, the clock, and BUS0 for an oscilloscope or logic analyzer; pin 6 is ground for the probe's ground clip.
9. **Decoupling.** 100 nF on every IC, 10 uF at the board's power entry.

## Board B requirements

1. Ribbon connector in, and the power connector for the twisted pair from Board A's J2 (screw terminal or 2-pin Molex), with 100 uF electrolytic plus 10 uF ceramic at the power entry. A bench supply must be connectable without unplugging the pair, for use when Board A's JP1 shunt is removed: a second 2-position screw terminal (J3) in parallel with the pair's terminal (J2). The +5 V and ground also go to pins that plug into the breadboard's power rails on both sides (J8, J9), so the rails are powered from Board A.
2. Every cable signal to a labeled 0.1 inch header on a pitch that plugs straight into a standard breadboard's rows, with silkscreen names on both sides. Group them: address and base match, data bus, control, clock, INH request, power.
3. Multiple ground and +5 V header pins distributed along the board so the breadboard's rails can be fed from more than one point.
4. One power LED. No signal LEDs; at 8 MHz they show nothing useful, and the scope test points on Board A cover diagnostics.
4a. A momentary push button that pulls the reset line low, so the Minimal can be reset from the breadboard without reaching over to the computer. Debounce is unnecessary; the Minimal's own reset circuit handles that.
5. No second 74AHCT245 on the data lines. It was considered for isolating the breadboard's bus, but its 5 to 10 ns delay would push the prototype's read data past the worst-case budget, and Board A's U4, termination and SW2 already isolate and protect the bus. A prototype that needs load isolation puts its own 74AHCT245 on the breadboard.
6. No decode, no logic beyond the optional transceiver. The prototype on the breadboard builds its own register decode from A0..A3, base match, and the strobes, exactly as the final card will.

## Deliverables

- Schematic for Board A and Board B, with part numbers and values.
- Cable pin assignment with the interleaved ground pattern, and the connector part numbers for both ends.
- A timing estimate for the INH path (breadboard INH request through the cable and Board A's PLD to the backplane, and its timing relative to the RAM output strobe) and for the data path on a CPU read (breadboard drives bus, through Board B's optional transceiver if fitted, the cable, Board A's 245, to the backplane), against the 125 ns cycle, using 74AHCT datasheet propagation delays and about 5 ns per meter of ribbon.
- A parts list.
- A validation plan: build the timer card's circuit (48-bit cycle counter, 74HC590 chain, 74HC688 decode, 22V10 control; see the repo) on the breadboard through the adapter, run it for an hour, and read the counter against a known interval; then a scope capture of INH versus the RAM output strobe with the cable in the path. Describe what a pass looks like.
- A list of design decisions you made that I should confirm.

Ask me questions where the requirements above are ambiguous rather than guessing. Do not use em dashes in anything you write.
