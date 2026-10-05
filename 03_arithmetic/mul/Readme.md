# MUL: Flags Analysis

For `mul` (unsigned multiply), only **CF and OF are defined**. The rule is simple:

- If the high half of the product is zero, the full result fits in the low half → **CF = 0, OF = 0**.
- If the high half of the product is non-zero, the full result needs the wider destination → **CF = 1, OF = 1**.

All other flags (**SF, ZF, AF, PF**) are **undefined** after `mul`. Their values must not be interpreted.

Where the result goes depends on the operand size:

| Operand size        | Multiplicand | Source       | High half | Low half |
|---------------------|--------------|--------------|-----------|----------|
| 8-bit (`mul r/m8`)  | `al`         | `r/m8`       | `ah`      | `al`     |
| 16-bit (`mul r/m16`)| `ax`         | `r/m16`      | `dx`      | `ax`     |
| 32-bit (`mul r/m32`)| `eax`        | `r/m32`      | `edx`     | `eax`    |

CF and OF are always set to the same value by `mul` — they both indicate whether the high half is non-zero.

## Program 1: `mul1.asm`

**Operation:** `mul byte [num2]` where `al = 25` and `num2 = 10`  
**Result:** `ax = 250` (0x00FA), so `al = 0xFA` and `ah = 0x00`. Check: 25 × 10 = 250.

Flags before: all arithmetic flags cleared. After: only `CF` and `OF` are defined; both are cleared.

| Flag | Status shown in GDB | Meaning |
|------|---------------------|---------|
| CF   | Cleared             | Defined. High half `ah = 0x00`, so the product fits in `al`. |
| OF   | Cleared             | Defined. Same reason as CF. |
| ZF   | Cleared             | Undefined after `mul`. Do not interpret. |
| SF   | Cleared             | Undefined after `mul`. Do not interpret. |
| PF   | Cleared             | Undefined after `mul`. Do not interpret. |
| AF   | Cleared             | Undefined after `mul`. Do not interpret. |

## Program 2: `mul2.asm`

**Operation:** `mul word [num2]` where `ax = 3000` and `num2 = 200`  
**Result:** `dx:ax = 600000` (0x000927C0), so `ax = 0x27C0` and `dx = 0x0009`. Check: 3000 × 200 = 600000.

Flags before: all arithmetic flags cleared. After: `CF` and `OF` are set; other flags are undefined.

| Flag | Status shown in GDB | Meaning |
|------|---------------------|---------|
| CF   | Set                 | Defined. High half `dx = 0x0009` is non-zero, so the product does not fit in `ax` alone. |
| OF   | Set                 | Defined. Same reason as CF. |
| ZF   | Cleared             | Undefined after `mul`. Do not interpret. |
| SF   | Cleared             | Undefined after `mul`. Do not interpret. |
| PF   | Cleared             | Undefined after `mul`. Do not interpret. |
| AF   | Cleared             | Undefined after `mul`. Do not interpret. |

**Takeaway:** `mul` is different from `add` and `sub`. It only gives you two useful flags — CF and OF — and both carry the same information: whether the high half of the product is non-zero. Unlike `add`/`sub`, SF, ZF, AF, and PF are not meaningful after a multiply. Any values shown for them in GDB are leftovers and must not be attributed to the multiplication.