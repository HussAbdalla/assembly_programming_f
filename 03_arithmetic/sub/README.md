# sub — Effect on FLAGS

## sub1.asm — 8-bit subtraction, small minus big
**Operation:** `al = num1 - num2` → 50 − 80
**Result:** `1110 0010` (226 unsigned / −30 signed)

| Flag | Status | Why |
|------|--------|-----|
| CF | 1 | 80 > 50, borrow required out of bit 7 |
| ZF | 0 | Result = 226, not zero |
| SF | 1 | MSB of `1110 0010` is 1 |
| OF | 0 | Both operands positive (same sign) — signed overflow impossible |
| PF | 1 | `1110 0010` has four 1-bits (even) |
| AF | 0 | Low nibbles 2 − 0 = 2, no borrow needed |

**Why this matters:** CF=1 means invalid as unsigned (226 is really a wrapped-around value), but OF=0 means valid as signed (−30 is correctly representable in 8-bit two's complement).

---

## sub2.asm — 16-bit subtraction, small minus big
**Operation:** `ax = num1 - num2` → 1000 − 2000
**Result:** `1111 1100 0001 1000` (64536 unsigned / −1000 signed)

| Flag | Status | Why |
|------|--------|-----|
| CF | 1 | 2000 > 1000, borrow required out of bit 15 |
| ZF | 0 | Result = 64536, not zero |
| SF | 1 | MSB of `0xFC18` is 1 |
| OF | 0 | Both operands positive (same sign) — signed overflow impossible |
| PF | 1 | Low byte `0x18` has two 1-bits (even) |
| AF | 0 | Low nibbles 8 − 0 = 8, no borrow needed |

**Why this matters:** Same CF/OF relationship as sub1.asm — confirms this pattern holds consistently across register widths.

---

## sub3.asm (sbb) — 16-bit subtraction with borrow chaining

### Instruction 1: `sub ax, [num2]` → 0 − 1
**Result:** `0xFFFF` (65535 unsigned / −1 signed)

| Flag | Status | Why |
|------|--------|-----|
| CF | 1 | 1 > 0, borrow required out of bit 15 |
| ZF | 0 | Result = 65535, not zero |
| SF | 1 | MSB of 0xFFFF is 1 |
| OF | 0 | Both operands positive (same sign) — signed overflow impossible |
| PF | 1 | 0xFF has eight 1-bits (even) |
| AF | 1 | Low nibble 0 − 1 needs a borrow |

### Instruction 2: `sbb ax, 0` → 0xFFFF − 0 − CF(1)
**Result:** `0xFFFE`

| Flag | Status | Why |
|------|--------|-----|
| CF | 0 | Effective subtrahend = 0+1 = 1; 0xFFFF ≥ 1, no borrow out of bit 15 |
| ZF | 0 | Result = 65534, not zero |
| SF | 1 | MSB of 0xFFFE is 1 |
| OF | 0 | Minuend negative, result also negative — minuend's sign preserved, no contradiction |
| PF | 0 | 0xFE has seven 1-bits (odd) |
| AF | 0 | 0xF − (0x0 + carry-in 1) = 0xE, no borrow needed |

**Why this matters:** Mirrors `adc` in reverse — `sbb` propagates a borrow into a second subtraction, the basis of multi-word subtraction.