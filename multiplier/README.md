# Integer Multiplication Accelerator for the Minimal 64x4 Computer
This project enables hardware-accelerated multiplication on the [Minimal 64x4 Home Computer](https://github.com/slu4coder/Minimal-64x4-Home-Computer/), exploring how multiplication works at a fundamental level in both software and hardware, using the Minimal 64x4 homebrew computer as a real-world test platform. Starting with a classic shifted-add multiplication algorithm implemented entirely in assembly, the project demonstrates both the mechanics and performance limits of software-based multiplication on an 8-bit CPU.

To dramatically improve performance, a dedicated hardware multiplier card is designed and built using discrete 74-series logic. The board implements a fully combinational 8-bit × 8-bit multiplier with memory-mapped I/O, allowing the CPU to offload multiplication and receive a 16-bit result in just a few clock cycles. The design is then extended in software to support full 16-bit × 16-bit multiplication through careful carry management.

The impact of hardware acceleration is demonstrated using a Mandelbrot renderer, where thousands of multiplications per frame make performance differences immediately visible. By comparing software-only and hardware-accelerated runs on real hardware, the project shows how a simple, purpose-built coprocessor can transform computational performance on a retro-style system.

# Hardware Multiplier Design
The hardware multiplier is a memory-mapped expansion board for the Minimal 64x4 that implements an 8-bit × 8-bit unsigned multiplication entirely in combinational logic. The design uses a planar array of 74AHCT08 AND gates to generate all partial products, followed by a cascade of 74HCT283 4-bit adders to sum those partial products and propagate carries. Because the multiplier core is purely combinational, no clocking, state machine, or control logic is required; once valid operands are presented, the result settles after the inherent gate propagation delay.

Operands are latched from the system data bus using 74AHCT374 registers to isolate the combinational logic from the shared bus and to ensure stable inputs during computation. The 16-bit result is continuously driven internally and presented to the host system through 74AHCT245 bidirectional bus transceivers when the board is selected. Address decoding is handled using 74HCT688 magnitude comparators, with a small programmable logic device (ATF16V8) providing glue logic for timing, output enable control, and bus arbitration.

Worst-case propagation delay is dominated by carry propagation through the adder array and is approximately 160 ns, corresponding to roughly two CPU cycles on an 8 MHz Minimal 64x4. From the CPU’s perspective, multiplication is reduced to a small number of memory writes and reads, enabling a substantial performance improvement over the software-only shifted-add implementation.


```text
                     ┌─────────────────────────┐
                     │        CPU / BUS        │
                     │  Address • Data • Ctrl  │
                     └───────────┬─────────────┘
                                 │
                        Memory-Mapped I/O
                                 │
        ┌────────────────────────▼────────────────────────┐
        │            Hardware Multiplier Board            │
        │                                                 │
        │  ┌───────────────┐      ┌────────────────────┐  │
        │  │ Input Latch A │◄─────┤                    │  │
        │  │  (8-bit)      │      │                    │  │
        │  └───────────────┘      │                    │  │
        │                         │                    │  │
        │  ┌───────────────┐      │  Combinational     │  │
        │  │ Input Latch B │◄─────┤  Multiplier Core   │  │
        │  │  (8-bit)      │      │  (AND Plane +      │  │
        │  └───────────────┘      │   Adder Array)     │  │
        │                         │                    │  │
        │                         │                    │  │
        │                         └─────────┬──────────┘  │
        │                                   │             │
        │                          16-bit Result          │
        │                                   │             │
        │                    ┌──────────────▼────────────┐│
        │                    │  Output Bus Transceivers  ││
        │                    │       (16-bit)            ││
        │                    └──────────────┬────────────┘│
        │                                   │             │
        │  ┌──────────────────────────┐     │             │
        │  │ Address Decode & Control │◄────┘             │
        │  │ (Comparators + PLD)      │                   │
        │  └──────────────────────────┘                   │
        └─────────────────────────────────────────────────┘
```
The overall structure of the hardware multiplier is shown in the block diagram above. The board interfaces with the CPU exclusively through memory-mapped I/O on the shared system bus. Operand values are written by the CPU into two 8-bit input latches (Operand A and Operand B), which isolate the multiplier core from bus activity and continuously present stable inputs to the combinational logic.

At the center of the design is a purely combinational multiplier core composed of an AND-gate partial-product plane followed by an adder array that collapses those partial products into a 16-bit result. Because the core has no internal state, clocking, or control signals, the result is produced automatically as signals propagate through the logic once the operands are latched.

The 16-bit result is connected to the system data bus through output bus transceivers that are enabled only during valid read cycles. Address decoding and timing control logic ensure the board responds only within its assigned address range and never drives the bus outside of those cycles. From the CPU’s perspective, the hardware multiplier appears as a small set of deterministic, memory-mapped registers, with computation latency bounded solely by combinational propagation delay.


# Compiling the Software
All software found in the [software](./software/) directory is compiled with [BespokeASM](https://github.com/michaelkamprath/bespokeasm), which supports the Minimal 64x4. [Carsten Herting](https://github.com/slu4coder), the creator of the Minimal 64x4, also provides an assembler for it, but that assembler is quite minimal (by design). One key shortcoming is the inability to import library code into the current compilation. BespokeASM enables importing libraries and includes several other features that this project’s software depends on.

To compile the speech code in this repository, use the following command:

```sh
bespokeasm compile -c /path/to/bespokeasm/examples/slu4-minimal-64x4/slu4-minimal-64x4.yaml -n -p -t intel_hex -D USE_ACCELERATOR mandelbrot.min64x4
```
Where `/path/to/bespokeasm` is the path to the BespokeASM repository on your computer. You may remove the `-D USE_ACCELERATOR` if you wish to compile the program to use software-only multiplication.

This will produce an Intel HEX representation of the compiled program, which can then be sent to the Minimal 64x4 using its `receive` command.

# Video

[![A Homebrew GPU for a Homebrew CPU](https://img.youtube.com/vi/M8dk0JpkrbY/maxresdefault.jpg)](https://youtu.be/M8dk0JpkrbY)