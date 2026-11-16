# Atari 2600 Style Joystick Adapter for Minimal 64x4 Computer
This project lets the [Minimal 64x4](https://github.com/slu4coder/Minimal-64x4-Home-Computer) read an Atari 2600 style joystick: four direction switches and one fire button on a DE-9 connector. It comes with joystick libraries for assembly and for [Extended Min](https://github.com/michaelkamprath/extended-min), and with joystick versions of Alien Invaders and Blocks.

**Status:** a prototype on a solderless breadboard, built behind the [breadboard adapter](../expansion-backplane/hardware/breadboard-connector/). There is no PCB yet. The schematic passes KiCad's electrical rules check and the assembly sources compile, but nothing here has run on hardware so far.

## Hardware Design
The adapter is one chip, a 74HC245 used as a bus buffer, and five pull-up resistors. The KiCad project is in [`hardware/breadboard`](./hardware/breadboard/); it has a schematic only.

* Each joystick switch connects its DE-9 pin to ground when closed. A 10 kΩ pull-up holds the line high while the switch is open.
* The five switch lines go to A0..A4 of the 74HC245. A5..A7 are tied to ground. The direction pin (pin 1) is tied to +5 V, so data only ever flows from the A side to the B side, which is wired to the data bus.
* The 74HC245's output enable (pin 19) is wired to the breadboard adapter's `~RO`, so it drives the cable's data lines during every CPU read. Board A passes them on to the CPU only for a CPU read inside its 16-byte window. Outside the window Board A's own transceiver is switched off, so the 74HC245 fights nothing there. On a CPU write `~RO` is high and the 74HC245 is off, which matters inside the window, where Board A drives the cable toward the breadboard. No other logic is needed.
* `INH_REQ` is tied to +5 V. Board A of the breadboard adapter raises `INH` only during a CPU read inside its window, whatever `INH_REQ` does at other times, so a prototype that answers every address in the window can ask permanently.
* `~MATCH`, A0..A3, `~RI`, CLK, `~RESET` and USER are not used. There is no register for the CPU to write.

### Register
With SW1 on Board A set to `7` (A7 off, A6 on, A5 on, A4 on), the adapter answers a CPU read of any address in `0xFE70`..`0xFE7F` with the same value. The software uses `0xFE70`.

| Bit | Switch | DE-9 pin |
|---|---|---|
| 0 | up | 1 |
| 1 | down | 2 |
| 2 | left | 3 |
| 3 | right | 4 |
| 4 | fire | 6 |
| 5..7 | always 0 | |

A bit reads **0 while its switch is closed** and 1 while it is open. With no joystick plugged in the register reads `0x1F`. DE-9 pin 8 is ground. Pin 7 (+5 V) and the paddle pins 5 and 9 are not connected.

A CPU write to the window lands in the Minimal's RAM under it, as with the other cards. The libraries use that to detect the adapter: they write `0xFF` to `0xFE70` and read it back. The adapter answers with bits 5..7 at 0; without it the RAM answers `0xFF`.

`0xFE70` was chosen so the address differs from the other cards' addresses (`0xFEC0` speech, `0xFED0` multiplier, `0xFE28` timer) in at least two of the decoded bits A7..A3.

### Parts for the breadboard

| Ref | Part |
|---|---|
| U1 | 74HC245 octal bus transceiver, DIP-20 |
| RN1 | 5 × 10 kΩ bussed resistor network, SIP-6 (or five 10 kΩ resistors) |
| C1 | 100 nF ceramic capacitor, across U1's supply pins |
| J1 | DE-9 connector with pins (male), with a breakout or wire tails for the breadboard |

Only the 74HC245's output-enable time matters, because the switch lines do not change during a CPU read. From `~RO` arriving at the breadboard, the breadboard adapter's budget for data on the cable is about 49 ns typical, 36 ns at the 25 °C maximum and 26 ns at the maximum over temperature (30 cm cable). A 74HC245 enables its outputs in about 15 ns typically and 30 ns at most at 25 °C, so it fits on the bench; its maximum over temperature, about 38 ns, does not fit the stacked worst case. These 74HC245 figures are from memory; check them against the datasheet of the part used. A 74AHCT245 wired the same way fits every case.

## Software Design
All software is in the [software](./software/) directory.

### Assembly library
`joysticklib.min64x4` provides:

* `joystick_init`
  Detects the adapter and remembers the result. Returns 1 in `A` if the adapter answered, 0 if not. Call it once at start.

* `joystick_read`
  Returns the closed switches in `A`, with a set bit meaning closed (the library inverts the hardware's value). Returns 0 if the adapter is not fitted, so a program does not have to check. The bit masks are `JOYSTICK_UP`, `JOYSTICK_DOWN`, `JOYSTICK_LEFT`, `JOYSTICK_RIGHT` and `JOYSTICK_FIRE`.

* `joystick_present`
  Returns in `A` what `joystick_init` found.

### Extended Min library
`joysticklib.xmin` provides:

* `joyInit()` detects the adapter and returns 1 or 0.
* `joyRead()` returns the closed switches as a sum of `kJoyUp`, `kJoyDown`, `kJoyLeft`, `kJoyRight` and `kJoyFire`, or 0 without the adapter.
* `joyKey()` turns the joystick into key presses for programs written around a keyboard: fire gives `" "`, up `"w"`, left `"a"`, right `"d"`, down `"s"`, and 0 when there is nothing new. A switch gives its key once when it closes; left, right and down repeat while held. Call it once per pass of the main loop. The repeat timing counts calls, so adjust `kJoyRptDly` and `kJoyRptRate` to the speed of the program's loop.

### Programs

* `joystick_test.min64x4` shows the raw register value and the closed switches until a key is pressed. Run it first to check the wiring.
* `joystick_invaders.min64x4` is the [speech synthesizer version of Alien Invaders](../speech-synthesizer/software/speech_invaders.min64x4) with joystick control added: left, right and fire. Fire also starts a game. The keyboard still works, and the game runs without the joystick adapter. Speech is a compile-time option: built without `ENABLE_SPEECH` the game is silent and does not need the speech synthesizer card; built with it, the card is required.
* `joystick_blocks.xmin` is slu4's Blocks with joystick control added through `joyKey()`: left and right move, down drops one row, up or fire rotates, and fire stands for SPACE on the title screens. The title screen and the game screen list the keyboard and joystick controls side by side; the joystick column appears only when the adapter is detected. Save `joysticklib.xmin` on the Minimal under that name so the `use` line finds it. Each pass of the game loop now also polls the joystick, so the game may run a little slower than the original; lower the starting value of `wait` if it does.

# Compiling the Software
The assembly sources are compiled with [BespokeASM](https://github.com/michaelkamprath/bespokeasm):

```sh
bespokeasm compile -c /path/to/bespokeasm/examples/slu4-minimal-64x4/slu4-minimal-64x4.yaml -n -p -t intel_hex joystick_test.min64x4
```

`joystick_invaders.min64x4` compiles the same way and gives the game without speech. To build the speech in, define `ENABLE_SPEECH` and add the speech library's directory to the include path:

```sh
bespokeasm compile -c /path/to/bespokeasm/examples/slu4-minimal-64x4/slu4-minimal-64x4.yaml -n -p -t intel_hex -D ENABLE_SPEECH -I ../../speech-synthesizer/software joystick_invaders.min64x4
```

Send the Intel HEX output to the Minimal 64x4 with its `receive` command. The `*.xmin` files are run by Extended Min on the Minimal itself.
