# General Assembler Syntax
The general syntax for a line of assembly code is one of the following forms:

```asm
{label:} instruction {operands} {; comment}

constant_assignment {; comment}

directive {; comment}

{directive} label: {; comment}

{label:} data_directive {data_value} {; comment}

; {comment}
```

Items in curly braces `{..}` are optional.

Additional notes:

* Comments are prefixed with a semicolon `;`. Any characters after and including the first semicolon on a line are considered comments.
* Each line may contain multiple instructions, labels, or directives, with these limitations:
    * Only one line comment is recognized per line.
    * Instructions can only be followed by other instructions or a line comment.
    * Preprocessor directives cannot share a line with any instruction, label, or other directive.
    * _While there may be scenarios where multiple instructions per line are useful, this is generally discouraged as it can make code harder to debug and error messages more confusing._
* Whitespace is generally ignored except where needed to separate parts of a line. Indentation can be used for readability with no impact on assembled code.
* There is no explicit limit on line length.
* File extensions are not interpreted by the compiler, but are used for syntax highlighting if supported by your editor. See [Installing Language Extensions](https://github.com/michaelkamprath/bespokeasm/wiki/Installation-and-Usage#installing-language-extensions) for more information.


## Numeric Literals
Numeric literals, whether immediate values or memory addresses, can be written in decimal, hexadecimal, binary, or as single character ordinal values:

| Type | Syntax |
|:--|--:|
| Decimal | `124` |
| Hex | `$7C` |
| Hex | `0x7C` |
| Hex | `7CH` |
| Binary | `b01111100` |
| Binary | `%01111100` |
| Character Ordinal | `'\|'` |

### Numeric Expressions

Numeric expressions that can be resolved at compile time are supported. A numeric expression can include numeric literals, address labels, constant labels, or numeric operators. Supported operators:

| Operator | Description | Comment |
|:-:|:--|:--|
| `+` | Addition | |
| `-` | Subtraction | If there is no value to the left of the operator, then the value to the right of the operator is negated. |
| `*` | Multiply | |
| `/` | Divide | |
| `%` | Modulo | Should be surrounded by whitespace to avoid confusion with binary numbers. |
| `&` | Bit-wise AND | |
| `\|` | Bit-wise OR | |
| `^` | Bit-wise XOR | |
| `>>` | Right shift | Shifts the bits of the value left of the operator right the number of times given by the value right of the operator. |
| `<<` | Left shift | Shifts the bits of the value left of the operator left the number of times given by the value right of the operator. |
| `BYTEx(..)` | Byte value | Returns the value of the `x`-th byte of the expression contained in parenthesis. The `x` can only be a single digit, so `BYTE0(..)` through `BYTE9(..)` are allowed. Uses 0-based indexing where 0 is least significant byte regardless of endian setting. |
| `LSB(..)` | LSB | Equivalent to `BYTE0(..)` |
| `(..)` | Grouping | Parentheses must be paired. |

Note that numeric expressions are not the same thing as an offset for a register indirect [addressing mode](#addressing-modes), though the offset value can be expressed as a numeric expression.

### Single Character Ordinals
The ASCII value of a single character may be used as an integer anywhere a numeric expression is allowed. Indicate a character ordinal by bounding the character with single quotes `'`. This does not work for multiple characters, nor with double quotes `"`.

## Labels
A label is a string resolved at compile time to a specific numeric value. Labels can use alphanumeric characters and underscores `_`, but cannot start with a number. All labels must be unique. A label cannot have the same name as any directive, and non-register labels cannot share a name with a register label.

### Label Types
#### Address Label
An address label represents a specific address in the byte sequence being assembled. It does not generate byte code on its own, but can be used as an instruction argument. The address is implied by its position in the code.

A label is any case-sensitive alphanumeric string immediately preceding a colon `:`. Only one label is allowed per line, but a label can be followed by a directive or instruction on the same line. For example:
```asm
a_label: .byte $22        ; directive on same line as label
```

#### Constant Label
A constant is a special label with an explicitly assigned numeric value. Constants can be placed anywhere in the code. Assignment uses `=` or `EQU`:

```asm
constant_var = 10204
constant_var EQU 10204
```

Constants may be assigned either numeric literals or any numeric expression resolvable at compile time (including other labels). Constant labels are case-sensitive.

#### Register Labels
A register label is defined in the [instruction set configuration file](./Instruction-Set-Configuration-File). It represents hardware registers in instruction operands. Address and constant labels cannot use a string declared as a register label.

### Label Scope
Both address labels and constant labels can be defined to be applicable only in a given scope. A scope defines to what extent a label is visible and usable by other lines of code. The allowed scopes are:

* **Global** - Visible everywhere. These labels are not prefixed with a `.` or `_`.
* * **Named** - User-defined scopes with custom prefixes that can be shared across files. See [Named Label Scopes](#named-label-scopes).
* **File** - Visible only in the same file. Use a leading `_` to indicate file scope.
* **Local** - Visible only between two non-local labels in the same file, or between a non-local label and an `.org` directive, or between a non-local label and the end of file. Use a leading `.` to indicate local scope.
  * Local labels cannot be defined before the first non-local label or between a `.org` and a non-local label.


**Scope Precedence:**
When resolving symbols, BespokeASM follows this precedence order:
1. Local scope (`.` prefix)
2. Named scopes in reverse activation order (uses named scope's defined prefix)
3. File scope (`_` prefix)
4. Global scope (no prefix)

#### Named Label Scopes
Named label scopes allow you to create custom symbol namespaces with user-defined prefixes. This is useful for organizing symbols in libraries and larger projects, making them accessible across multiple files.

##### Creating a Named Scope
The syntax for creating a new named label scope is:
```asm
#create-scope "scope-name" prefix="prefix_"
```
Where:
* `scope-name`: The name of the scope being created. This name may not contain white space. Using the same `scope-name` across multiple `#create-scope` statements in the same compilation is considered to be an error.
* `prefix`: The string that symbols need to be prefixed with to be included in the namespace. The `prefix=...` parameter is optional. If not included, the prefix value defaults to the underscore character (`_`). Named scopes with `_` prefix take precedence over built-in file scope. Scope prefixes cannot start with `.` to avoid confusion with local scope.

When this directive is encountered in compilation, the named scope is also activated as if the `#use-scope` directive was used.

**Important:** Labels and constants can only be **created into** a named scope in the same file where that named scope was created. This restriction allows libraries to control their namespace. If a different file activates the named scope and defines a label with that scope's prefix, the label will fall back to the normal scope hierarchy (global, file, or local scope) instead of being added to the named scope. Labels already in the named scope can still be **referenced** from any file that activates that scope.

##### Using a Named Scope
In order to activate a named label scope to be used in the current compilation file, the following syntax is used:
```asm
#use-scope "scope-name"
```
When `#use-scope` is encountered in compilation, then labels in the identified named label space are resolved on subsequent code lines until the named label space is deactivated or the end of the file. Name label space activation is valid only in the current file and does not project into any files loaded using `#include`.

**Forward References:** Named scopes can be activated with `#use-scope` **before** they are created with `#create-scope`. This is intentional and enables flexible library workflows where a main file can declare its dependencies at the top, then include library files that define those scopes later. The assembler will validate that all used scopes are eventually defined during compilation.

##### Deactivating a Named Scope
```asm
#deactivate-scope "scope-name"
```


##### Examples

**Basic Usage:**
```asm
; Define a graphics scope
#create-scope "graphics" prefix="gfx_"
#use-scope "graphics"

gfx_screen_width: .2byte 320    ; Goes to graphics named scope
global_config: .byte 100        ; Goes to global scope (no matching prefix)
_file_local: .byte 0            ; Goes to file scope (no matching prefix)

; Named scopes can be shared across files via #include
```

**File Restriction Example:**
```asm
; === library.asm ===
#create-scope "mylib" prefix="lib_"

lib_init: .byte 1               ; Creates label in mylib named scope
lib_version: .byte 2            ; Creates label in mylib named scope

; === main.asm ===
#use-scope "mylib"              ; Activate the scope for resolution
#include "library.asm"

; Can reference library labels
jmp lib_init                    ; Works! Resolves to label in mylib scope

; But cannot create labels into the library's scope
lib_custom: .byte 3             ; Does NOT go to mylib scope!
                                ; Falls back to global scope instead
```

**Underscore Prefix for Exporting Internal Symbols:**
```asm
; === mathlib.asm ===
; Use underscore prefix to "export" what would normally be file-local symbols
#create-scope "mathlib" prefix="_"

_multiply: .byte 100            ; Goes to mathlib named scope (not file scope!)
_divide: .byte 200              ; Goes to mathlib named scope (not file scope!)
_internal_flag: .byte 0         ; Also goes to mathlib named scope

; === main.asm ===
#use-scope "mathlib"            ; Activate mathlib scope
#include "mathlib.asm"

; Can now reference these "exported" symbols
call _multiply                  ; Works! Resolves from mathlib named scope
call _divide                    ; Works! Resolves from mathlib named scope

; Note: Named scope with _ prefix takes precedence over built-in file scope
```

**Forward Reference Pattern:**
```asm
; === main.asm ===
; Declare dependencies before they're defined
#use-scope "graphics"
#use-scope "audio"

gfx_buffer: .2byte 640          ; Tries to use graphics scope (not created yet)
                                ; Falls back to global since scope not created here

#include "graphics_lib.asm"     ; This file creates the "graphics" scope
#include "audio_lib.asm"        ; This file creates the "audio" scope

; Now can reference labels from those scopes
jmp gfx_init                    ; Resolves from graphics named scope
call snd_play                   ; Resolves from audio named scope
```



## Memory Zones
A named memory zone is a contiguous address range in the ISA's address space, identified by an alphanumeric string. Additional conditions:

* A named memory zone must be completely contained by the allowed memory space of the configured ISA.
* Multiple named memory zones may overlap with each other
* When byte code is assembled, multiple byte codes assigned to the same absolute memory address is a fatal error.
* Named memory zones are a compile time construct, and are intended to only be a means to manage memory ranges and byte code memory locations in assembly code.
* Memory zones have a start and end absolute memory address. Byte code assigned to that memory zone with an absolute address outside of the memory zone's range will be an error.
* A memory zone's name cannot be also used for any label.

### Global Memory Zone
By default, a memory zone named `GLOBAL` is defined to be the full range of memory addresses allowed by the instruction set configuration file. For example, if the ISA defines a 16-bit address type, then the `GLOBAL` memory zone will be addresses `0x0000` though `0xFFFF`. 

The `GLOBAL` memory zone can be redefined in the ISA configuration to be a subset of what is permitted by the memory address bit size.

## Instructions
Instruction are converted into byte code. It is composed of a specific instruction mnemonic and an option list of operands according to this format:
```
MNEMONIC [OPERAND1[, OPERAND2[...]]]
```

* Instructions mnemonics may compose of any alphabetic character or the following special characters: `.`. The first character of an mnemonic must be a alphabetic character.
* Instruction operands are separated by a comma
* Instruction operands supported types are configured in the [Instruction Set Configuration File](./Instruction-Set-Configuration-File#instructions).
* Instruction matching is not case sensitive.


### Addressing Modes
BespokeASM supports several addressing mode notations for instruction operands, though the precise meaning of each is defined by the instruction set configuration file and the hardware that the instruction set will run on.  Explained here is the nominal application of each addressing mode notation.

| Mode | Notation | Description | Decorator Placement | Hardware Expectations |
|:-:|:-:|:--|:-:|:--|
| Immediate |`numeric_expression` | A constant value to be used as an operand. The constant value is indicated by a numeric expression. | - | Values embedded I program byte code should be generally readable. |
| Indirect | `[numeric_expression]`| A value that resides at a memory address indicated by a constant value. The constant value memory address is indicated by a numeric expression. | - | Ability to set a memory address register or similar. |
|Deferred | `[[numeric_expression]]` | The numeric constant value indicated by a numeric expression represents an address at which is the value of another address  where the value of interest resides. Basically, this is a doubly dereferenced memory address. Note the use of double square brackets in the notation. | - | Ability to follow a doubly dereferenced memory address. |
| Register | `register_label` | The value in a specified register. The register is indicated by a [register label](#register-labels). | Adjacent to register label.</br>_e.g.:_ `a++` | Hardware registers that are generally accessible. |
| Indexed Register | `register_label + offset_operand ` | Indicates a value that is the combination of the register value and the offset operand value. The combination is nominally a sum (`+` operator). | - | Ability to combine a register value with any configured offset operand source. |
| Indirect Register | `[register_label + offset]`| The specified register contains a memory address where the value is. An offset can be provided which should be added to the value in the register get the memory address where the desired value is. The register is indicated by a [register label](#register-labels), and the offset is provided as a numeric expression and follows the register label with a `+` or `-` sign in between it and and the register label. | Adjacent to square brackets.</br>_e.g.:_ `[sc+5]++` | Hardware registers that can set the memory address used to access memory devices. In order to support offsets, there should be the ability to produce a memory address by adding a value to the register value without necessarily changing the register value. |
| Indirect Indexed Register | `[register_label + offset_operand]` | Similar to **Indirect Register**, except that the offset can be set by any other addressing mode operand. When the configure offset operand is a numeric type, this behaves the same as **Indirect Register** except that the offset can only be `+` to the register, and there are no bounds checking on the value. The true value of this addressing mode is when the offset operand is configured to be **Register**, **Indirect Register** or **Indirect** value. | Adjacent to square brackets.</br>_e.g.:_ `[sc+i]++` | Similar hardware needs as **Indirect Register**, with the general ability to set the offset value from any configured offset operand source. |
| Relative Address | `numeric_expression` or `{numeric_expression}` | Generates a relative address offset which is the difference between the expression value of this operand and the address value of current instruction, where this the current instruction's address value can be either be the program counter value before the instruction begins, or the program counter value after all machine code for the instruction has been loaded. Useful for relative jumps or data moves. Notation can be configured. |- | Should be able to do offsets against the program counter value. |

#### Decorators
Addressing mode notation that supports decorators are indicted in the table above. A decorator can be used to modify an operand or even the instruction's action on the operand. The precise meaning depends on how the instruction set is configured. A prefix decorator typically indicates that the modification should occur before the instruction is executed, while a postfix decorator indicate the modification occurs after the instruction occurs. **BespokeASM** will recognize an operand with a decorator as a whole different operand than if the decorator wasn't present. 

For example, consider the following two instructions:
```asm
mov [a],b
mov [a]+,b
```
A instruction set could be configured such that the first instruction means to move the value in register `b` into memory at the address indicated by register `a` (Indirect Register addressing mode), while the second instruction with the decorate could mean to move the value in register `b` into memory at the address indicated by register `a` and then increment the value in register `a` after the value copy has occurred. 

Here are the decorators that **BespokeASM** supports. There specific mean is configured in the instruction set configuration.

| Decorator | Typical Meaning | Prefix | Postfix |
|:--|:--|:-:|:-:|
|`+`| Increment | ✓ |  ✓ |
|`-`| Decrement | ✓ |  ✓ |
|`++`| Increment | ✓ |  ✓ |
|`--`| Decrement | ✓ |  ✓ |
|`!`| Custom | ✓ |  ✓ |
|`@`| Custom | ✓ |  ✓ |

## Directives
Directives tell the assembler to do specific things when creating the byte code. Directives start with a period `.` or a hash `#`. 

### Bytecode Addressing
#### Memory Zone Scope
By default, code in any given source file is assembled into the `GLOBAL` memory zone. To set the current memory zone scope to something different, the following directive is used:

```asm
.memzone <memory zone name>
```

Note that the `GLOBAL` memory zone name can be used this directive. Subsequent assembly code lines will be compiled into the indicated memory zone scope until the end of the current assembly file or another directive that changes the memory zone scope. Addresses assigned to the byte code will be per the code ordering. 

Non-contiguous uses of a given memory zone scope will be compiled as if the assembly code in each use instance was concatenated together in the order processed by the assembler.

If a source file that is currently using a non-`GLOBAL` memory zone includes another source file, that included source file will be compiled into the `GLOBAL` memory zone scope per normal file processing as described above. When compilation returns to the original source file that included the additional source file, compilation will continue using the same memory zone scope that was active when the `#include` directive was processed. This means that source files must always declare any non-`GLOBAL` memory zone scope they wish to use, and such declarations only persist for the scope of that source file.

#### Relative Origin within a Memory Zone
A relative origin within a memory zone can be set with the `.org` directive:

```asm
.org <address offset value> "<memory zone name>"
```

Where `<address offset value>` is the positive offset from the start of the specific memory zone, and `<memory zone name>` is the optional name of a memory zone. The `<memory zone name>` value is denoted by quotes so as to offset it from the `<address offset value>`, especially if that was set with an expression.

As an example, if a memory zone named "variables" is defined to be the range of `0x2000` through `0x2FFF`, then:

```asm
.org 0x0100 "variables"
```

Would be the same as setting the current origin to and absolute value of `0x2100`. 

When using `GLOBAL` as the `<memory zone name>` then `<address offset value>` will be interpreted as an offset form the start of the `GLOBAL` memory zone as it would with any other named memory zone. If the `GLOBAL` memory zone has not be redefined, the net effect is the same as using `.org` with an absolute address. However, if the start address of the `GLOBAL` memory zone has been redefined in the ISA configuration file, then `<address offset value>` will be applied as an offset from the redefined start of `GLOBAL`.

The effective absolute address represented in this form of the `.org` directive is validated against the overall valid address range defined by the `GLOBAL` memory zone. If the absolute address is outside this range, then **BespokeASM** will emit an error.

#### Absolute Origin
Using the `org` direction without specifying a `<memory zone name>` will cause the `<address offset value>` to be interpreted as an absolute address.

For example:

```asm
.org $3400
```

Will set the current address to $3400. This is an absolute address value and not an offset to the `GLOBAL` memory zone.

The address represented in this form of the `.org` directive is validated against the overall valid address range defined by the `GLOBAL` memory zone. If the address is outside this range, then **BespokeASM** will emit an error.

#### Page Align
The address used for the next emitted byte code can be set to the next arbitrarily-defined page boundary, which would be an address value that is a multiple of a given number. This is done using the `.align` directive with this format:

```asm
.align <page size>
```

Where `<page size>` is an optional expression indicating the size of the page the next byte code address should align to. If the `<page size>` expression is not provided, then the page size define in the instruction set configuration will be used.

### Bytecode
There are a few byte code generation directives supported:

| Directive | Description |
|:--|:--|
| `.fill N, Y` | Fills the next `N` words with the byte value `Y` |
| `.zero N` | Shorthand for `.fill N, 0` |
| `.zerountil X` | Fills the next words up to and including address `X` with the value of `0`. Will emit nothing if address `X` is less than the address location of this directive. |

### Data Directives
A data directive allows for explicitly set byte code. Like an instruction, its relative position in the assembly code defines its memory address, but unlike the instruction the byte code edited is directly defined in the assembly code. When paired with a label, a data directive can be used to define variables and other memory blocks.

The data directives have several forms, each indicating how much data is being defined:

| Directive | Data Value Size | Data Length | Multi-Word Endianness |
|:--|:-:|:--|:--|
| `.byte` | 1 byte | Variable | N/A |
| `.2byte` | 2 bytes | Variable | Default |
| `.4byte` | 4 bytes | Variable | Default |
| `.8byte` | 8 bytes | Variable | Default |
| `.cstr` | 1 byte | Variable | N/A |
| `.asciiz` | 1 byte | Variable | N/A |

The syntax of usage is simply the directive followed the a data values to be written. More than one value can be provided by a comma separated list of values or labels/constants. The value assembled into the byte code will be masked by the data value size of the directive.

The `.byte`, `.cstr`, and `.asciiz` directives can be used to define character strings delineated by a `"` or `'`. Quotes and apostrophes within the quoted string should be escaped. The data values generated will be the ASCII values for each character in the string. Python-style character escapes (e.g., `\t`, `\n`, `\x21`) can be used. The `.cstr` and `.asciiz` directives can be used only with strings and will appends a configurable byte value to the end of the string. This terminating byte value defaults to zero (`0`), but [can be configured to be a different value](https://github.com/michaelkamprath/bespokeasm/wiki/Instruction-Set-Configuration-File#general).

For multi-byte types (`.2byte`, `.4byte`, etc), the multi-word endianness representation of each individual value uses the configured default multi-word endianness specified in the instruction set configuration file. 

This example includes a label to be used to make the data's address usable elsewhere in the assembly code:
```asm
const_value = $BE

single_bytes:
    .byte $DE
    .byte $AD
    .byte const_value
    .byte $EF
byte_list:
    .byte $DE, 0xAD, const_value, $EF
str_with_no_terminating_null:
    .byte "It\'s a test string"
str_with_terminating_null:
    .cstr "It\'s a test string"

int16_value:
	.2byte $dead, $beef

int32_value:
	.4byte $deadbeef
```
### Data and Word Size
Since **BespokeASM** is designed to be used with a variety of different CPUs, it can [support differing word sizes](./Instruction-Set-Configuration-File#data-words-and-endianness). The data directives describe data in specific 8-bit byte sizes. If the CPU's word size is larger than the individual data value, then each individual data value will be placed into the least signficant bits of a distinct word by default. For example, the following `.byte` declaration:

```asm
.byte $01, $02, $03, $04
```

Would be represented in 16-bit words as follows:

```
address 0: 0x0001
address 1: 0x0002
address 2: 0x0003
address 3: 0x0004
```

If the word size is smaller than the individual data value, then the data value will be subdivided into the least number of words needed to represent the data size, and then laid out according to the multi-word endianness. For example, if the word size is 16-bits and the multi-word endianness is `big`, then the following `.4byte` declaration:

```asm
.4byte $01020304
```

Would be represented in 16-bit words as follows:

```
address 0: 0x0102
address 1: 0x0304
```

#### String Data and Byte Packing
By default, strings in `.byte`, `.cstr`, and `.asciiz` directives are expanded so that each character (and the terminator for `.cstr`/`.asciiz`) is placed in its own word, regardless of the word size. For example, the following `.cstr` declaration:

```asm
.cstr "Hello, World!"
```

Would be represented in 16-bit words as follows:

```
address  0: 0x0048
address  1: 0x0065
address  2: 0x006c
address  3: 0x006c
address  4: 0x006f
address  5: 0x0020
address  6: 0x0057
address  7: 0x006f
address  8: 0x0072
address  9: 0x006c
address 10: 0x0064
address 11: 0x0021
address 12: 0x0000
```

Note that the last word is a terminating null byte.

_Note: The default behavior can be changed by enabling the [`string_byte_packing`](./Instruction-Set-Configuration-File#string-byte-packing-for-data-directives) option in the configuration file. When enabled, quoted strings in `.byte` and `.cstr` will be packed tightly into words, and the padding behavior is controlled by the [`string_byte_packing_fill`](./Instruction-Set-Configuration-File#string-byte-packing-for-data-directives) option. See the [Instruction Set Configuration File documentation](./Instruction-Set-Configuration-File#string-byte-packing-for-data-directives) for details and examples._

#### Embedded Strings
If the features [is enabled for the instruction set](https://github.com/michaelkamprath/bespokeasm/wiki/Instruction-Set-Configuration-File#general), a quoted string using double quotes `"` may be placed anywhere in the assembly code and byte code will be generated based on the UTF8 values of the string's characters follow with the string termination character configured for the ISA.  This effectively allows the creation of a `.cstr` without the `.cstr` directive. Note that single quotes `'` cannot be used for creating embedded strings.

### Preprocessor
#### Include Other Files
Additional assembly files other than the target file indicated in the command invocation can be included in the compilation. This is done with the `#include` preprocessor directive. The specific format is:
```asm
#include "filename.asm"
```
Where `filename.asm` is the name of the file desired to be included. BespokeASM will search the include directories to find a file with the indicated filename. The include directory list includes the directory that contains the target file identified on command invocation, and any additional include directories identified by arguments to the command invocation.

When an assembly file is included by this directive, it is functionally equivalent to the the contents of the included file be present where the `#include` directive is. If `.org` directives are used in the included file, care should be taken such that the address of instructions do not collide between source files. BespokeASM will error if it detects that two or more instructions occupy the same address.

The `#include` directive is affected by [conditional compilation](#compilation-control). If the `#include` directive appears inside an inactive conditional block (e.g., within `#if 0` ... `#endif`), the file will not be included.

The inclusion of assembly files can be nested. However, BespokeASM will error if any given file ends up being included more than once.

#### Require Language Version
An assembly source file can require a version check of the assembly language version as identified `identifier` key of [the General section](./Instruction-Set-Configuration-File#general) of the assembly language configuration file being used for compilation. This is done using a `#require` preprocessor directive. Failing to meet the version requirement will cause the assembly process to exit with an error.

The `#require` directive supports two syntax formats:

**Legacy String Format:**
```asm
#require "language-id comparator version-string"
```
where:
* `language-id` is the language `name` value in the `identifier` block of the general configuration section.
* `comparator` is a comparison operator, such as `>=`, `>`, `==`, etc. The most common comparison operator will be `>=`.
* `version-string` is a semantic version string, e.g. `1.2.3`

Examples:
```asm
#require "test-lang >= 0.5.0"
#require "test-lang < 1.0.0"
```

**Symbol-Based Format:**
```asm
#require symbol-expression comparator symbol-expression
```
This format uses the same [symbol expression](#language-version-symbols) syntax as compilation control directives and supports the built-in language version symbols.

The version check is done at the moment the line with the `#require` preprocessor directive is processed. This means any given code file can have multiple `#require` checks. This is useful if you want to enforce a version range.

Examples:
```asm
#require __LANGUAGE_NAME__ == test-lang
#require __LANGUAGE_VERSION__ >= 2.0.1
```

#### Creating Memory Zones
A memory zone can be defined with the following directive

```asm
#create_memzone <memory zone name> <start address> <end address>
```

Where `<memory zone name>` is an alphanumeric string with no spaces which will serve as the memory zone name, `<start address>` is the absolute address of the start of the memory zone, and `<end address>` is the absolute address of the end of the memory zone. Both `<start address>` and `<end address>` must be defined with integer literals.

Any defined memory zone must be fully contained in the `GLOBAL` memory zone. Defining multiple memory zones with the same name is an error.

#### Macros
C-like preprocessor macros can be defined with the `#define` directive. The syntax is:
```asm
#define <symbol> <value>
```
Where `<symbol>` is the symbol for the macro that can be used elsewhere in the code, and `<value>` is the replacement value for wherever that symbol is used. The `<value>` can be left empty, which is equivalent to assigning an empty string to be the replacement value for the `<symbol>`. If the replacement value is intend to be interpreted as a string, it should be quoted with either single or double quotes.

Later, when a defined `<symbol>` is used in code, **BespokeASM** will immediately replace the `<symbol>`'s text with the defined `<value>` for that `<symbol>`. If the `<symbol>` is not defined when that line of code is read, then no replacement occurs. Symbol replacement is done recursively, so if one preprocessor macro symbol's replacement value is a string that contains another preprocessor mark symbol, that second preprocessor macro symbol is then replaced. This continues until nor symbol replacement occurs. An error will be generated if a symbol replacement loop is detected (e.g., symbol A is replaced by symbol B, which itself is replaced by symbol A).

Note that parametric preprocessor macro symbols (e.g., `FOO(x)`) are not allowed. While this feature can be used to create simple code macros in code, complex and even parametric macros should be created with [Instructions Macros feature](https://github.com/michaelkamprath/bespokeasm/wiki/Instruction-Set-Configuration-File#instruction-macros). Similarly, these preprocessor macros can be used to define constants in code, but the [constant label feature](#constant-label) is a better way to do that. The primary use case for preprocessor macros is to define symbols that can be used in [compilation control](#compilation-control).

#### Language Version Symbols
BespokeASM provides built-in preprocessor symbols that expose information about the language defined in the ISA configuration file. These symbols are automatically available for use in [compilation control](#compilation-control) directives and the symbol-based format of the [`#require` directive](#require-language-version).

The following built-in symbols are available:

| Symbol | Description | Example Value |
|:--|:--|:--|
| `__LANGUAGE_NAME__` | The language name from the ISA configuration | `sap1-lang` |
| `__LANGUAGE_VERSION__` | The complete semantic version string | `1.2.3` |
| `__LANGUAGE_VERSION_MAJOR__` | The major version number | `1` |
| `__LANGUAGE_VERSION_MINOR__` | The minor version number | `2` |
| `__LANGUAGE_VERSION_PATCH__` | The patch version number | `3` |

These symbols can be used in expressions within `#if`, `#elif`, and `#require` directives to create conditional compilation based on language version. When used in comparisons, language name comparisons are typically done as string comparisons, while version number comparisons are done as numeric comparisons.

**Examples:**
```asm
#if __LANGUAGE_NAME__ == sap1-lang
    ; SAP-1 specific code
#endif

#if __LANGUAGE_VERSION_MAJOR__ >= 2
    ; Use newer features available in v2+
#elif __LANGUAGE_VERSION_MAJOR__ == 1
    #if __LANGUAGE_VERSION_MINOR__ >= 5
        ; Use features available in v1.5+
    #else
        ; Fallback for v1.0-1.4
    #endif
#else
    ; Fallback for older versions
#endif

#require __LANGUAGE_VERSION_MAJOR__ >= 1
#require __LANGUAGE_VERSION_MAJOR__ < 3
```

**Important Note:** Mixed expressions that combine language version symbols with complex operators (`&&`, `||`, parentheses for grouping) are not supported. For example:
```asm
; ❌ NOT SUPPORTED - will generate an error
#if __LANGUAGE_VERSION_MAJOR__ >= 1 && OTHER_SYMBOL == 2
#if (SYMBOL == 4) && (__LANGUAGE_VERSION_MAJOR__ >= 0)

; ✅ SUPPORTED - use separate blocks instead
#if __LANGUAGE_VERSION_MAJOR__ >= 1
#if OTHER_SYMBOL == 2
    ; Both conditions are true
#endif
#endif
```

#### Compilation Control
Control over which lines of assembly get compiled can be done with the C-like `#if`, `#elif`, `#else`, `#ifdef`, `#ifndef`, and `#endif` preprocessor directives. Individual lines of assembly are braced by compilation controls preprocessor directives, which control whether those lines of assembly will be compiled or not. Each can be used as follows:

* `#if <symbol-expression> <comparison> <symbol-expression>` - Initiates a compilation control block by performing a comparison between two symbol expressions.
* `#if <symbol-expression>` - Initiates a compilation control block by comparing a symbol expression to the implied comparison of `!= 0`
* `#elif <symbol-expression> <comparison> <symbol-expression>` - Optional. Must follow and `#if` or `#elif` compilation control directive in sequence. Creates a subordinate condition block by performing a comparison between two symbol expressions.
* `#elif <symbol-expression>` - Optional. Must follow and `#if` or `#elif` compilation control directive in sequence. Creates a subordinate condition block by comparing a symbol expression to the implied comparison of `!= 0`.
* `#ifdef <symbol>` - Initiate a compilation control block by determining if a preprocessor macro symbol is defined
* `#ifndef <symbol>` - Initiate a compilation control block by determining if a preprocessor macro symbol is not defined
* `#else` - Option. Creates a subordinate condition block that evaluates true only if all previous subordinate condition blocks in the overall compilation control block are false. Must follow a `#if`, `#elif`, `#else`, `#ifdef`, or `#ifndef`.
* `#endif` - Terminates a compilation control block. Must come at the a compilation control block.

A `<symbol-expression>` is an [expression](#numeric-expressions) that uses preprocessor macros, [language version symbols](#language-version-symbols), numeric values, and mathematical operators, but not code labels (address labels, constants, etc). Symbol expressions used in compilation control processor directives must resolve to numeric or string values.

**Important:** BespokeASM does not support complex boolean operators (`&&`, `||`) or parentheses for grouping within individual `#if` or `#elif` directives. Each directive can only contain a single comparison. To achieve complex conditional logic, use nested `#if` blocks as shown in the examples above.

**Examples with user-defined symbols:**
```asm
#define SYMBOL1 "test-string"
#define SYMBOL2 57

#if SYMBOL1 == "my string"
    mov a,1
#elif SYMBOL2 > 50
    mov a,2
#else
    mov a,3
#endif
```
In this example, only the `mov a,2` line will get compiled.

**Examples with language version symbols:**
```asm
#if __LANGUAGE_NAME__ == sap1-lang
    ; Include SAP-1 specific optimizations
    ldi zero
    out
#elif __LANGUAGE_NAME__ == z80-lang
    ; Include Z80 specific code
    ld a, 0
    out (0), a
#endif

#if __LANGUAGE_VERSION_MAJOR__ >= 2
    ; Use new instruction available in version 2.0+
    push [stack_ptr]
#else
    ; Use legacy instruction for older versions
    mov [stack_ptr], sp
    push sp
#endif
```

#### Bytecode Emission Control
The `#mute` and `#unmute` preprocessor directives can be used to control whether the byte code derived from code is included in certain outputs, notably the binary image and pretty printing with theIntel Hex or MinHex formats. This is useful when you have code that defines symbols to be used else where but the byte code resulting from the definition of those symbols should not be part of the final byte code results.

To suppress the emission of byte code from the subsequent code lines in a file, use the `#mute` preprocessor directive. To restore the emission of byte code from subsequent code lines, use the `#unmute` or `#emit` preprocessor directives. The mute and unmute actions stack within the scope of a code file. That is, if for example two `#mute` preprocessor directives have ben included in code, two subsequent `#unmute` or `#emit` preprocessor director are needed to restore byte code emission. The `#mute` directive has no impact on byte code resulting from code loaded subsequently from an `#include` directive. [Conditional compilation](#compilation-control) will control whether a `#mute` or `#unmute` preprocessor directive is applied.

#### Compile-time Printing
During compilation you can emit informational messages using the `#print` preprocessor directive. This is useful for surfacing configuration state, feature flags, or progress while assembling.

Syntax:
```asm
#print "message"
#print <min-verbosity> "message"
```

Behavior:
- Prints the quoted string to standard output.
- Respects [conditional compilation](#compilation-control): messages are emitted only when the current condition block is active.
- Respects [bytecode emission control](#bytecode-emission-control): messages are suppressed while muted (`#mute`/`#unmute`).
- If `<min-verbosity>` is provided, the message is printed only when the assembler's log verbosity is greater than or equal to that integer as set by the `-v` command line option. If omitted, the message is always eligible to print (subject to the two controls above).

Examples:
```asm
#print "building utilities"
#print 2 "detailed build step"

#if FEATURE_ENABLED
  #print "feature enabled"
#endif

#mute
#print "this will not print"
#unmute
#print "mute lifted"
```

# Examples
## Ben Eater SAP-1
The following example using the instruction set for [Ben Eater's SAP-1 Breadboard CPU](https://eater.net/8bit). 
```asm
; Count by Loop
;
; For the Ben Eater SAP-1 breadboard CPU
;

zero = 0              ; constant value for 0
one = 1               ; constant value for 1

start:
  ldi zero            ; load value of 0 into A
  out                 ; display

add_loop:
  add increment       ; add current value at 0xF to A
  jc increment_step   ; increment the step if overflow
  out                 ; display
  jmp add_loop        ; loop

increment_step:
  lda increment       ; load current increment value
  add one_value       ; add 1 to increment value
  jc restart_loops    ; if it overflows, just reset everything
  sta increment       ; save updated increment value
  jmp start           ; restart counting

restart_loops:
  ldi one             ; load the value of 1 into register A
  sta increment       ; reset the increment value to 1
  jmp start           ; restart counting

one_value:
  .byte 1             ; 1 value needed for incrementing the increment value

increment:
  .byte 1             ; storage for the current increment value
```

## Recursion with Subroutines
Here is an example that employs an instruction set that enable subroutines (`call`, `rts`), a stack (`push`, `pop`) and indirect addressing modes. It uses 16-bit addressing and little endian. The example configuration file for this instruction set is here. Also assumes a memory map with `$0000` is the start of ROM and `$8000` is the start of RAM.

```asm

;
; Variables
;

.org $8000           ; variables should be in RAM
n_value:
  .byte 5            ; N value to calculate factorial for

;
; Code
;

.org 0               ; code goes in ROM
start:
  push [n_value]     ; push the value at n_value onto the stack
  call factorial     ; jump to the factorial subroutine
  out                ; factorial results are in A register. display it
  hlt                ; done

; factorial subroutine
;
; Input:
;   stack - function return pointer
;   stack+2 - The input N value to calculate factorial. A single 8-bit value
;
; Output:
;   A register - the results of the factorial calculation. A single 8-bit value
;
; Registers used: A
;
factorial:
  mov [sp+2],a      ; copy the N value to A register
  je .end,1       ; jump to f_stop if A is 1
  sub 1             ; subtract 1 from A to get (N-1)
  push a            ; put the n-1 value on the stack
  call factorial    ; recurse into factorial
  pop               ; remove the (N-1) value from stack
  push [sp+2]       ; push the N value on the stack
  push a            ; push the factorial(n-1) results on stack
  call multiply     ; call multiply subroutine
  pop               ; pop factorial(n-1) from stack
  pop               ; pop N-value from stack
.end:               ; local-scope label indicating the end of the subroutine
  rts               ; return from subroutine. Register A contains factorial(N)

; multiply subroutine
;
; Input:
;   stack - function return pointer
;   stack+2 - A single 8-bit value to multiply
;   stack+3 - A single 8-bit value to multiply
;
; Output:
;   A register - the results of the multiply calculation. A single 8-bit value
;
; Registers use: A, I
;
multiply:
  mov [sp+2],a     ; copy the multiplicand to A
  je .zero,0       ; jump to zero handler if multiplicand is 0
  mov a,b          ; copy multiplicand to B to set up for add loop
  mov [sp+3],i     ; copy multiplier to I
  dec i            ; decrement I for 0-based loop
  jc .zero         ; was multiplier zero? If so, carry was set on the dec so jump to m_zero
.loop:             ; local scope label indicating the start of the summation loop
  jz .end          ; jump to done if multiplier counter is now zero
  add b            ; add b to a
  dec i            ; decrement multiplier counter
  jmp .loop        ; restart addition loop
.zero:             ; local scope label indicating when a 0-multiplicand is handled
  mov a,0          ; set the return value to zero
.end:              ; local-scope label indicating the end of the subroutine
  rts              ; return from subroutine
```
