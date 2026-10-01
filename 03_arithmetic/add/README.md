# add — Effect on FLAGS

## add1.asm — 8-bit addition, signed overflow case
**Operation:** `al = num1 + num2` → 120 + 10
**Result:** `1000 0010` (130 unsigned / −126 signed)

| Flag | Status | Why |
|------|--------|-----|
| CF | 0 | 120 + 10 = 130 fits within 0–255, no carry out of bit 7 |
| ZF | 0 | Result = 130, not zero |
| SF | 1 | MSB of `1000 0010` is 1 |
| OF | 1 | Two positive operands produced a negative (MSB=1) result — signed overflow |
| PF | 1 | `1000 0010` has two 1-bits (even) |
| AF | 1 | Low nibbles 8 + 10 = 18 > 15, carries into bit 4 |

**Why this matters:** Result is valid as unsigned (130) but invalid as signed (overflow). CF=0 says "fine for unsigned," OF=1 says "broken for signed" — same bits, two different judgments depending on interpretation.

---

## add2.asm — 16-bit addition, no boundary triggered
**Operation:** `ax = num1 + num2` → 32000 + 500
**Result:** `0111 1110 1111 0100` (32500)

| Flag | Status | Why |
|------|--------|-----|
| CF | 0 | 32500 fits within 0–65535, no carry out |
| ZF | 0 | Result = 32500, not zero |
| SF | 0 | MSB of `0x7EF4` is 0 |
| OF | 0 | Both operands positive, result positive — no contradiction |
| PF | 0 | Low byte `0xF4` has five 1-bits (odd) |
| AF | 0 | Low nibbles 0 + 4 = 4, no carry into bit 4 |

**Why this matters:** All flags clear — contrasts with add1.asm: same instruction, but operands stay well within range for a 16-bit register, so no boundary condition triggers.

---

## add3.asm / adc4.asm — 16-bit addition with carry chaining

### Instruction 1: `add ax, [num2]` → 0xFFFF + 1
**Result:** `0x0000` (wraparound)

| Flag | Status | Why |
|------|--------|-----|
| CF | 1 | 65535 + 1 = 65536 needs a 17th bit |
| ZF | 1 | Result wrapped to 0 |
| SF | 0 | MSB of 0x0000 is 0 |
| OF | 0 | Operands have opposite signs (−1 and +1) — signed overflow impossible |
| PF | 1 | 0x00 has zero set bits (even) |
| AF | 1 | Low nibbles 0xF + 0x1 = 0x10, carries into bit 4 |

### Instruction 2: `adc ax, 0` → 0 + 0 + CF(1)
**Result:** `0x0001`

| Flag | Status | Why |
|------|--------|-----|
| CF | 0 | 0+0+1 = 1, no carry out of bit 15 |
| ZF | 0 | Result = 1, not zero |
| SF | 0 | MSB of 0x0001 is 0 |
| OF | 0 | Destination and operand positive, result positive — no contradiction |
| PF | 0 | 0x01 has one set bit (odd) |
| AF | 0 | 0x0+0x0+carry-in(1) = 0x1, no carry into bit 4 |

**Why this matters:** Classic unsigned wraparound (CF=1, ZF=1, OF=0 since operand signs differed). `adc` propagates the carry into a second addition — the basis of multi-word addition across registers wider than hardware natively supports.