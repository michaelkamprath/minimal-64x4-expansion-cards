# slu4-min64x4-asm

slu4 Minimal 64x4 Home Computer

## General Information

### ISA Overview

| ISA Attribute | Value |
|:--|:--|
| **Description** | slu4 Minimal 64x4 Home Computer |
| **ISA Name** | `slu4-min64x4-asm` |
| **Version** | 1.1.0 |
| **File Extension** | *.min64x4 |

---

| Hardware Attribute | Value |
|:--|:--|
| **Address Size** | 16 bits |
| **Word Size** | 8 bits |
| **Page Size** | 256 addresses |
| **Default Origin Address** | 0 |

---

| Endianness Attribute | Value |
|:--|:--|
| **Multi-Word Endianness** | `little` |
| **Intra-Word Endianness** | `big` |

---

| String Handling Attribute | Value |
|:--|:--|
| **C-String Terminator** | 0 |
| **Embedded Strings Allowed** | Enabled |
| **String Byte Packing** | Disabled |

### Register Set

| Symbol | Title | Bit Size |
| :-- | :-- | --: |
| `a` | Accumulator | 8 |
| `b` | Temporary | 8 |
| `bank` | Memory Bank | 8 |
| `sp` | Stack Pointer | 8 |

### Compatibility

**Minimum BespokeASM Version:** 0.7.0

## Operand Sets

### `absolute_address` - Absolute Address

*Category: addressing*

| Operand | Syntax | Value | Addressing Mode | Description |
| :-- | :-- | :-- | :-- | :-- |
| Absolute Address (`address`) | `address` | numeric expression expressed as 16 bit value | `absolute_address` | Two byte absolute address |

### `immediate_16bit` - Immediate 16-bit

*Category: immediate*

| Operand | Syntax | Value | Addressing Mode |
| :-- | :-- | :-- | :-- |
| Immediate 16-bit (`int16`) | `int16` | numeric expression expressed as 16 bit value | `immediate` |

### `immediate_32bit` - Immediate 32-bit

*Category: immediate*

| Operand | Syntax | Value | Addressing Mode |
| :-- | :-- | :-- | :-- |
| Immediate 32-bit (`int32`) | `int32` | numeric expression expressed as 32 bit value | `immediate` |

### `immediate_8bit` - Immediate 8-bit

*Category: immediate*

| Operand | Syntax | Value | Addressing Mode |
| :-- | :-- | :-- | :-- |
| Immediate 8-bit (`int8`) | `int8` | numeric expression expressed as 8 bit value | `immediate` |

### `address_lsb`

| Operand | Syntax | Value | Addressing Mode | Description | Details |
| :-- | :-- | :-- | :-- | :-- | :-- |
| Page Offset Address (`address_lsb`) | `address_lsb` | numeric expression expressed as 8 bit value | Address | One byte used as the LSB for the address. The MSB is the current instruct's address MSB. | Uses only the least significant bits of the address value.<br><br>High address bits are taken from the current instruction address. |

### `offset`

| Operand | Syntax | Value | Addressing Mode | Description |
| :-- | :-- | :-- | :-- | :-- |
| Offset (`offset`) | `offset` | numeric expression expressed as 8 bit value | Immediate | one byte is interpreted as an offset |

### `relative_address`

| Operand | Syntax | Value | Addressing Mode | Description |
| :-- | :-- | :-- | :-- | :-- |
| Relative Address (`address`) | `address` | numeric expression expressed as 16 bit value | Address | Two byte address pointing two two bytes in memory that contain another address to use. |

### `relative_zero_page`

| Operand | Syntax | Value | Addressing Mode | Description | Details |
| :-- | :-- | :-- | :-- | :-- | :-- |
| Zero Page Address (`zp_addr`) | `zp_addr` | numeric expression expressed as 8 bit value | Address | one byte is interpreted as a zero page address to a word that in turn is used as a relative address. | Valid within `ZERO_PAGE` memory zone.<br><br>Uses only the least significant bits of the address value. |

### `zero_page`

| Operand | Syntax | Value | Addressing Mode | Description | Details |
| :-- | :-- | :-- | :-- | :-- | :-- |
| Zero Page Address (`zp_addr`) | `zp_addr` | numeric expression expressed as 8 bit value | Address | one byte is interpreted as a zero page address | Valid within `ZERO_PAGE` memory zone.<br><br>Uses only the least significant bits of the address value. |

# Instructions

## Arithmetic

### `ABB` : Add abs byte to byte: *adr2 = *adr2 + *adr1

*Calling syntax:*

```asm
ABB address1, address2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address1` | address | numeric expression expressed as 16 bit value |
| `address2` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `ABQ` : Add abs byte to zero-page long: *Q = *Q + *addr

*Calling syntax:*

```asm
ABQ address, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `ABV` : Add abs byte to zero-page word: *V = *V + *addr

*Calling syntax:*

```asm
ABV address, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `ABW` : Add byte at ad1 to word at ad2: *ad2 = *ad2 + *ad1

*Calling syntax:*

```asm
ABW address1, address2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address1` | address | numeric expression expressed as 16 bit value |
| `address2` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `ABZ` : Add abs byte to zero-page byte: *Z = *Z + *addr

*Calling syntax:*

```asm
ABZ address, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AC.B` : Add A with C to byte at addr: *addr = *addr + A + C

*Calling syntax:*

```asm
AC.B address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AC.Z` : Add A with C to zero-page byte: *Z = *Z + A + C

*Calling syntax:*

```asm
AC.Z zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `ACB` : Add byte at addr with C to A: A = A + *addr + C

*Calling syntax:*

```asm
ACB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `ACI` : Add immediate value to A with C: A = A + imm + C

*Calling syntax:*

```asm
ACI int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `ACV` : Add A with C to zero-page word

*Calling syntax:*

```asm
ACV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `ACW` : Add A with C to word at ads addr: *adr = *adr + A + C

*Calling syntax:*

```asm
ACW address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `ACZ` : Add zero-page byte with C to A: A = A + *Z + C

*Calling syntax:*

```asm
ACZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AD.B` : Add A to byte at abs addr: *addr = *addr + A

*Calling syntax:*

```asm
AD.B address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AD.R` : Add A to rel address: *(*addr) = *(*addr) + A

*Calling syntax:*

```asm
AD.R address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AD.T` : Add A to rel zero-page address: *(*Z) = *(*Z) + A

*Calling syntax:*

```asm
AD.T zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AD.Z` : Add A to zero-page byte: *Z = *Z + A

*Calling syntax:*

```asm
AD.Z zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `ADB` : Add byte at addr to A: A = A + *addr

*Calling syntax:*

```asm
ADB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `ADI` : Add immediate to A: A = A + imm

*Calling syntax:*

```asm
ADI int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `ADL` : Add A to long: *addr = *addr + A

*Calling syntax:*

```asm
ADL address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `ADQ` : Add A to zero-page long: *Q = *Q + A

*Calling syntax:*

```asm
ADQ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `ADR` : Add byte at rel addr to A: A = A + *(*addr)

*Calling syntax:*

```asm
ADR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `ADT` : Add byte at rel zero-page addr to A: A = A + *(*Z)

*Calling syntax:*

```asm
ADT zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `ADV` : Add A to zero-page word: *V = *V + A

*Calling syntax:*

```asm
ADV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `ADW` : Add A to word at abs addr: *addr = *addr + A

*Calling syntax:*

```asm
ADW address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `ADZ` : Add zero-page byte to A: A = A + *Z

*Calling syntax:*

```asm
ADZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AIB` : Add imm to byte at abs addr: *addr = *addr + imm

*Calling syntax:*

```asm
AIB int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AIL` : Add immediate byte to long: *L = *L + imm

*Calling syntax:*

```asm
AIL int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `AIQ` : Add immediate byte to zero-page long: *Q = *Q + imm

*Calling syntax:*

```asm
AIQ int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `AIR` : Add imm to byte at rel addr: *(*adr) = *(*adr) + imm

*Calling syntax:*

```asm
AIR int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AIT` : Add imm to byte at rel Z addr: *(*Z) = *(*Z) + imm

*Calling syntax:*

```asm
AIT int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AIV` : Add immediate byte to zero-page word: *V = *V + imm

*Calling syntax:*

```asm
AIV int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AIW` : Add imm byte to abs word: *addr = *addr + imm

*Calling syntax:*

```asm
AIW int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `AIZ` : Add immediate to zero-page byte: *Z = *Z + imm

*Calling syntax:*

```asm
AIZ int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AN.B` : Bitwise AND: *addr = *addr & A

*Calling syntax:*

```asm
AN.B address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `AN.Z` : Bitwise AND: *Z = *Z & A

*Calling syntax:*

```asm
AN.Z zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `ANB` : Bitwise AND: A = A & *addr

*Calling syntax:*

```asm
ANB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `ANI` : Bitwise AND: A = A & imm

*Calling syntax:*

```asm
ANI int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `ANR` : Bitwise AND: A = A & *(*addr)

*Calling syntax:*

```asm
ANR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `ANT` : Bitwise AND: A = A & *(*Z)

*Calling syntax:*

```asm
ANT zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `ANZ` : Bitwise AND: A = A & *Z

*Calling syntax:*

```asm
ANZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `AVV` : Add zero-page word to zero-page word: *V2 = *V2 + *V1

*Calling syntax:*

```asm
AVV zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AZB` : Add zero-page to byte at abs addr: *addr = *addr + *Z

*Calling syntax:*

```asm
AZB zp_addr, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AZL` : Add zero-page byte to long: *L = *L + *Z

*Calling syntax:*

```asm
AZL zp_addr, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `AZQ` : Add zero-page byte to zero-page long: *Q = *Q + *Z

*Calling syntax:*

```asm
AZQ zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `AZV` : Add zero-page byte to zero-page word: *V = *V + *Z

*Calling syntax:*

```asm
AZV zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `AZW` : Add zero-page byte to word at addr: *addr = *addr + *Z

*Calling syntax:*

```asm
AZW zp_addr, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `AZZ` : Add zero-page byte to zero-page byte: *Z2 = *Z2 + *Z1

*Calling syntax:*

```asm
AZZ zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CLC` : Clear carry flag (C=0)

*Calling syntax:*

```asm
CLC
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | a | Undefined |
| Flag | C | Set to 0 |
| Flag | N | Set to 1 |
| Flag | Z | Set to 0 |

---

### `DEB` : Decrement byte: *addr = *addr - 1

*Calling syntax:*

```asm
DEB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `DEC` : Decrement A: A = A - 1

*Calling syntax:*

```asm
DEC
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `DEL` : Decrement long: *addr = *addr - 0x00000001

*Calling syntax:*

```asm
DEL address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `DEQ` : Decrement zero-page long: *Q = *Q - 0x00000001

*Calling syntax:*

```asm
DEQ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `DEV` : Decrement zero-page word: *V = *V - 0x0001

*Calling syntax:*

```asm
DEV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `DEW` : Decrement word at abs addr: *addr = *addr - 0x0001

*Calling syntax:*

```asm
DEW address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `DEZ` : Decrement *Z = *Z - 1

*Calling syntax:*

```asm
DEZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `INB` : Increment byte at abs addr: *addr = *addr + 1

*Calling syntax:*

```asm
INB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `INC` : Increment A: A = A + 1

*Calling syntax:*

```asm
INC
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `INL` : Increment long: *addr = *addr + 0x00000001

*Calling syntax:*

```asm
INL address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `INQ` : Increment zero-page long: *Q = *Q + 0x00000001

*Calling syntax:*

```asm
INQ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `INV` : Increment zero-page word: *V = *V + 0x0001

*Calling syntax:*

```asm
INV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `INW` : Increment word at abs addr: *addr = *addr + 0x0001

*Calling syntax:*

```asm
INW address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `INZ` : Increment zero-page byte: *Z = *Z + 1

*Calling syntax:*

```asm
INZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LAB` : Load A A-indexed from addr: A = *(addr + A)

*Calling syntax:*

```asm
LAB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects LSB of result |
| Flag | C | Reflects LSB of result |
| Flag | Z | Reflects LSB of result |

---

### `LAP` : Load A A-indexed from page: A = *(pg<<8 + A)

*Calling syntax:*

```asm
LAP int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `LL0` : Logical left-shift A 0 steps (C=0)

*Calling syntax:*

```asm
LL0
```

---

### `LL1` : Logical left-shift A 1 step (C=0)

*Calling syntax:*

```asm
LL1
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LL2` : Logical left-shift A 2 steps (C=0)

*Calling syntax:*

```asm
LL2
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LL3` : Logical left-shift A 3 steps (C=0)

*Calling syntax:*

```asm
LL3
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LL4` : Logical left-shift A 4 steps (C=0)

*Calling syntax:*

```asm
LL4
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LL5` : Logical left-shift A 5 steps (C=0)

*Calling syntax:*

```asm
LL5
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LL6` : Logical left-shift A 6 steps (C=0)

*Calling syntax:*

```asm
LL6
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LL7` : Logical left-shift A 7 steps (C=0)

*Calling syntax:*

```asm
LL7
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LLB` : Logical shift byte left 1 step (C=0)

*Calling syntax:*

```asm
LLB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LLL` : Logical shift long left 1 step (C=0)

*Calling syntax:*

```asm
LLL address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `LLQ` : Logical shift fast long left 1 step (C=0)

*Calling syntax:*

```asm
LLQ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `LLV` : Logical shift fast word left 1 step (C=0)

*Calling syntax:*

```asm
LLV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `LLW` : Logical shift word left 1 step (C=0)

*Calling syntax:*

```asm
LLW address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `LLZ` : Logical shift left *Z 1 step (C=0)

*Calling syntax:*

```asm
LLZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LR0` : Logical right-shift A 0 steps (C=0) (= RR0, RL9)

*Calling syntax:*

```asm
LR0
```

---

### `LR1` : Logical right-shift A 1 step (C=0)

*Calling syntax:*

```asm
LR1
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LR2` : Logical right-shift A 2 steps (C=0) §

*Calling syntax:*

```asm
LR2
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LR3` : Logical right-shift A 3 steps (C=0) §

*Calling syntax:*

```asm
LR3
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LR4` : Logical right-shift A 4 steps (C=0) §

*Calling syntax:*

```asm
LR4
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LR5` : Logical right-shift A 5 steps (C=0) §

*Calling syntax:*

```asm
LR5
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LR6` : Logical right-shift A 6 steps (C=0) §

*Calling syntax:*

```asm
LR6
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LR7` : Logical right-shift A 7 steps (C=0)

*Calling syntax:*

```asm
LR7
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LRB` : Logical shift right abs byte 1 step (C=0)

*Calling syntax:*

```asm
LRB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LRZ` : Logical shift right zero-page byte 1 step (C=0)

*Calling syntax:*

```asm
LRZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `LZB` : Load A Z-indexed from addr: A = *(addr + *Z)

*Calling syntax:*

```asm
LZB zp_addr, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects LSB of result |
| Flag | C | Reflects LSB of result |
| Flag | Z | Reflects LSB of result |

---

### `LZP` : Load A Z-indexed from page: A = *(pg<<8 + *Z)

*Calling syntax:*

```asm
LZP zp_addr, int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `NEB` : Negate byte at abs address: *addr = -(*addr)

*Calling syntax:*

```asm
NEB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Copied from zero flag |
| Flag | Z | Reflects result |

---

### `NEG` : Negate A: A = -A

*Calling syntax:*

```asm
NEG
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Copied from zero flag |
| Flag | Z | Reflects result |

---

### `NEL` : Negate long (C = 1 only if long = 0)

*Calling syntax:*

```asm
NEL address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Copied from zero flag |
| Flag | Z | Reflects MSB of result |

---

### `NEQ` : Negate zero-page long (C = 1 only if long = 0)

*Calling syntax:*

```asm
NEQ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Copied from zero flag |
| Flag | Z | Reflects MSB of result |

---

### `NEV` : Negate zero-page word (C = 1 only if word=0)

*Calling syntax:*

```asm
NEV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Copied from zero flag |
| Flag | Z | Reflects MSB of result |

---

### `NEW` : Negate word at abs address (C = 1 only if word=0)

*Calling syntax:*

```asm
NEW address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Copied from zero flag |
| Flag | Z | Reflects MSB of result |

---

### `NEZ` : Negate zero-page byte: *Z = -(*Z)

*Calling syntax:*

```asm
NEZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Copied from zero flag |
| Flag | Z | Reflects result |

---

### `NOB` : Bitwise NOT byte: *addr = ~(*addr)

*Calling syntax:*

```asm
NOB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `NOL` : Bitwise NOT long at abs address

*Calling syntax:*

```asm
NOL address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |

---

### `NOQ` : Bitwise NOT zero-page long

*Calling syntax:*

```asm
NOQ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |

---

### `NOT` : Bitwise NOT A: A = ~A

*Calling syntax:*

```asm
NOT
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `NOV` : Bitwise NOT zero-page word: *V = ~(*V)

*Calling syntax:*

```asm
NOV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |

---

### `NOW` : Bitwise NOT word at abs address

*Calling syntax:*

```asm
NOW address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |

---

### `NOZ` : Bitwise NOT *Z: *Z = ~(*Z)

*Calling syntax:*

```asm
NOZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `OR.B` : Bitwise OR: *addr = *addr | A

*Calling syntax:*

```asm
OR.B address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `OR.Z` : Bitwise OR: *Z = *Z | A

*Calling syntax:*

```asm
OR.Z zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `ORB` : Bitwise OR: A = A | *addr

*Calling syntax:*

```asm
ORB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `ORI` : Bitwise OR: A = A | imm

*Calling syntax:*

```asm
ORI int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `ORR` : Bitwise OR: A = A | *(*addr)

*Calling syntax:*

```asm
ORR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `ORT` : Bitwise OR: A = A | *(*Z)

*Calling syntax:*

```asm
ORT zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `ORZ` : Bitwise OR: A = A | *Z

*Calling syntax:*

```asm
ORZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `RL0` : Rotate left A 0 steps via C (= RR9)

*Calling syntax:*

```asm
RL0
```

---

### `RL1` : Rotate left A 1 step via C (= RR8)

*Calling syntax:*

```asm
RL1
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `RL2` : Rotate left A 2 steps via C (= RR7)

*Calling syntax:*

```asm
RL2
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `RL3` : Rotate left A 3 steps via C (= RR6)

*Calling syntax:*

```asm
RL3
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `RL4` : Rotate left A 4 steps via C (= RR5)

*Calling syntax:*

```asm
RL4
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `RL5` : Rotate left A 5 steps via C (= RR4)

*Calling syntax:*

```asm
RL5
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `RL6` : Rotate left A 6 steps via C (= RR3)

*Calling syntax:*

```asm
RL6
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `RL7` : Rotate left A 7 steps via C (= RR2)

*Calling syntax:*

```asm
RL7
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `RLB` : Rotate left byte at abs addr 1 step via C

*Calling syntax:*

```asm
RLB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `RLL` : Rotate left abs long 1 step via C

*Calling syntax:*

```asm
RLL address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `RLQ` : Rotate left zero-page long 1 step via C

*Calling syntax:*

```asm
RLQ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `RLV` : Rotate left zero-page word 1 step via C

*Calling syntax:*

```asm
RLV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `RLW` : Rotate left word at abs addr 1 step via C

*Calling syntax:*

```asm
RLW address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `RLZ` : Rotate left zero-page byte 1 step via C

*Calling syntax:*

```asm
RLZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `RR1` : Rotate right A 1 step via C (= RL8)

*Calling syntax:*

```asm
RR1
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `RRB` : Rotate right byte at abs addr 1 step via C

*Calling syntax:*

```asm
RRB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `RRZ` : Rotate right zero-page byte 1 step via C

*Calling syntax:*

```asm
RRZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SBB` : Sub abs byte from byte: *adr2 = *adr2 - *adr1

*Calling syntax:*

```asm
SBB address1, address2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address1` | address | numeric expression expressed as 16 bit value |
| `address2` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SBQ` : Sub abs byte from zero-page long: *Q = *Q - *addr

*Calling syntax:*

```asm
SBQ address, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `SBV` : Sub abs byte from zero-page word: *V = *V - *addr

*Calling syntax:*

```asm
SBV address, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SBW` : Sub abs byte from word at adr2: *adr2 = *adr2 - *adr1

*Calling syntax:*

```asm
SBW address1, address2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address1` | address | numeric expression expressed as 16 bit value |
| `address2` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `SBZ` : Sub abs byte from zero-page byte: *Z = *Z - *addr

*Calling syntax:*

```asm
SBZ address, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SC.B` : Sub A with C from byte: *addr = *addr - A - 1 + C

*Calling syntax:*

```asm
SC.B address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SC.Z` : Sub A with C from zero-page byte: *Z = *Z - A - 1 + C

*Calling syntax:*

```asm
SC.Z zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SCB` : Sub byte at addr from A with C: A = A - *addr - 1 + C

*Calling syntax:*

```asm
SCB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SCI` : Sub imm value from A with C: A = A - imm - 1 + C

*Calling syntax:*

```asm
SCI int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SCV` : Sub A with C from zero-page word

*Calling syntax:*

```asm
SCV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `SCW` : Sub A with C from word at abs addr

*Calling syntax:*

```asm
SCW address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `SCZ` : Sub zero-page byte with C from A: A = A - *Z - 1 + C

*Calling syntax:*

```asm
SCZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SEC` : Set carry flag (C=1)

*Calling syntax:*

```asm
SEC
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | a | Undefined |
| Flag | C | Set to 1 |
| Flag | N | Set to 0 |
| Flag | Z | Set to 0 |

---

### `SIB` : Sub imm from byte: *addr = *addr - imm

*Calling syntax:*

```asm
SIB int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SIL` : Sub imm byte from long at abs address: *L = *L - imm

*Calling syntax:*

```asm
SIL int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `SIQ` : Sub imm byte from zero-page long Z: *Z = *Z - imm

*Calling syntax:*

```asm
SIQ int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `SIR` : Sub imm from byte at rel addr: *(*ad) = *(*ad) + imm

*Calling syntax:*

```asm
SIR int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SIT` : Sub imm from byte at rel Z addr: *(*Z) = *(*Z) + imm

*Calling syntax:*

```asm
SIT int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SIV` : Sub imm byte from zero-page word: *V = *V - imm

*Calling syntax:*

```asm
SIV int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SIW` : Sub imm byte from abs word: *addr = *addr - imm

*Calling syntax:*

```asm
SIW int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `SIZ` : Sub immediate from zero-page byte: *Z = *Z - imm

*Calling syntax:*

```asm
SIZ int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `STT` : Store A at rel address in zero page: *(*Z) = A

*Calling syntax:*

```asm
STT zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `STZ` : Store A to Z: *Z = A

*Calling syntax:*

```asm
STZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SU.B` : Sub A from byte at abs addr: *addr = *addr - A

*Calling syntax:*

```asm
SU.B address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SU.R` : Sub A from byte at rel addr: *(*addr) = *(*addr) - A

*Calling syntax:*

```asm
SU.R address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SU.T` : Sub A from rel zero-page address: *(*Z) = *(*Z) - A

*Calling syntax:*

```asm
SU.T zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SU.Z` : Sub A from zero-page byte: *Z = *Z - A

*Calling syntax:*

```asm
SU.Z zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SUB` : Sub byte at addr from A: A = A - *addr

*Calling syntax:*

```asm
SUB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SUI` : Sub immediate from A: A = A - imm

*Calling syntax:*

```asm
SUI int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SUL` : Sub A from long: *addr = *addr - A

*Calling syntax:*

```asm
SUL address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `SUQ` : Sub A from zero-page long: *Q = *Q - A

*Calling syntax:*

```asm
SUQ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `SUR` : Sub byte at rel addr from A: A = A - *(*addr)

*Calling syntax:*

```asm
SUR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SUT` : Sub byte at rel zero-page addr from A: A = A - *(*Z)

*Calling syntax:*

```asm
SUT zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SUV` : Sub A from zero-page word: *v = *V - A

*Calling syntax:*

```asm
SUV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `SUW` : Sub A from word at abs addr: *addr = *addr - A

*Calling syntax:*

```asm
SUW address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `SUZ` : Sub zero-page byte from A: A = A - *Z

*Calling syntax:*

```asm
SUZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SVV` : Sub zero-page word from word: *V2 = *V2 - *V1

*Calling syntax:*

```asm
SVV zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SZB` : Sub zero-page byte from abs byte: *addr = *addr - *Z

*Calling syntax:*

```asm
SZB zp_addr, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SZL` : Sub zero-page byte from long: *L = *L - *Z

*Calling syntax:*

```asm
SZL zp_addr, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `SZP` : Store A Z-indexed to page: *(pg<<8 + *Z) = A

*Calling syntax:*

```asm
SZP zp_addr, int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `int8` | numeric | numeric expression expressed as 8 bit value |

---

### `SZQ` : Sub zero-page byte from zero-page long: *Q = *Q - *Z

*Calling syntax:*

```asm
SZQ zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `SZV` : Sub zero-page byte from zero-page word: *V = *V - *Z

*Calling syntax:*

```asm
SZV zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `SZW` : Sub zero-page byte from abs word: *addr = *addr - *Z

*Calling syntax:*

```asm
SZW zp_addr, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |
| Flag | N | Reflects MSB of result |
| Flag | C | Reflects MSB of result |
| Flag | Z | Reflects MSB of result |

---

### `SZZ` : Sub zero-page byte from Z byte: *Z2 = *Z2 - *Z1

*Calling syntax:*

```asm
SZZ zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `XR.B` : Bitwise XOR: *addr = *addr ^ A

*Calling syntax:*

```asm
XR.B address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `XR.Z` : Bitwise XOR: *Z = *Z ^ A

*Calling syntax:*

```asm
XR.Z zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `XRB` : Bitwise XOR: A = A ^ * addr §

*Calling syntax:*

```asm
XRB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `XRI` : Bitwise XOR: A = A ^ imm §

*Calling syntax:*

```asm
XRI int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `XRR` : Bitwise XOR: A = A ^ *(*addr) §

*Calling syntax:*

```asm
XRR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `XRT` : Bitwise XOR: A = A ^ *(*Z)

*Calling syntax:*

```asm
XRT zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `XRZ` : Bitwise XOR: A = A ^ *Z

*Calling syntax:*

```asm
XRZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

## Flow Control

### `BCC` : Branch on carry clear

*Calling syntax:*

```asm
BCC address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `BCS` : Branch on carry set

*Calling syntax:*

```asm
BCS address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `BEQ` : Branch on zero

*Calling syntax:*

```asm
BEQ address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `BGT` : Branch on greater

*Calling syntax:*

```asm
BGT address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `BLE` : Branch on less or equal

*Calling syntax:*

```asm
BLE address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `BMI` : Branch on minus

*Calling syntax:*

```asm
BMI address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `BNE` : Branch on non-zero

*Calling syntax:*

```asm
BNE address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `BPL` : Branch on plus

*Calling syntax:*

```asm
BPL address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `FCC` : Fast branch on carry clear

*Calling syntax:*

```asm
FCC address_lsb
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address_lsb` | address | numeric expression expressed as 8 bit value |

---

### `FCS` : Fast branch on carry set

*Calling syntax:*

```asm
FCS address_lsb
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address_lsb` | address | numeric expression expressed as 8 bit value |

---

### `FEQ` : Fast branch on zero

*Calling syntax:*

```asm
FEQ address_lsb
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address_lsb` | address | numeric expression expressed as 8 bit value |

---

### `FGT` : Fast branch on greater

*Calling syntax:*

```asm
FGT address_lsb
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address_lsb` | address | numeric expression expressed as 8 bit value |

---

### `FLE` : Fast branch on less or equal

*Calling syntax:*

```asm
FLE address_lsb
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address_lsb` | address | numeric expression expressed as 8 bit value |

---

### `FMI` : Fast branch on minus

*Calling syntax:*

```asm
FMI address_lsb
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address_lsb` | address | numeric expression expressed as 8 bit value |

---

### `FNE` : Fast branch on non-zero

*Calling syntax:*

```asm
FNE address_lsb
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address_lsb` | address | numeric expression expressed as 8 bit value |

---

### `FPA` : Fast jump to lsb addr

*Calling syntax:*

```asm
FPA address_lsb
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address_lsb` | address | numeric expression expressed as 8 bit value |

---

### `FPL` : Fast branch on plus

*Calling syntax:*

```asm
FPL address_lsb
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address_lsb` | address | numeric expression expressed as 8 bit value |

---

### `JAR` : Jump A-indexed to rel address: PC = *(addr + A)

*Calling syntax:*

```asm
JAR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Flag | N | Reflects LSB of result |
| Flag | C | Reflects LSB of result |
| Flag | Z | Reflects LSB of result |

---

### `JAS` : Jump to subroutine conserving A §

*Calling syntax:*

```asm
JAS address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `JPA` : Jump to abs address: PC = addr

*Calling syntax:*

```asm
JPA address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `JPR` : Jump to rel address: PC = *addr

*Calling syntax:*

```asm
JPR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `JPS` : Jump to subroutine

*Calling syntax:*

```asm
JPS address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Undefined |

---

### `NOP` : No operation

*Calling syntax:*

```asm
NOP
```

---

### `RTS` : Return from subroutine

*Calling syntax:*

```asm
RTS
```

## Input/Output

### `INK` : Read PS/2 input to A: A = PS2

*Calling syntax:*

```asm
INK
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `INT` : Read UART input to A: A = UART

*Calling syntax:*

```asm
INT
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

---

### `OUT` : Output A to UART: UART = A

*Calling syntax:*

```asm
OUT
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Flag | N | Set to 1 |
| Flag | C | Set to 0 |
| Flag | Z | Set to 0 |

---

### `RAP` : Read A-indexed FLASH data: A = *(pg<<8 + A)

*Calling syntax:*

```asm
RAP int81, int82
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int81` | numeric | numeric expression expressed as 8 bit value |
| `int82` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `RDB` : Read FLASH data from abs 3-byte address

*Calling syntax:*

```asm
RDB int16, int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int16` | numeric | numeric expression expressed as 16 bit value |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `RDR` : Read FLASH data from rel 3-byte address

*Calling syntax:*

```asm
RDR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `RZP` : Read Z-indexed FLASH data: A = *(pg<<8 + *Z)

*Calling syntax:*

```asm
RZP zp_addr, int81, int82
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `int81` | numeric | numeric expression expressed as 8 bit value |
| `int82` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `WDB` : Write FLASH data to abs 3-byte address §

*Calling syntax:*

```asm
WDB int16, int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int16` | numeric | numeric expression expressed as 16 bit value |
| `int8` | numeric | numeric expression expressed as 8 bit value |

---

### `WDR` : Write FLASH data to rel 3-byte address §

*Calling syntax:*

```asm
WDR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `WIN` : Wait for input

*Calling syntax:*

```asm
WIN
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Flag | N | Undefined |
| Flag | C | Undefined |
| Flag | Z | Undefined |

## Memory Movement

### `CLB` : Clear byte at addr: *addr = 0x00

*Calling syntax:*

```asm
CLB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `CLL` : Clear long: *addr = 0x00000000

*Calling syntax:*

```asm
CLL address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Set to 0 |

---

### `CLQ` : Clear zero-page long: *Q = 0x00000000

*Calling syntax:*

```asm
CLQ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Set to 0 |

---

### `CLV` : Clear fast word: *V = 0x0000

*Calling syntax:*

```asm
CLV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Set to 0 |

---

### `CLW` : Clear word at addr: *addr = 0x0000

*Calling syntax:*

```asm
CLW address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Set to 0 |

---

### `CLZ` : Clear Z: *Z = 0x00

*Calling syntax:*

```asm
CLZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

---

### `LDB` : Load A from abs address: A = *addr

*Calling syntax:*

```asm
LDB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `LDI` : Load A immediate: A = imm

*Calling syntax:*

```asm
LDI int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `LDR` : Load A from relative address: A = *(*addr)

*Calling syntax:*

```asm
LDR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `LDT` : Load A from rel address in zero page: A = *(*Z)

*Calling syntax:*

```asm
LDT zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `LDZ` : Load A from Z: A = *Z

*Calling syntax:*

```asm
LDZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `MBB` : Move byte from abs adr1 to abs adr2: *adr2 = *adr1

*Calling syntax:*

```asm
MBB address1, address2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address1` | address | numeric expression expressed as 16 bit value |
| `address2` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `MBZ` : Move byte at abs addr to zero-page: *Z = *addr

*Calling syntax:*

```asm
MBZ address, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `MIB` : Move imm byte to abs addr: *addr = imm

*Calling syntax:*

```asm
MIB int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `MIR` : Move imm byte to rel addr: *(*addr) = imm

*Calling syntax:*

```asm
MIR int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `MIT` : Move imm byte to rel zero-page addr: *(*T) = imm

*Calling syntax:*

```asm
MIT int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `MIV` : Move imm word to zero-page word: *V = imm

*Calling syntax:*

```asm
MIV int16, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int16` | numeric | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |

---

### `MIW` : Move imm word to abs addr: *addr = imm

*Calling syntax:*

```asm
MIW int16, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int16` | numeric | numeric expression expressed as 16 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |

---

### `MIZ` : Move imm byte to zero-page: *Z = imm

*Calling syntax:*

```asm
MIZ int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `MVV` : Move zero-page word to zero-page word: *V2 = *V1

*Calling syntax:*

```asm
MVV zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |

---

### `MWV` : Move word at abs addr to fast word: *V = *addr

*Calling syntax:*

```asm
MWV address, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated (MSB of result) |

---

### `MZB` : Move zero-page byte to abs addr: *addr = *Z

*Calling syntax:*

```asm
MZB zp_addr, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `MZZ` : Move zero-page byte to zero-page byte: *Z2 = *Z1

*Calling syntax:*

```asm
MZZ zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `STB` : Store A to address: *addr = A

*Calling syntax:*

```asm
STB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `STR` : Store A at relative address: *(*addr) = A

*Calling syntax:*

```asm
STR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

## Stack Management

### `LDS` : Load from stack: A = *(0xff00 + SP + off)

*Calling syntax:*

```asm
LDS int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `PHS` : Push A onto stack

*Calling syntax:*

```asm
PHS
```

---

### `PLS` : Pull A from stack

*Calling syntax:*

```asm
PLS
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |

---

### `STS` : Store on stack: *(0xff00 + SP + off) = A

*Calling syntax:*

```asm
STS int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

## Comparison

### `CBB` : Compare abs byte to byte: A = *adr2 - *adr1

*Calling syntax:*

```asm
CBB address1, address2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address1` | address | numeric expression expressed as 16 bit value |
| `address2` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CBZ` : Compare abs byte to zero-page byte: A = *addr - *Z

*Calling syntax:*

```asm
CBZ address, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CIB` : Compare imm to byte at abs addr: A = *addr - imm

*Calling syntax:*

```asm
CIB int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CIR` : Compare imm to byte at rel addr: A = *(*addr) - imm

*Calling syntax:*

```asm
CIR int8, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CIT` : Compare imm to byte at rel Z: A = *(*Z) - imm

*Calling syntax:*

```asm
CIT int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CIZ` : Compare imm to zero-page byte: A = *Z - imm

*Calling syntax:*

```asm
CIZ int8, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CPB` : Compare byte at addr to A: eval A - *addr

*Calling syntax:*

```asm
CPB address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CPI` : Compare immediate value to A: eval A - imm

*Calling syntax:*

```asm
CPI int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CPR` : Compare rel byte to A: eval A - *(*addr)

*Calling syntax:*

```asm
CPR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CPT` : Compare byte at rel Z to A: eval A - *(*Z)

*Calling syntax:*

```asm
CPT zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CPZ` : Compare zero-page byte to A: eval A - *Z

*Calling syntax:*

```asm
CPZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CZB` : Compare zero-page byte to byte: A = *addr - *Z

*Calling syntax:*

```asm
CZB zp_addr, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

---

### `CZZ` : Compare zero-page byte to byte: A = *Z2 - *Z1

*Calling syntax:*

```asm
CZZ zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | A | Updated |
| Flag | N | Reflects result |
| Flag | C | Reflects result |
| Flag | Z | Reflects result |

# Macros

## Arithmetic

### `AQQ` : add two zero-page longs. result in second zero page operand

*Calling syntax:*

```asm
AQQ zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

---

### `PLS4` : pls4

*Calling syntax:*

```asm
PLS4
```

---

### `SQQ` : subtract two zero-page longs. *Q2 = *Q2 - Q1

*Calling syntax:*

```asm
SQQ zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

## Memory Movement

### `M2IV` : move 2 byte word from immediate to zero-page word

*Calling syntax:*

```asm
M2IV int16, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int16` | numeric | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

---

### `M4IQ` : move 4 bytes long from immediate to zero-page long

*Calling syntax:*

```asm
M4IQ int32, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int32` | numeric | numeric expression expressed as 32 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

---

### `MLL` : move long from abs address to abs address

*Calling syntax:*

```asm
MLL address1, address2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address1` | address | numeric expression expressed as 16 bit value |
| `address2` | address | numeric expression expressed as 16 bit value |

---

### `MLQ` : move long from abs address to zero-page long

*Calling syntax:*

```asm
MLQ address, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

---

### `MQQ` : move zero-page long to zero-page long

*Calling syntax:*

```asm
MQQ zp_addr1, zp_addr2
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr1` | address | numeric expression expressed as 8 bit value |
| `zp_addr2` | address | numeric expression expressed as 8 bit value |

## Stack Management

### `MLS4` : move a long (4 bytes) from abs address to stack starting at passed offset

*Calling syntax:*

```asm
MLS4 address, offset
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `offset` | numeric | numeric expression expressed as 8 bit value |

---

### `MQS4` : move 4 bytes from zero-page long to stack starting at passed offset

*Calling syntax:*

```asm
MQS4 zp_addr, offset
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `offset` | numeric | numeric expression expressed as 8 bit value |

---

### `MS2V` : move 2 bytes from stack to zero-page word

*Calling syntax:*

```asm
MS2V offset, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `offset` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

---

### `MS2W` : move 2-byte word from stack at offset to absolute address

*Calling syntax:*

```asm
MS2W offset, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `offset` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `MS4L` : move a long (4 bytes) from stack starting at passed offset to abs address

*Calling syntax:*

```asm
MS4L offset, address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `offset` | numeric | numeric expression expressed as 8 bit value |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `MS4Q` : move 4 bytes from stack starting at passed offset to zero-page long

*Calling syntax:*

```asm
MS4Q offset, zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `offset` | numeric | numeric expression expressed as 8 bit value |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

---

### `MVS2` : move 4 bytes from zero-page word to stack starting at passed offset

*Calling syntax:*

```asm
MVS2 zp_addr, offset
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |
| `offset` | numeric | numeric expression expressed as 8 bit value |

---

### `MWS2` : move 2-byte word at absolute address to stack at offset

*Calling syntax:*

```asm
MWS2 address, offset
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |
| `offset` | numeric | numeric expression expressed as 8 bit value |

---

### `PHS2I` : Push Immediate Word onto Stack

*Calling syntax:*

```asm
PHS2I int16
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int16` | numeric | numeric expression expressed as 16 bit value |

---

### `PHS2S` : Push Word at Stack Offset onto Stack

Pushes a 2 byte value currently at given stack offset onto the stack (again).

*Calling syntax:*

```asm
PHS2S offset
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `offset` | numeric | numeric expression expressed as 8 bit value |

---

### `PHS4A` : Push Word in Memory onto Stack

push 4 bytes at an absolute address onto the stack

*Calling syntax:*

```asm
PHS4A address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `PHS4I` : Push Immediate Long onto Stack

*Calling syntax:*

```asm
PHS4I uint32
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `uint32` | numeric | numeric expression expressed as 32 bit value |

---

### `PHS4S` : Push Long at Stack Offset onto Stack

Pushes a 4 byte value currently at given stack offset onto the stack (again).

*Calling syntax:*

```asm
PHS4S offset
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `offset` | numeric | numeric expression expressed as 8 bit value |

---

### `PHSI` : Push Immediate Byte onto Stack

*Calling syntax:*

```asm
PHSI int8
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `int8` | numeric | numeric expression expressed as 8 bit value |

---

### `PHSPTR` : Push Immediate Memory Address onto Stack

push an immediate absolute address to the stack per MinOS calling convention

*Calling syntax:*

```asm
PHSPTR address
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `address` | address | numeric expression expressed as 16 bit value |

---

### `PHSQ` : push a zero page long to the stack

*Calling syntax:*

```asm
PHSQ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

---

### `PHSV` : push a zero-page word to the stack

*Calling syntax:*

```asm
PHSV zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

---

### `PHSZ` : Push Zero-page Byte onto the Stack

*Calling syntax:*

```asm
PHSZ zp_addr
```

where

| Operand | Type | Value |
| :-- | :-- | :-- |
| `zp_addr` | address | numeric expression expressed as 8 bit value |

---

### `PLS2` : pull 2 bytes from the stack

*Calling syntax:*

```asm
PLS2
```

---

### `SPINIT` : Initialize Stack Pointer

*Calling syntax:*

```asm
SPINIT
```

#### Modifies

| Type | Target | Description |
| --- | --- | --- |
| Register | sp | Initialized |