# Minimal 64x4 Breadboard Prototyping Adapter

Bench tooling for prototyping Minimal 64x4 expansion cards on a solderless breadboard before designing their PCBs. The adapter plugs into a slot of the [expansion backplane](../expansion-backplane/), or directly into the Minimal's expansion port, and brings a buffered, address-decoded view of the expansion bus to a breadboard over a short ribbon cable.

The design requirements are in [`breadboard-adapter-design-prompt.md`](breadboard-adapter-design-prompt.md).

## Boards

| Directory | Board | Role |
|---|---|---|
| [`backplane-adapter/`](backplane-adapter/) | Board A | Plugs into a backplane slot. Buffers all signals, decodes the `$FE` page plus a switch-selected 16-byte base window, drives INH, and protects the Minimal from the breadboard. |
| [`breadboard-breakout/`](breadboard-breakout/) | Board B | Receives the cable and the twisted power pair from Board A's J2, and brings every signal to labeled 0.1 inch headers that plug into a breadboard. |

The two boards are separate KiCad projects. Both use the shared symbol and footprint libraries in [`kicad-libraries/`](../../../kicad-libraries/).

## Cable pinout

**This table is the single source of truth for the ribbon cable.** The two boards are in separate KiCad projects, so nothing checks automatically that both ends agree. Any change to the cable connector on either schematic must be made here first, then applied to both boards.

Cable: 40-way, 0.05 inch pitch IDC ribbon, 15 to 20 cm. Every signal conductor has a ground conductor on each side. The clock and both strobes must sit between two grounds.

Status: **proposed**, as drawn on Board A J1. Every odd pin is ground, so along the ribbon every signal (even pin) sits between two grounds. Pin 40 is an extra ground. No logic power rides on the ribbon. `~MATCH` is active low; `INH_REQ` is active high.

| Pin | Signal | Pin | Signal |
|---|---|---|---|
| 1 | GND | 2 | A0 |
| 3 | GND | 4 | A1 |
| 5 | GND | 6 | A2 |
| 7 | GND | 8 | A3 |
| 9 | GND | 10 | ~MATCH (16-byte base window) |
| 11 | GND | 12 | ~RO |
| 13 | GND | 14 | ~RI |
| 15 | GND | 16 | CLK |
| 17 | GND | 18 | INH_REQ (breadboard to adapter) |
| 19 | GND | 20 | BUS0 |
| 21 | GND | 22 | BUS1 |
| 23 | GND | 24 | BUS2 |
| 25 | GND | 26 | BUS3 |
| 27 | GND | 28 | BUS4 |
| 29 | GND | 30 | BUS5 |
| 31 | GND | 32 | BUS6 |
| 33 | GND | 34 | BUS7 |
| 35 | GND | 36 | ~RESET (unbuffered pass-through) |
| 37 | GND | 38 | USER (unbuffered pass-through) |
| 39 | GND | 40 | GND |

Main +5 V and ground to the breadboard use a separate 20 or 22 AWG twisted pair from Board A's J2 to Board B, not the ribbon.

## Using the adapter

### Power

- **JP1 fitted (normal use):** the breadboard is powered from the backplane, or from the Minimal when Board A plugs directly into its expansion port, through F1 and J2. One supply powers both sides, so they always switch on and off together. F1, a 0.5 A resettable fuse, limits what a breadboard fault can draw through the Minimal's or the backplane's 5 V path; it trips at about 1 A and resets once the fault is removed and it cools. A prototype that needs more than a few hundred mA should use its own supply on Board B's J3, with JP1 removed. When Board A plugs directly into the Minimal, the computer may be running from its UART's USB power, which leaves only a few hundred mA for the breadboard.
- **JP1 removed:** J2's +5 V is disconnected and the breadboard needs its own supply. Ground stays common through J2 and the ribbon. **Switch both supplies on and off together.** If one side is powered and the other is not, the powered side's drivers push current through the 33 Ω termination into the unpowered chips' input protection diodes, on the order of 100 mA per line. That is an operating error, not a design fault, but it can damage parts on either side.

### Enable switch

SW2 is **closed for enabled**. With it open, the adapter is disconnected from the bus: the data transceiver (U4) is off and INH is held low. A freshly built adapter therefore does nothing until SW2 is switched on.

## Rules for a breadboard prototype

The adapter keeps a misbehaving breadboard from damaging the Minimal, but a prototype that follows these rules behaves like a real card and avoids fighting the adapter's own drivers:

1. **Decode from `~MATCH` and A0..A3.** `~MATCH` is low for every access to the 16-byte window set by SW1. A4..A15 are not on the cable.
2. **Raise INH_REQ only for a CPU read of one of your registers**, qualified by `~RO` low and `~RI` high, the same way the [timer card's PLD](../../../timer/pld-files/timer.pld) qualifies its INH. The adapter ignores INH_REQ outside reads anyway, but qualifying it keeps the prototype honest for its final PCB.
3. **Drive the breadboard's data bus only while `~RO` is low** (and you are raising INH_REQ). Outside those cycles the adapter's U4 drives the cable side of the data bus toward the breadboard, so a prototype that drives early fights U4 through the 33 Ω termination: current-limited, no damage, but messy waveforms.
4. **Keep the decode fast.** The prototype has about 14 ns, worst case, from its inputs to INH_REQ (see the timing estimate below). One pass through a -7 PLD (ATF16V8C or ATF22V10C) or two 74AHCT gates fits. A 74HC decode chain does not.
5. **Capture CPU writes on the rising edge of CLK while `~RI` is low**, not on `~RI`'s rising edge. See the write timing below.
6. **Never drive `~RESET` high.** It is wired straight through to the Minimal's reset line. Only pull it low (Board B's button does this).

## Building a prototype's logic

### Timing budget

A prototype's logic has two budgets, both counted from the moment its inputs arrive on the breadboard. The inputs that matter are the later of `~MATCH` and `~RO`, after U3 and the ribbon. They arrive about 57 ns after the CPU's clock edge typically, 67 ns at the 25 °C maximum, and 76 ns at the −40..85 °C maximum. Read data must be on the backplane by about 115 ns (see the timing estimate below).

| From inputs at the breadboard to... | Typ | Max 25 °C | Max −40..85 °C |
|---|---|---|---|
| **INH_REQ out** (X in the timing estimate) | 39 ns | 24 ns | **14 ns** |
| **Read data driven onto the cable** | 50 ns | 37 ns | **27 ns** |

The second budget covers the ribbon and U4's A-to-B delay on the way back to the backplane.

- **ATF16V8C-7 (7.5 ns):** fits both budgets, with 7 ns to spare at the worst case. A -10 part also fits. A -15 part misses the worst case by about 1 ns but is fine at room temperature.
- **Data drivers:** after the PLD's 7.5 ns, about 20 ns remain for the output enable of whatever drives the data bus. A 74AHCT245 or 74AHCT541 (16 ns max) fits. For anything slower, such as a counter's own output enable, check its datasheet. If it is over about 20 ns, put a 74AHCT245 or 541 in front of it.
- **INH release** does not depend on the prototype. Board A's PLD drops INH as soon as its own decode sees the address leave the window.

### A nominal ATF16V8C

[`pld-files/breadboard-prototype-template.pld`](pld-files/breadboard-prototype-template.pld) is a starting point that compiles with galette. Its example register map uses the window `$FEx0..$FEx3`, where `x` is the base nibble set by SW1:

| Address | CPU write | CPU read |
|---|---|---|
| `$FEx0` | WR0 | RD0 (INH_REQ) |
| `$FEx1` | WR1 | RD1 (INH_REQ) |
| `$FEx2` | WR2 | RD2 (INH_REQ) |
| `$FEx3` | WR3 | nothing; the CPU reads RAM |
| `$FEx4..$FExF` | nothing | nothing |

How it is put together:

- **INH_REQ repeats the read decode instead of ORing the RD outputs.** The timer card's PLD does `INH = /OE0 + /OE1 + ...`, which feeds outputs back into the array: a second PLD pass. On the breadboard that would double X to 15 ns, over the 14 ns worst-case budget.
- **Write strobes are registered on CLK with `~RI` low**, like `WS_CLK.R` on the speech card. Each strobe rises one clock-to-output delay after the cable's CLK edge and lasts exactly one cycle.
- **Clock each data latch (74AHCT374 or 574) on its strobe's rising edge.** The CPU's data stays on the bus for roughly 25 ns after that edge. The falling edge comes a cycle later, after the data is gone, which is why the strobes are active high.
- **Registered mode costs two pins.** Pin 1 becomes the register clock and pin 11 the registered outputs' `/OE`, which must be tied to GND. That leaves 8 outputs: in the template, INH_REQ, four write strobes and three read enables. A prototype with no write registers gets pins 1 and 11 back as inputs.
- **`~RESET` is on pin 9** for clearing prototype state. No equation in the template uses it yet.
- **A0..A3 hold time at the write sample is tight**, by a few ns, on every card. The Minimal can change the address 10 to 15 ns after the clock edge, about when the port's CLK, one 74HC32 behind the CPU's clock, arrives. The adapter makes this no worse, because CLK and A0..A3 pass through U3 together.

## Timing estimate (CPU read through the adapter)

### The Minimal's read cycle

From the "Timing Signals" section of the [Minimal 64x4 manual](https://docs.google.com/document/d/1-nDv_8WEG1FrlO3kEK0icoYo-Z-jlhpCMiCstxGOCjQ/edit?usp=sharing). Its diagram was measured on the author's board at 8.3 MHz and 5 V, with every time referenced to the CPU clock's rising edge at the start of the cycle. The [Revision 1.1 schematic](https://github.com/slu4coder/Minimal-64x4-Home-Computer/blob/main/Revision%201.1/Schematics.pdf) shows how the port's signals connect:

- **`~RO` and `~RI`** (port pins 31 and 32) come straight from the control-word FLASH (U21), with no gating. They are the same `~RO` that drives the RAM's and FLASH's output enables. Port `~RI` is the control-word level, not the RAM write pulse: the SRAM's `~WE` is `~RI_PLS`, which is `~RI` ORed with a short clock pulse (U7D) near the edge that ends the cycle.
- **CLK** (port pin 6) is `CLK_PC` through one 74HC32 gate (U33C), so it lags the CPU's clock edge by one gate delay.
- **INH** (port pin 3) has a 4.7 kΩ pulldown and drives the SRAM's `~CE` directly. In the `$FE` page the FLASH is already deselected by A15.

| Event | Time after the rising edge |
|---|---|
| Address lines (MAR outputs) change | 0 to 15 ns |
| Control word settles, including the port's `~RO` and `~RI` | 45 to 50 ns, and stays until 45 to 50 ns into the next cycle |
| The Minimal's own RAM puts read data on the bus | about 80 ns |
| Bus sampled | at the next rising edge, with 10 ns setup required |

At 8 MHz (125 ns cycle), **read data must be on the backplane bus by about 115 ns.** The address can change in the same cycle as the read: in `LDB` the MAR is loaded at the edge that starts the read step, so the address path starts at 15 ns, not at 0. These are measured, typical values. The manual notes that 74HC switching slows by about 20 % at 4.5 V.

### The adapter's two paths

A read has to wait for both the address and `~RO`, so there are two paths and the later one sets the timing. Datasheet propagation delays. Columns are typical, maximum at 25 °C, and maximum over −40 to 85 °C. The ATF16V8C datasheet gives only a maximum (7.5 ns for the -7 grade), so that figure is used in every column. The 74AHCT245 has no separate direction-change time; its output-enable time is used for U4's turnaround. **X** is the prototype's own decode delay from its inputs to INH_REQ.

**Address path:**

| Step | Part | Typ | Max 25 °C | Max −40..85 °C |
|---|---|---|---|---|
| Address valid | Minimal (measured) | 15 | 15 | 15 |
| Address to `~PAGE` / `~NIB` | 74HC688 (TI CD74HC688, 4.5 V, 50 pF) | 14 | 34 | 42 |
| `~MATCH` | ATF16V8C-7 | 7.5 | 7.5 | 7.5 |
| `~MATCH` onto cable | 74AHCT245 U3, A to B, 50 pF (TI) | 5.3 | 8.7 | 9.5 |
| 33 Ω + 20 cm ribbon to breadboard | estimate | 2 | 2 | 2 |
| Breadboard decode to INH_REQ | prototype logic | X | X | X |
| INH_REQ back over the ribbon | estimate | 1 | 1 | 1 |
| INH and DIR | ATF16V8C-7 | 7.5 | 7.5 | 7.5 |
| **INH** | | **52 + X** | **76 + X** | **85 + X** |
| U4 turnaround to driving the backplane | 74AHCT245 output enable, 50 pF (TI) | 9.7 | 14.8 | 16 |
| **Read data on the backplane** | | **62 + X** | **91 + X** | **101 + X** |

**`~RO` path** (the prototype qualifies INH_REQ with `~RO`, as rule 2 asks):

| Step | Part | Typ | Max 25 °C | Max −40..85 °C |
|---|---|---|---|---|
| `~RO` valid | Minimal (measured) | 50 | 50 | 50 |
| `~RO` onto cable, 33 Ω, ribbon | U3 + estimate | 7.3 | 10.7 | 11.5 |
| Breadboard decode, ribbon back, INH and DIR | X + 1 + 7.5 | 8.5 + X | 8.5 + X | 8.5 + X |
| **INH** | | **66 + X** | **69 + X** | **70 + X** |
| U4 turnaround | 74AHCT245 output enable | 9.7 | 14.8 | 16 |
| **Read data on the backplane** | | **76 + X** | **84 + X** | **86 + X** |

All values in ns. The PLD's own `~RO` term (50 + 7.5 + 16 = 74 ns worst case) never limits. The breadboard drives its data onto the cable while it raises INH_REQ. The data's trip through the 33 Ω, the ribbon and U4's A-to-B delay (about 12 ns worst case) runs in parallel with the INH_REQ, PLD and turnaround steps, so it does not add to the total.

### Result

The `~RO` path sets the typical case and the address path sets the worst cases. With a PLD decode on the breadboard (X = 7.5 ns):

| | Typ | Max 25 °C | Max −40..85 °C |
|---|---|---|---|
| Read data on the backplane | 83 | 98 | 108 |
| Margin to the 115 ns deadline | 32 | 17 | 7 |

The adapter fits, but the worst-case margin is thin. **The breadboard's decode budget, from its inputs to INH_REQ, is about 14 ns for the worst case over temperature, or 24 ns at room temperature.** That is one PLD pass or two 74AHCT gates, which is why rule 4 matters.

For comparison, the timer card raises INH at about 58 / 58 / 65 ns: its 74HC688 and PLD, assuming a 7.5 ns part, and its `~RO` qualification. The adapter's INH arrives 15 to 28 ns later than that. Until INH rises, the Minimal's RAM drives the bus, and U4 keeps pointing toward the cable. The hand-over to U4 is the same as on any card, just later.

### INH release and the following opcode fetch

INH drives the Minimal's SRAM `~CE` directly, and every read step ends in `IC`, so the next cycle is an opcode fetch from RAM. `~RO` stays low from one step into the next, so INH falls only when the decode sees the address leave the window. The SRAM's access then starts from that point: the AS6C1008-55 needs up to 55 ns from `~CE` to valid data, against the same 115 ns deadline.

The adapter's PLD includes its own window match (`/PAGE * /NIB`) in INH and DIR. INH therefore falls as soon as U1 or U2 sees the new address, without waiting for the breadboard to drop INH_REQ over the cable:

| | Typ | Max 25 °C | Max −40..85 °C |
|---|---|---|---|
| Address change, 74HC688, PLD: INH falls | 37 | 57 | 65 |
| Plus SRAM chip-enable access (55 ns max): fetch data on the bus | 92 | 112 | 120 |

That matches the timer card, which has the same 74HC688 and PLD. Without the local match, INH would have followed INH_REQ through the full round trip, falling at 60 / 83 / 92 ns with X = 7.5 ns and putting even the typical fetch at the deadline. The worst-case column is datasheet maximums stacked on the RAM's maximum access time, and it applies to any card that decodes the `$FE` page with a 74HC688.

**What is still unconfirmed:** the manual's numbers are typical values measured on one board, not worst cases. The scope capture of INH against `~RO` in the validation plan checks them on this machine.

## Write timing (CPU write through the adapter)

On a write, the Minimal's SRAM takes the data on `~RI_PLS`, a short pulse near the clock edge that ends the cycle. The port does not carry that pulse; it carries the control-word `~RI`. That level stays low until 45 to 50 ns into the *next* cycle, and by then the address may already have changed (up to 15 ns after the edge). A prototype that latches on `~RI`'s rising edge can therefore latch the right data at the wrong address.

Capture writes on the rising edge of CLK while `~RI` is low and the decode matches, as the timer card's registered PLD outputs (`LATCH.R`, `CLEAR.R`) do. Through the adapter, CLK, A0..A3, `~MATCH`, `~RO` and `~RI` all pass through U3 together, and the data passes through U4 B to A with a similar delay. Their timing relative to each other at the breadboard therefore matches what a card on the port sees, shifted later by about 7 to 12 ns. `~MATCH` changes later than A0..A3 because of the 74HC688 and the PLD, which only adds hold time.

## Board A parts list

What to look for when ordering. Part numbers are given only where one has been chosen.

| Ref | Qty | Part |
|---|---|---|
| U1, U2 | 2 | 74HC688, 8-bit identity comparator, DIP-20 |
| U3, U4 | 2 | 74AHCT245, octal bus transceiver, DIP-20 |
| U5 | 1 | ATF16V8C-7PU, SPLD, DIP-20 (program from `pld-files/backplane-adapter.pld` with galette) |
| (sockets) | 5 | DIP-20 IC socket, 0.3 inch |
| (sockets) | 2 | DIP-16 IC socket, 0.3 inch, for RN1 and RN2 |
| (socket) | 1 | 5-pin single-row socket strip, 2.54 mm, for RN3 |
| RN1, RN2 | 2 | 8 × 33 Ω isolated resistor network, DIP-16 (Bourns 4116R-1-330LF) |
| RN4, RN5 | 2 | Spare 8 × 68 Ω isolated resistor network, DIP-16 (Bourns 4116R-1-680LF), not fitted |
| RN3 | 1 | 4 × 10 kΩ bussed resistor network, SIP-5, common on pin 1 |
| R1, R2 | 2 | 10 kΩ, 1/4 W, axial |
| C1 to C5 | 5 | 100 nF ceramic, 2.5 mm lead spacing |
| C6 | 1 | 10 µF electrolytic, 16 V or more, 5 mm diameter, 2 mm lead spacing |
| SW1 | 1 | 4-position DIP switch, slide, 0.3 inch |
| SW2 | 1 | Enable switch: right-angle SPDT slide switch, C&K OS102011MA1Q or equivalent (common on pin 2, pin 1 = enabled, pin 3 unused) |
| CN1 | 1 | 2 × 16 right-angle male pin header, 2.54 mm (same as the other cards) |
| J1 | 1 | 2 × 20 right-angle shrouded box header, 2.54 mm (Sullins SBH11-PBPC-D20-RA-BK) |
| J2 | 1 | 2-position screw terminal, 5.08 mm, for the 20 to 22 AWG power pair |
| J3 | 1 | 1 × 6 right-angle male pin header, 2.54 mm |
| JP1 | 1 | 1 × 2 male pin header, 2.54 mm, plus one jumper shunt |
| F1 | 1 | Resettable PTC fuse, 0.5 A hold, about 0.92 A trip, radial (Bourns MF-RHT050) |
| (cable) | 1 | 40-way 0.05 inch flat ribbon, 15 to 20 cm, with two 2 × 20 IDC sockets (keyed) |
| (wire) | 1 | 20 or 22 AWG twisted pair, red/black, for J2 to Board B |

## Board B parts list

| Ref | Qty | Part |
|---|---|---|
| J1 | 1 | 2 × 20 vertical shrouded box header, 2.54 mm (Sullins SBH11-PBPC-D20-ST-BK) |
| J2, J3 | 2 | 2-position screw terminal, 5.08 mm: J2 for the twisted pair from Board A's J2, J3 for a bench supply (only with Board A's JP1 removed) |
| J4 to J7 | 4 | Male pin headers, 2.54 mm, that plug into the breadboard: 1 × 5 (ADDR: A0..A3, ~MATCH), 1 × 4 (CTRL: ~RO, ~RI, CLK, INH_REQ), 1 × 8 (DATA: BUS0..7), 1 × 2 (~RESET, USER) |
| J8, J9 | 2 | 1 × 2 male pin header, 2.54 mm (PWR_L, PWR_R): +5 V and GND at the top of each pin row, next to the cable connector. They plug into the breadboard's terminal strips, not its power rails. Run a jumper from each to the rails; the GND jumper is the breadboard's ground reference, so fit at least one before using any signal. |
| J10 | 1 | 1 × 1 male pin header, 2.54 mm: a third GND pin at the far end of the right pin row. Jumper it to a ground rail as well to shorten the ground return for the signals at that end. |
| H1, H2 | 2 | M3 holes in the two corners farthest from the breadboard for optional standoffs (about 11 mm, check against your breadboard). They sit beyond the cable connector, so pushing the ribbon plug in does not lever the pins out of the breadboard |
| C1 | 1 | 100 µF electrolytic, 10 V or more, 6.3 mm diameter, 2.5 mm lead spacing |
| C2 | 1 | 10 µF ceramic (X5R or X7R), 10 V or more, 5 mm lead spacing |
| D1 | 1 | 3 mm LED, power indicator |
| R1 | 1 | 1 kΩ, 1/4 W, axial |
| SW1 | 1 | 6 mm tactile push button, through-hole (reset) |

