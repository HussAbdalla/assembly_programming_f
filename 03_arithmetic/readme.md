# 03_arithmetic — Effect of Arithmetic Instructions on the FLAGS Register

This report documents the expected effect of `add`, `sub`, `mul`, and `div`
(and their carry/borrow-chaining counterparts `adc`/`sbb`) on the x86 status
flags: **CF, ZF, SF, OF, PF, AF**. All values were reasoned by hand from the
operand bit patterns (not captured via debugger), based on the standard
x86 flag rules.

---

## Addition

### `add1.asm` — 8-bit addition, signed overflow case
**Operation:** `al = num1 + num2` → 120 + 10
**Result:** `1000 0010` (130 unsigned / −126 signed)

| Flag | Reasoning | Value |
|------|-----------|-------|
| CF | 120 + 10 = 130 fits within 0–255, no carry out of bit 7 | 0 |
| ZF | Result = 130, not zero | 0 |
| SF | MSB of `1000 0010` is 1 | 1 |
| OF | Two positive operands produced a negative (MSB=1) result — signed overflow | 1 |
| PF | `1000 0010` has two 1-bits (even) | 1 |
| AF | Low nibbles 8 + 10 = 18 > 15, carries into bit 4 | 1 |

**Note:** Result is valid as unsigned (130) but invalid as signed (overflow). CF=0/OF=1 shows these two flags judge different things on the same bit pattern.

---

### `add2.asm` — 16-bit addition, no boundary triggered
**Operation:** `ax = num1 + num2` → 32000 + 500
**Result:** `0111 1110 1111 0100` (32500)

| Flag | Reasoning | Value |
|------|-----------|-------|
| CF | 32500 fits within 0–65535, no carry out | 0 |
| ZF | Result = 32500, not zero | 0 |
| SF | MSB of `0x7EF4` is 0 | 0 |
| OF | Both operands positive, result positive — no contradiction | 0 |
| PF | Low byte `0xF4` has five 1-bits (odd) | 0 |
| AF | Low nibbles 0 + 4 = 4, no carry into bit 4 | 0 |

**Note:** All flags clear — contrasts with add1.asm: same instruction, but operands stay well within range for a 16-bit register.

---

### `add3.asm` / `adc4.asm` — 16-bit addition with carry chaining
**Operation 1:** `ax = 0xFFFF + 1`
**Result 1:** `0x0000` (wraparound)

| Flag | Reasoning | Value |
|------|-----------|-------|
| CF | 65535 + 1 = 65536 needs a 17th bit | 1 |
| ZF | Result wrapped to 0 | 1 |
| SF | MSB of 0x0000 is 0 | 0 |
| OF | Operands have opposite signs (−1 and +1) — signed overflow impossible | 0 |
| PF | 0x00 has zero set bits (even) | 1 |
| AF | Low nibbles 0xF + 0x1 = 0x10, carries into bit 4 | 1 |

**Operation 2 (`adc ax, 0`):** `ax = 0 + 0 + CF(1)`
**Result 2:** `0x0001`

| Flag | Reasoning | Value |
|------|-----------|-------|
| CF | 0+0+1 = 1, no carry out of bit 15 | 0 |
| ZF | Result = 1, not zero | 0 |
| SF | MSB of 0x0001 is 0 | 0 |
| OF | Destination and operand positive, result positive — no contradiction | 0 |
| PF | 0x01 has one set bit (odd) | 0 |
| AF | 0x0+0x0+carry-in(1) = 0x1, no carry into bit 4 | 0 |

**Note:** Classic unsigned wraparound (CF=1, ZF=1, OF=0 since operand signs differed). `adc` propagates the carry into a second addition — the basis of multi-word addition.

---

## Subtraction

### `sub1.asm` — 8-bit subtraction, small minus big
**Operation:** `al = num1 - num2` → 50 − 80
**Result:** `1110 0010` (226 unsigned / −30 signed)

| Flag | Reasoning | Value |
|------|-----------|-------|
| CF | 80 > 50, borrow required out of bit 7 | 1 |
| ZF | Result = 226, not zero | 0 |
| SF | MSB of `1110 0010` is 1 | 1 |
| OF | Both operands positive (same sign) — signed overflow impossible | 0 |
| PF | `1110 0010` has four 1-bits (even) | 1 |
| AF | Low nibbles 2 − 0 = 2, no borrow needed | 0 |

**Note:** CF=1 (invalid unsigned) but OF=0 (valid signed, −30 is in range). Mirror image of add1.asm's CF=0/OF=1 case.

---

### `sub2.asm` — 16-bit subtraction, small minus big
**Operation:** `ax = num1 - num2` → 1000 − 2000
**Result:** `1111 1100 0001 1000` (64536 unsigned / −1000 signed)

| Flag | Reasoning | Value |
|------|-----------|-------|
| CF | 2000 > 1000, borrow required out of bit 15 | 1 |
| ZF | Result = 64536, not zero | 0 |
| SF | MSB of `0xFC18` is 1 | 1 |
| OF | Both operands positive (same sign) — signed overflow impossible | 0 |
| PF | Low byte `0x18` has two 1-bits (even) | 1 |
| AF | Low nibbles 8 − 0 = 8, no borrow needed | 0 |

**Note:** Same CF/OF relationship as sub1.asm, confirmed consistent across register widths.

---

### `sub3.asm` (`sbb.asm`) — 16-bit subtraction with borrow chaining
**Operation 1:** `ax = 0 − 1`
**Result 1:** `0xFFFF` (65535 unsigned / −1 signed)

| Flag | Reasoning | Value |
|------|-----------|-------|
| CF | 1 > 0, borrow required out of bit 15 | 1 |
| ZF | Result = 65535, not zero | 0 |
| SF | MSB of 0xFFFF is 1 | 1 |
| OF | Both operands positive (same sign) — signed overflow impossible | 0 |
| PF | 0xFF has eight 1-bits (even) | 1 |
| AF | Low nibble 0 − 1 needs a borrow | 1 |

**Operation 2 (`sbb ax, 0`):** `ax = 0xFFFF − 0 − CF(1)`
**Result 2:** `0xFFFE`

| Flag | Reasoning | Value |
|------|-----------|-------|
| CF | Effective subtrahend = 0+1 = 1; 0xFFFF ≥ 1, no borrow out of bit 15 | 0 |
| ZF | Result = 65534, not zero | 0 |
| SF | MSB of 0xFFFE is 1 | 1 |
| OF | Minuend negative, result also negative — minuend's sign preserved, no contradiction | 0 |
| PF | 0xFE has seven 1-bits (odd) | 0 |
| AF | 0xF − (0x0 + carry-in 1) = 0xE, no borrow needed | 0 |

**Note:** Mirrors `adc` in reverse — `sbb` propagates a borrow into a second subtraction, the basis of multi-word subtraction.

---

## Multiplication

> **Rule:** `mul` only defines **CF** and **OF** — and they always move together. Both are set to 1 if the product spilled into the upper half of the destination register (AH/DX/EDX); both are 0 if the product fit entirely in the lower half. **ZF, SF, PF, AF are undefined** by `mul`.

### `mul1.asm` — 8-bit multiplication, no overflow
**Operation:** `ax = al * num2` → 25 × 10
**Result:** AX = `0x00FA` (250), AH = 0x00

| Flag | Reasoning | Value |
|------|-----------|-------|
| CF | AH = 0x00 — product fit entirely in AL | 0 |
| OF | Same rule as CF for MUL | 0 |
| ZF/SF/PF/AF | Not defined by MUL | Undefined |

---

### `mul2.asm` — 16-bit multiplication, overflow
**Operation:** `DX:AX = ax * num2` → 3000 × 200 = 600,000
**Result:** DX:AX = `0x0009:0x27C0`

| Flag | Reasoning | Value |
|------|-----------|-------|
| CF | DX = 0x0009 ≠ 0 — product spilled into upper half | 1 |
| OF | Same rule as CF for MUL | 1 |
| ZF/SF/PF/AF | Not defined by MUL | Undefined |

**Note:** Direct contrast with mul1.asm — confirms CF=OF collapse into a single "did it fit in the lower half?" question.

---

### `mul3.asm` — 32-bit multiplication, overflow
**Operation:** `EDX:EAX = eax * num2` → 100,000 × 300,000 = 30,000,000,000
**Result:** EDX:EAX = `0x00000006:0xFC23AC00`

| Flag | Reasoning | Value |
|------|-----------|-------|
| CF | EDX = 0x00000006 ≠ 0 — product spilled into upper half | 1 |
| OF | Same rule as CF for MUL | 1 |
| ZF/SF/PF/AF | Not defined by MUL | Undefined |

**Note:** Confirms the CF/OF rule holds consistently across byte, word, and dword operand widths.

---

## Division

> **Rule:** `div`/`idiv` **do not define any status flags** (CF, ZF, SF, OF, PF, AF are all left undefined/architecturally unreliable). Any flag values seen in a debugger after `div` are leftover artifacts from a prior instruction, not something `div` computed. The only guaranteed outputs are the quotient and remainder.

### `div1.asm` — 8-bit division
**Operation:** `al = ax / divisor`, `ah = ax % divisor` → 100 ÷ 7
**Result:** AL = 14 (quotient), AH = 2 (remainder)

| Flag | Value |
|------|-------|
| CF, ZF, SF, OF, PF, AF | Undefined — DIV does not set these |

---

### `div2.asm` — 16-bit division (DX:AX dividend)
**Operation:** `ax = (DX:AX) / divisor`, `dx = (DX:AX) % divisor` → 50000 ÷ 300
**Result:** AX = 166 (quotient), DX = 200 (remainder)

| Flag | Value |
|------|-------|
| CF, ZF, SF, OF, PF, AF | Undefined — DIV does not set these |

---

### `div3.asm` — 32-bit division (EDX:EAX dividend)
**Operation:** `eax = (EDX:EAX) / divisor`, `edx = (EDX:EAX) % divisor` → 300,000,000 ÷ 1000
**Result:** EAX = 300000 (quotient), EDX = 0 (remainder)

| Flag | Value |
|------|-------|
| CF, ZF, SF, OF, PF, AF | Undefined — DIV does not set these |

**Note:** Even though the remainder here is exactly 0, ZF does *not* reflect that — DIV simply never touches it. Branching on "divided evenly" requires an explicit `test edx, edx` or `cmp edx, 0` afterward.

**Edge case (not implemented, worth mentioning):** if the true quotient doesn't fit in the destination register (e.g. EDX ≥ divisor), or the divisor is 0, the CPU raises a `#DE` (Divide Error) exception rather than setting any flag.

---

## Summary

| Instruction | Flags defined | Key behavior |
|---|---|---|
| `add` / `adc` | CF, ZF, SF, OF, PF, AF | CF = unsigned carry-out; OF = signed overflow (same-sign operands → opposite-sign result) |
| `sub` / `sbb` | CF, ZF, SF, OF, PF, AF | CF = unsigned borrow-needed; OF = signed overflow (differing-sign operands → result sign mismatches minuend) |
| `mul` | CF, OF only | Both set together: 1 if product spills into upper half of destination, else 0 |
| `div` | None | All status flags left undefined; only quotient/remainder are guaranteed |