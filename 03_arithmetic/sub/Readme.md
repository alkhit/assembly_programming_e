# SUB: Flags Analysis

For `sub`, the six arithmetic flags are all defined and follow the standard rules:

| Flag | Set when... |
|------|-------------|
| CF   | Unsigned borrow out of the MSB: minuend < subtrahend (viewed as unsigned). |
| ZF   | Result is zero. |
| SF   | MSB of result is 1. |
| OF   | Signed overflow: subtracting two operands of different signs produced a result whose sign is wrong. |
| AF   | Borrow out of bit 3 (low nibble of minuend < low nibble of subtrahend). |
| PF   | Low byte of result has an even number of 1-bits. |

`sub` sets every arithmetic flag, unlike `mul` (only CF/OF) and `div` (none).

## Program 1: `sub1.asm`

**Operation:** `sub al, [num2]` where `al = 50` (0x32) and `num2 = 80` (0x50)  
**Result:** `al = 50 - 80 = -30` → stored as `0xE2` (binary `11100010`). Check: 0xE2 as unsigned is 226 = 256 - 30.

Flags before: all arithmetic flags cleared. After: `CF PF SF`.

| Flag | Status shown in GDB | Meaning |
|------|---------------------|---------|
| CF   | Set                 | Unsigned borrow. 50 < 80, so the subtraction needs to borrow from bit 8 (outside `al`). |
| ZF   | Cleared             | Result 0xE2 is not zero. |
| SF   | Set                 | Bit 7 of 0xE2 (`11100010`) is 1 → negative in signed interpretation (−30). |
| OF   | Cleared             | Signed −30 fits in the 8-bit signed range (−128..+127). No overflow. |
| AF   | Cleared             | Low nibble of 50 is 0x2, of 80 is 0x0. 2 − 0 = 2 → no borrow from bit 3. |
| PF   | Set                 | Low byte 0xE2 = `11100010` has four 1-bits → even parity. |

## Program 2: `sub2.asm`

**Operation:** `sub ax, [num2]` where `ax = 1000` (0x03E8) and `num2 = 2000` (0x07D0)  
**Result:** `ax = 1000 - 2000 = -1000` → stored as `0xFC18` (binary `1111110000011000`). Check: 0xFC18 as unsigned is 64536 = 65536 - 1000.

Flags before: all arithmetic flags cleared. After: `CF PF SF`.

| Flag | Status shown in GDB | Meaning |
|------|---------------------|---------|
| CF   | Set                 | Unsigned borrow. 1000 < 2000, so the subtraction needs to borrow from bit 16 (outside `ax`). |
| ZF   | Cleared             | Result 0xFC18 is not zero. |
| SF   | Set                 | Bit 15 of 0xFC18 (`1111110000011000`) is 1 → negative in signed interpretation (−1000). |
| OF   | Cleared             | Signed −1000 fits in the 16-bit signed range (−32768..+32767). No overflow. |
| AF   | Cleared             | Low nibble of 1000 is 0x8, of 2000 is 0x0. 8 − 0 = 8 → no borrow from bit 3. |
| PF   | Set                 | Low byte 0x18 = `00011000` has two 1-bits → even parity. |

**Takeaway:** `sub` sets all six arithmetic flags based on the subtraction result. In both programs, the minuend is smaller than the subtrahend, so the unsigned result wraps and CF = 1. The signed result is still in range, so OF = 0. SF follows the top bit of the stored result, ZF is clear because the result isn't zero, AF reflects the nibble borrow, and PF reflects the parity of the low byte.