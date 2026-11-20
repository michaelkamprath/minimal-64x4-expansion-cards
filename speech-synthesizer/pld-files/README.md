# `PLD` Files for Programmable Logic

## Compiling PLD
There are two versions of the controller configuration. Program the one that matches the speech chip fitted to the board:

| File | For | `0xFE_3` reset |
| --- | --- | --- |
| `minimal-64x4-speech-controller-tms5220.pld` | TMS5220 | Not implemented; writes to `0xFE_3` are ignored |
| `minimal-64x4-speech-controller-tms5220c.pld` | TMS5220C | Implemented; see the board [README](../README.md) |

Do not use the TMS5220C version with an original TMS5220. It asserts `/RS` and `/WS` together to reset the chip, which is undefined for the TMS5220. The two versions are otherwise identical.

The `.pld` files in this directory may be compiled with either [GALasm](https://github.com/daveho/GALasm) or [Galette](https://github.com/simon-frankau/galette), a GALasm-compatible assembler that reads the same `.pld` files. Either one produces a `.jed` file amongst others. Then the `.jed` file is used to program a `ATF22V10C` or `GAL22V10`.

With GALasm:

```sh
galasm minimal-64x4-speech-controller-tms5220.pld
galasm minimal-64x4-speech-controller-tms5220c.pld
```

Or with Galette:

```sh
galette minimal-64x4-speech-controller-tms5220.pld
galette minimal-64x4-speech-controller-tms5220c.pld
```

## Programming the 22V10
The `ATF22V10C(UES)` type should be used when programming the ATF22V10C with `minipro`.

```sh
minipro -p "ATF22V10C(UES)" -w minimal-64x4-speech-controller-tms5220.jed
```

## More Information
More information about programmable logic devices can be found at these links:
* [Understanding Programmable Logic - The GAL22V10](https://retrobrewcomputers.org/n8vem-pbwiki-archive/0/82152589/Programmable%20logic%20devices%20-%20v1.6.pdf) - A good overviews of what programmable logic devices are.
* [GAL-ASM-Starterkit](https://www.mikrocontroller.net/attachment/20953/GAL-ASM-Starterkit_Manual.pdf) - Documentation the PLD file syntax from another PLD assembler.
