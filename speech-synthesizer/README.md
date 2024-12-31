# TMS5220 Speech Synthesizer for Minimal 64x4 Computer
This project creates a TMS5220-based speech synthesizer board for the [Minimal 64x4](https://github.com/slu4coder/Minimal-64x4-Home-Computer) expansion port. The high-level design criteria are:

1. **Accommodate the “slow memory” nature of the TMS5220** by using hardware to buffer the interactions between the CPU and the TMS5220 such that the CPU is free to do other tasks while the TMS5220 processes data. This contrasts with typical 1980s computer hardware designs, which would halt the CPU while the TMS5220 processed data.
2. **Only support “external” speech data; PHROMs are not supported.** Not supporting PHROMs greatly simplifies the board and saves money, as original PHROMs are hard to come by in the modern age. Most games of the 1980s did not use PHROM data, instead providing their own LPC-encoded speech data.

This board supports both the TMS5220 and TMS5220C variants of the speech synthesizer chip.

## Hardware Design
The board responds to three memory-mapped addresses (the board is adjustable to set the `_` value below, but it defaults to `0xC`):

* `0xFE_0` – **TMS5220 Command Register**
  This address allows the host computer to write data to the TMS5220’s command register, which is used for sending “external speech” data to the TMS5220. In general, the host computer will write certain TMS5220 commands and speech data to this address.

* `0xFE_1` – **TMS5220 Status Register**
  This address allows the host computer to fetch the internal TMS5220 status register. To do that, the host computer writes some data to this address (any value, as the value is not actually used) to start the sequence of events to get the internal status from the TMS5220. When this is done, the status value will be readable by the host computer at this address. The status data being ready is indicated by the board’s `BUSY` line, which is readable at the **Board Status Register**. The `BUSY` line will be asserted high from the moment this address is written to until the status value is ready to be read.

* `0xFE_2` – **Board Status Register**
  This address exposes several internal states of this board, independent of the TMS5220, and can be read by the host computer at any time. The individual bits within the byte value read are below. Of these lines, the host computer generally only interacts with the `/INT` and `BUSY` bits.

  * **bit 0 – `BUSY`**
    Indicates whether the TMS5220 or the board, in general, is working on something. The host computer should not interact with the TMS5220 Command or Status registers while this bit is high.
  * **bit 1 – `/INT`**
    The current value of the active-low interrupt line coming out of the TMS5220. When this bit is low, it indicates that the TMS5220 requires attention. The specific issue can be determined by reading the TMS5220 Status Register.
  * **bit 2 – `/READY`**
    The current value of the active-low `READY` line out of the TMS5220. Host computer software should use the `BUSY` line to arbitrate interaction with the board, but the `READY` line is exposed here for diagnostic purposes.
  * **bit 3 – `WS`**
    The current status of the `WS` flip flop (active high). Exposed for diagnostic purposes.
  * **bit 4 – `RS`**
    The current status of the `RS` flip flop (active high). Exposed for diagnostic purposes.

* `0xFE_3` – **TMS5220C Reset** – *Currently Unimplemented*
  The TMS5220C variant can be reset by holding both the `/RS` and `/WS` lines low at the same time. This action has undefined results for the non-C TMS5220 variant.


### TMS5220 “Slow Memory” Issue
A key pin to understand on the TMS5220 is the `READY` signal at pin 18. The TMS5220 drives the `READY` pin low when it is ready to accept input or provide output to the host computer. When the `READY` pin is high, the TMS5220 is busy processing data and does not immediately act on—if not outright ignore—any interactions the CPU initiates. In many 1980s-era computers, the `READY` pin was connected to the CPU’s halt or wait-state pin, causing the CPU to stop while the TMS5220 processed data. This board takes a different approach. In some ways, this is necessary because the Minimal 64x4 has no line that would allow an accessory to pause or halt the CPU. However, this design choice is otherwise agnostic of that limitation.

As a result, data interactions between the host computer and the TMS5220 must be cached to handle the timing differences. The key to caching these CPU interactions is the ATF22V10C programmable logic chip and the 74HCT74 flip flops. The flip flops hold the `/WS` and `/RS` signals on the TMS5220, and the ATF22V10C is programmed to create a state machine that manages these flip flops. For example, here is the sequence of events for writing a byte of data to the TMS5220:

1. When the address bus is set to the memory-mapped address of `0xFEC0`, the 74HCT688s will assert this `MMAP` line, and the `M0` and `M1` address lines will be 0. To write to the TMS5220, the host computer `/RI` line will be asserted. The ATF22V10C’s programmable logic then asserts `WS_CLK`, which is the clock line that drives the `WS` flip flop, in turn asserting the active-low `/WS` line to the TMS5220. This tells the TMS5220 that we want to write data to its data lines.
2. At the same time `WS_CLK` causes the `WS` flip flop to set its state, `WS_CLK` also causes the data bus input 74HCT374 flip flop to load the contents of the host computer’s data bus. Because of this, the expansion board can capture and hold the data bus value the host computer is writing to the TMS5220, allowing the host computer to move on to other tasks while this flip flop holds the data bus value. Furthermore, the `/WS` line to the TMS5220 also drives the output enable on the 74HCT374 octal flip flop, so the buffered data will be held and driven onto the board’s internal data bus for the entire time `/WS` is low—even as the host computer does other work. This is the essence of the “slow memory” issue: the TMS5220 requires several host-computer clock cycles to read in the data, and its data lines must be held for the entire duration. The 74HCT374 flip flop buffers the data and holds the lines active as long as needed.
3. In response to the `/WS` line being asserted, the TMS5220 drives the `READY` line high. When this occurs, the ATF22V10C waits until the `READY` line transitions from high to low, which indicates that the TMS5220 has finished processing the input data. When the `READY` line goes low, the ATF22V10C then asserts `WS_RST`, the reset line for the `WS` flip flop. This brings `/WS` high, which tells the TMS5220 that the write operation is complete. It also disables the output enable on the 74HCT374 that was buffering the data. In this way, `/WS` remains asserted until the TMS5220 indicates it is done reading the data from the buffers, as indicated by the `READY` line.
4. During this entire sequence, the ATF22V10C asserts the `BUSY` line for the board, which can be read by the host computer at address `0xFE_2` via the 74HCT245. This allows the host computer to query the board to see if the TMS5220 is done processing data.

A similar sequence is used when reading data from the TMS5220, centered around the `/RS` line.


### Undocumented Timing Issues
The TMS5220 appears to have some timing requirements that are not documented in its datasheet. These requirements were exposed by the Minimal 64x4, which runs at a clock speed faster than the typical host computers for which the TMS5220 was designed. These undocumented timing requirements were determined empirically and could likely be tweaked further:

* **Read Cycle for Status Transfers to External Speech Data Write**
  The datasheet indicates that the `t_wait` after `/RS` goes high is 12 µs. However, I found that something closer to 175 µs was needed between a status read and a subsequent external speech data write.

* **External Speech Data Write to Read Cycle for Status Transfers**
  Similarly, extra time is needed between writing external speech data and then reading the TMS5220 status. Again, I found that about 175 µs was needed.

These values were “tweaked until it worked.” The issues seem related to switching modes of interaction with the TMS5220. By contrast, writing an external speech data byte after a prior external speech data byte does adhere to the timing diagrams in the TMS5220 datasheet. Further experimentation might uncover better or smaller delays for these undocumented requirements.

## Software Design
The provided `speechlib.min64x4` file contains all the subroutines needed to use this board on the Minimal 64x4 home computer:

* `speech_init`
  Call this to initialize the board.

* `speak_synchronous`
  Speaks the LPC data in a passed buffer synchronously. Push the following arguments on the stack (in push order):
  1. **Buffer End Address** – 2-byte end address of the data buffer
  2. **Buffer Start Address** – 2-byte start address of the data buffer

* `speech_start`
  Speaks the LPC data in a passed buffer asynchronously, after synchronously waiting for any currently active talking to finish. Push the following arguments on the stack (in push order):
  1. **Buffer End Address** – 2-byte end address of the data buffer
  2. **Buffer Start Address** – 2-byte start address of the data buffer

* `speech_start_if_not_talking`
  Speaks the LPC data in a passed buffer asynchronously if there is no currently active speech. Push the following arguments on the stack (in push order):
  1. **Buffer End Address** – 2-byte end address of the data buffer
  2. **Buffer Start Address** – 2-byte start address of the data buffer

* `speech_update_buffer`
  Call this periodically (about every 1 ms) to enable asynchronous speech. No arguments.

* `speech_get_status`
  Gets the contents of the TMS5220 status register, which is returned in the `A` register. The following bit masks are defined to help interpret the results:
  * `SPEECH_STATUS_TALK` – The TMS5220 is currently talking.
  * `SPEECH_STATUS_BUFFER_LOW` – The TMS5220 FIFO buffer is half-empty. More LPC data needs to be sent.
    *_Note_*: The `speech_update_buffer` function automatically updates the FIFO if needed.
  * `SPEECH_STATUS_BUFFER_EMPTY` – The TMS5220 FIFO buffer is empty.


# Compiling the Software
All software found in the [software](./software/) directory is compiled with [BespokeASM](https://github.com/michaelkamprath/bespokeasm), which supports the Minimal 64x4. [Carsten Herting](https://github.com/slu4coder), the creator of the Minimal 64x4, also provides an assembler for it, but that assembler is quite minimal. One key shortcoming is the inability to import library code into the current compilation. BespokeASM enables importing libraries and includes several other features that this project’s software depends on.

To compile the speech code in this repository, use the following command:

```sh
bespokeasm compile -c /path/to/bespokeasm/examples/slu4-minimal-64x4/slu4-minimal-64x4.yaml -n -p -t intel_hex lpc_player.min64x4
```
Where `/path/to/bespokeasm` is the path to the BespokeASM repository on your computer. This will produce an Intel HEX representation of the compiled program, which can then be sent to the Minimal 64x4 using its `receive` command.

# Resources
The following resources I found useful when putting together this project

* [TMS5220 Datasheet](./datasheets/tms5220.pdf) - Always consult the datasheet first.
* [Thierry Nouspikel's "The Speech Synthesizer module"](https://www.unige.ch/medecine/nouspikel/ti99/speech.htm) - A detailed overview of how the TMS5220 is used in the TI 99/4a.
* [TI 99/4a Development forum on AtariAge](https://forums.atariage.com/topic/352156-looking-for-help-on-tms5220-circuit-software-design-and-usage/) - These folks are legends, helpful and knowledgable about anything related to Texas Instruments.
