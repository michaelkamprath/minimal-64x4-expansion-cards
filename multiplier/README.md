# Integer Multiplication Accelerator for the Minimal 64x4 Computer
This project enables hardware-accelerated multiplication on the [Minimal 64x4 Home Computer](https://github.com/slu4coder/Minimal-64x4-Home-Computer/), exploring how multiplication works at a fundamental level in both software and hardware, using the Minimal 64x4 homebrew computer as a real-world test platform. Starting with a classic shifted-add multiplication algorithm implemented entirely in assembly, the project demonstrates both the mechanics and performance limits of software-based multiplication on an 8-bit CPU.

To dramatically improve performance, a dedicated hardware multiplier card is designed and built using discrete 74-series logic. The board implements a fully combinational 8-bit × 8-bit multiplier with memory-mapped I/O, allowing the CPU to offload multiplication and receive a 16-bit result in just a few clock cycles. The design is then extended in software to support full 16-bit × 16-bit multiplication through careful carry management.

The impact of hardware acceleration is demonstrated using a Mandelbrot renderer, where thousands of multiplications per frame make performance differences immediately visible. By comparing software-only and hardware-accelerated runs on real hardware, the project shows how a simple, purpose-built coprocessor can transform computational performance on a retro-style system.

# Hardware Multiplier Design
The hardware multiplier is a memory-mapped expansion board for the Minimal 64x4 that implements an 8-bit × 8-bit unsigned multiplication entirely in combinational logic. The design uses a planar array of 74AHCT08 AND gates to generate all partial products, followed by a cascade of 74HCT283 4-bit adders to sum those partial products and propagate carries. Because the multiplier core is purely combinational, no clocking, state machine, or control logic is required; once valid operands are presented, the result settles after the inherent gate propagation delay.

Operands are latched from the system data bus using 74AHCT374 registers to isolate the combinational logic from the shared bus and to ensure stable inputs during computation. The 16-bit result is continuously driven internally and presented to the host system through 74AHCT245 bidirectional bus transceivers when the board is selected. Address decoding is handled using 74HCT688 magnitude comparators, with a small programmable logic device (ATF16V8) providing glue logic for timing, output enable control, and bus arbitration.

Worst-case propagation delay is dominated by carry propagation through the adder array and is approximately 160 ns, corresponding to roughly two CPU cycles on an 8 MHz Minimal 64x4. From the CPU’s perspective, multiplication is reduced to a small number of memory writes and reads, enabling a substantial performance improvement over the software-only shifted-add implementation.

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