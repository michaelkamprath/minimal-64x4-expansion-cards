# Repository Guidelines
This repo hosts expansion cards for the Minimal 64x4 Home Computer and Minimal 64x4 Redux Home Computer (both are 8-bit TTL “CPU”). Use these notes to keep hardware, software, and docs consistent.

## Project Structure & Module Organization
- Card directories: `expansion-backplane/`, `multiplier/`, `speech-synthesizer/`, each with `hardware/` KiCad projects (e.g., `expansion-backplane/hardware/expansion-backplane/expansion-backplane.kicad_pro`) plus board-specific `datasheets/`, `pld-files/`, `references/`, or `software/`.
- Shared KiCad assets: `kicad-libraries/` (`minimal-64x4.kicad_sym`, `minimal-64x4.pretty`). CPU and assembly references: `docs/`.
- Software sits with its board and uses `.min64x4`; avoid committing tool lockfiles (`*.lck`) or OS metadata.
- Documentation for the assembly language for both computer types can be found in the `docs/` folder.

## Coding Style & Naming Conventions
- Details on the syntax of assembly language using BespokeASM can be found in this repository file: docs/Assembly-Language-Syntax.md
- The complete instruction set for the Minimal 64x4 computer is documented in this repository file: docs/slu4-minimal-64x4.md
- Assembly:
  - start files with `#require "slu4-min64x4-asm >= 1.1.0"`
  - uppercase opcodes, colon-suffixed labels, snake_case symbols, semicolon comments;
  - prefer one instruction per line.
  - Use named constants for MMIO (e.g., `$FED0` multiplier registers) and `.memzone` for layout.
  - Fast branches (`FPA`/`FCC`/`FEQ`/`FCS`) require same-page targets—use `.align` before hot routines or split routines to keep fast-branch targets within the same 256-byte page.
- KiCad: keep project names aligned to directories, store custom symbols/footprints in `kicad-libraries/`, and version fabrication outputs in `gerbers_*` folders. Keep DRC/ERC settings with the project; avoid committing KiCad lockfiles.

### Assembly Compilation
Use BespokeASM with the Minimal 64x4 configuration. First, you need to obtain the Minimal 64x4 instruction set configuration for BespokeASM.
The current Minimal 64x4 instruction set configuration for BespokeASM can be downloaded from: https://github.com/michaelkamprath/bespokeasm/blob/main/examples/slu4-minimal-64x4/slu4-minimal-64x4.yaml
To download it to a local temporary file to be used when compiling with BespokeASM, the following shell command can be used:

```bash
# Download the config file for Minimal 64x4
curl -o /tmp/slu4-minimal-64x4.yaml https://raw.githubusercontent.com/michaelkamprath/bespokeasm/main/examples/slu4-minimal-64x4/slu4-minimal-64x4.yaml
```

Then to use that instruction set configuration to compile a `*.min64x4` assembly code file:

```bash
# Compile assembly file to Intel HEX format
bespokeasm compile -c /tmp/slu4-minimal-64x4.yaml -n -p -t intel_hex file.min64x4

# Common compilation options:
# -c: Configuration file (required)
# -n: No binary output
# -p: Pretty print
# -t intel_hex: Output Intel HEX format
# -v: Verbose output
# -I: Include path for library files
# -D: Define a preprocessor symbol, used in #if, #ifdef, etc; Add `-D USE_ACCELERATOR` when targeting the multiplier card.
# -W: Force code warnings to be treated as errors
```

### Graphics API:

Graphics functions on the Minimal 64x4 use dedicated zero-page memory locations for parameters, **not the stack**.
- **`GRAPHICS_X1` (0x0080):** 16-bit X1 coordinate (little-endian).
- **`GRAPHICS_Y1` (0x0082):** 8-bit Y1 coordinate.
- **`GRAPHICS_X2` (0x0083):** 16-bit X2 coordinate (little-endian).
- **`GRAPHICS_Y2` (0x0085):** 8-bit Y2 coordinate.

Always set these locations immediately before calling a graphics function. The graphics functions supported in the Minimal 64x4 operating system are:
- `_SetPixel` : Sets a pixel at position (GRAPHICS_X1, GRAPHICS_Y1)
- `_ClearPixel` : Clears a pixel at position (GRAPHICS_X1, GRAPHICS_Y1)
- `_Line` : Draws a line using Bresenham’s algorithm from position (GRAPHICS_X1, GRAPHICS_Y1) to position (GRAPHICS_X2, GRAPHICS_Y2)
- `_Rect` : Draws a rectangle at (GRAPHICS_X1, GRAPHICS_Y1) of size (GRAPHICS_X2, GRAPHICS_Y2)
- `_Clear` : Clears the visible video RAM (viewport)


## Testing Guidelines
- Software: rerun `bespokeasm compile` on modified `.min64x4`; ensure includes/defines match hardware (e.g., `USE_ACCELERATOR`). Generate Intel HEX for device loading and keep sample inputs small for quick bench tests.
- Hardware: run ERC on schematics and DRC on PCBs before exporting; verify net ties, power flags, and stackup settings per board revision. Capture bench notes or scope shots for timing-sensitive changes.

## Commit & Pull Request Guidelines
- Commits: short, present-tense imperatives (e.g., `update syntax for new bespokeasm`, `change pcb to 4-layer`); group related edits.
- PRs: summarize intent and affected paths, link related issues or forum threads, and attach artifacts—Gerbers/BOM for hardware changes, compiled HEX for software, ERC/DRC screenshots or logs for validation.
