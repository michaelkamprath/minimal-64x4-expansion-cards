# TMS5220 Speech Synthesizer for Minimal 64x4 Computer
This project creates a TMS5220-base speech synthesizer board for the [Minimal 64x4](https://github.com/slu4coder/Minimal-64x4-Home-Computer) expansion port. The high level deisgn criteria is:

1. Accomadate the "slow memory" nature of the TMS5220 by using hardward to buffer the interactions between the CPU and the TMS5220 such that the CPU is free to do other things when the TMS5220 is processing data. This is in contrast to the typical hardware design among 1980's computers which was to halt the CPU while the TMS5220 was processing data sent to it.
2. Only support "external" speech data.  PHROMs are not supported. Not supporting PHROMs greately simplifies the board, but also save money as original PHROMs are hard to come by in the mordern age. Most games of the 1980s did not use PHROM data, instead providing their own LPC-encoded speech data.


This board supports both the TMS5220 and TMS5220C variants of the speech synthesizer chip.


## Hardware Design
The board responds to three memory mapped addresses (the board is adjustable to set the `_` value below, but it defaults to `0xC`:

* `0xFE_0` - **TMS5220 Command Register** - This address allows the host computer to write data to the TMS5220's command register, which is used for sending "external speech" data to the TMS5220. In general, the host computer will write certain TMS5220 commands and speech data to this address.
* `0xFE_1` - **TMS5220 Status Register** - This address allows the host computer to fetch the internal TMS5220 status register. To do that, the host computer needs to write some data to this address (any value as the value is not actually used) to start the sequence of events to get the internal status out of the TMS5220. When done, the status value will be readable by the host computer at this address. The status data being ready is indicated by the board's `BUSY` line readable at the **Board Status Register**. The `BUSY` line will be asserted high from the moment this address is written to until the moment the status value is ready to be read at this address.
* `0xFE_2` - **Board Status Register** - Thiadddress exposes several internal states of the this board. This is done independent of the TMS5220, thus this register can be read by the host computer at any time. The individual bits within the byte value read are below. Of these lines, the host computer will generally only interact with the `/INT` and `BUSY` bits.
  * `bit 0` - `BUSY` - Indicates whether the TMS5220 or the board generally is working on something. The host computer should not interact with the TMS5220 Command or Status registers while this bit is high.
  * `bit 1` - `/INT` - The current value of the active low interupt line comming out of the TMS5220. When this bit is low, that indicates the TMS5220 requires some attention. What the issues is can be determined by reading the TMS5220 Status Register.
  * `bit 2` - `/READY` - The current value of the active low `READY` line out fo the TMS5220. Host computer software should use the `BUSY` line to arbitrate interaction with the board, but the `READY` line is exposed here for diagnostic purposes.
  * `bit 3` - `WS` - The current status of the `WS` flip flop (active high). Exposed for diagnostic purposes.
  * `bit 4` - `RS` - The current status of the `RS` flip flop (active high). Exposed for fiagnostic purposes.
* `OxFE_3` - **TMS5220C Reset** - _Currently Unimplemented_ - The TMS5220C variant can be reset by holding both the `/RS` and `/WS` lines low at the same time. This action has undefined results for the TMS5220 (no "C") variant.

### TMS5220 "Slow Memory" Issue
A key pin to understand on the TMS5220 is the READY signal at pin 18. The TMS5220 will bring the READY pin low when it is ready to accept input or provide output to the host computer. When the READY pin is high, the TMS5220 is busy processing data and not immediate act on if not outright ignore any interactions the CPU initiates. For this reason, on many 1980's-era computer, the READY pin was connected to the CPU's halt or wait state pin, causing the CPU to stop processing while the TMS5220 was processing data, thus not risking sending data or commands faster than the TMS5220 can process. This board takes a different approach. In someways, it is necessary to do so because the Minimal 64x4 has no line that would allow an accessory to pause or halt the CPU's processing, but the approach here is agnostic of that fact.

As a result, data interactions between the host computer and the TMS5220 need to be cached to handle the timing differences. The key to caching the CPU interactions with the TMS5220 is the ATF22V10C programmable logic chip and the 74HCT74 flip flops. The flip flops are used to hold the `/WS` and `/RS` signals on the TMS5220, and the ATF22V10C is programmed to create a state machine to manage the flip flops. For example, here is the sequence of events for writing a byte of data to the TMS5220:

1. When the address bus is set to the memory mapped address of 0xFFD0, the 74HCT688s will assert this MMAP line, and the M0 and M1 address lines will be 0. Furthermore, to write the TMS5220, the host computer `RI` line will be asserted. The ATF22V10C’s programmable logic will then assert this `WS_CLK`, which is the clock line for driving the `WS` flip flop, which in turn asserts the active low `/WS` line to the TMS5220. This tells the TMS5220 that we want to write data to it’s data lines.
2. At the same time the `WS_CLK` line causes the `WS` flip flop to set its state, the `WS_CLK` line is also used to cause the data bus input 74HCT374 flip flop to load the contents of the host computer’s data bus. Because of this, the expansion board can capture and hold the data bus value the host computer is trying to write to the TMS5220, thus allowing the host computer to move on to other things while this flip flop is holding the data bus value for the TMS5220.  Furthermore the `/WS` line to the TMS5220 also drives the output enable on the 74HCT374 octal flip flip used to buffer the data from the host computer data bus, so the buffered data will be held and written to this internal data bus on the board the whole time the `/WS` line is held low, even as the host computer moves on to other things. This is where the “slow memory” issue of the TMS5220 plays out. The TMS5220 will take several host computer clock cycles to read in the data, and it’s data lines need to be held to the desired values the whole time. By using the 74HCT374 flip flop to buffer the data, holding the data lines active for an extended period of time is possible.
3. In response to the `/WS` line being asserted, the TMS5220 will drive the `READY` line high. When this occurs, the ATF22V10C is programmed to wait until the `READY` line subsequently transitions from high to low, which indicates that is done with processing the input data. When the `READY` line returns to low, the ATF22V10C then asserts the `WS_RST`, the reset line for the `WS` flip flop. This brings the `/WS` line high, which tells the TMS5220 that we are no longer writing data to the internal data bus. This also turns off the output enable on the 74HCT373 octal flip flop that was buffering the data to be written to the TMS5220. In this way, the `WS` line is asserted until the TMS5220 indicates it is done with reading the data from the buffers, as indicated by the `READY` line.
4. The whole time this sequence is occurring, the ATF22V10C asserts the `BUSY` line for the board, which can be read by the host computer at address 0xFE_2 via the 74HCT245. This then allows the host computer to query the board to determine if the TMS5220 is done with processing data.

The board uses a similar sequence when reading data from the TMS5220, though that sequence is centered around the `/RS` line.

## Software Design
The provided `speechlib.min64x4` file contains all the subroutines needed to use this board on the Minimal 64x4 home computer.

* `speech_init` - call to initialize the board
* `speak_synchronous` - Speak the LPC data in a passed buffer synchronously. Push the folling arguments on the stack (in push order):
  * **Buffer End Address** - The 2-byte end address of the data buffer
  * **Buffer Start Address** - The 2-byte start address of the data buffer
* `speech_start` - Speak the LPC data in a passed buffer asynchronously, synchronously waiting for any currently active talking to finish. Push the folling arguments on the stack (in push order):
  * **Buffer End Address** - The 2-byte end address of the data buffer
  * **Buffer Start Address** - The 2-byte start address of the data buffer
* `speech_start_if_not_talking` - Speak the LPC data in a passed buffer asynchronously if there is no currently active talking. Push the folling arguments on the stack (in push order):
  * **Buffer End Address** - The 2-byte end address of the data buffer
  * **Buffer Start Address** - The 2-byte start address of the data buffer
* `speech_update_buffer` - Call periodically, about every 1 ms, to enable asynchronous speaking. No arguments.
* `speech_get_status` - Get the contents of the TMS5220 status register, which is returned in the `A` register. The following bit masks are defined to help interpret the results:
  * `SPEECH_STATUS_TALK` - The TMS5220 is currently talking
  * `SPEECH_STATUS_BUFFER_LOW` - The TMS5220 FIFO buffer is half empty. Time to send more LPC data. _Note:_ The `speech_update_buffer` functions takes care of updating the FIFO buffer if needed.
  * `SPEECH_STATUS_BUFFER_EMPTY` - The TMS5220 FIFO buffer is empty.

### Undocumentated Timining Issues
The TMS5220 seems to have some timing requirements not documented in its datasheet. These requirements were exposed by the Minimal 64x4, which has a clock speed that is much faster than the host computers that the TMS5220 was designed for. These undocumented timing requirements were determined emperically and likely could be tweaked further.

* **Read Cycle for Status Transfers to External Speech Data write** - The data sheet indicats that the `t_wait` after `/RS` goes high is 12 µS. However, I found that something closer to 175 µS was needed between a status read and then an external speech data write.
* **External Speech Data write to Read Cycle for Status Transfers** -  Similar to the above, some extra time is needed between writing eexternal speech data and then reading the TMS5220 status. Again, I found this to be on the order of 175 µS.

These values were "tweaked until it worked". The issues seem to be related to changing modes of interaction with the TMS5220. In contrast, writing an external speech data byte after a previous external speech data byte does adhere to the timing diagrams in the TMS5220 data sheet. Further experimentation would likely find better and/or smaller values for the undocumented delays.

# Compiling the Software
All software found in the [software](./software/) directory is compiled with [BespokeASM](https://github.com/michaelkamprath/bespokeasm) which supports the Minimal 64x4. [Carsten Herting](https://github.com/slu4coder), the creator of the Minimal 64x4, does provide an assembler for it. However, the provided assembler is white minimal. A key shortcoming is the inaqbility to import library code to the current compilation. BespokeASM enables importing libraries and a few other features that are in use by the software found in this project.

To compile the speech code found in this repository, use the following command:

```sh
bespokeasm compile -c /path/to/bespokeasm/examples/slu4-minimal-64x4/slu4-minimal-64x4.yaml -n -p -t intel_hex lpc_player.min64x4
```
Where `/path/to/bespokeasm` is the path to the BespokeASM repository on the computer. This will produce the Intel hex rerpesentation of the compiled program that can then be sent to the Minim 64x4 using it's `receive` command.
