# `PLD` Files for Programmable Logic

## Compiling PLD
The `.pld` files in this directory may be compiled with [GALasm](https://github.com/daveho/GALasm), producing a `.jed` file amonst others. Then the `.jed` file is used to program a `ATF22V10C` or `GAL22V10`.

```sh
galasm minimal-64x4-speech-controller.pld
```

## Programming the 22V10
The `ATF22V10C(UES)` type should be used when programming the ATF22V10C with `minipro`.

```sh
minipro -p "ATF22V10C(UES)" -w minimal-64x4-speech-controller.jed
```

## More Information
More information about programmable logic devices can be found at these links:
* [Understanding Programmable Logic - The GAL22V10](https://retrobrewcomputers.org/n8vem-pbwiki-archive/0/82152589/Programmable%20logic%20devices%20-%20v1.6.pdf) - A good overviews of what programmable logic devices are.
* [Galette](https://github.com/simon-frankau/galette) - an alternative GAL PLD assembler
* [GAL-ASM-Starterkit](https://www.mikrocontroller.net/attachment/20953/GAL-ASM-Starterkit_Manual.pdf) - Documentation the PLD file syntax from another PLD assembler.
