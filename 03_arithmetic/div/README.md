# div — Effect on FLAGS

> DIV/IDIV do not define any status flags. CF, ZF, SF, OF, PF, AF are all left architecturally undefined. Any flag values seen in a debugger after `div` are leftover artifacts from a prior instruction, not something DIV computed. The only guaranteed outputs are the quotient and remainder.

## div1.asm — 8-bit division
**Operation:** `al = ax / divisor`, `ah = ax % divisor` → 100 ÷ 7
**Result:** AL = 14 (quotient), AH = 2 (remainder)

| Flag | Status | Why |
|------|--------|-----|
| CF, ZF, SF, OF, PF, AF | Undefined | DIV does not set these — not part of the instruction's defined behavior |

---

## div2.asm — 16-bit division (DX:AX dividend)
**Operation:** `ax = (DX:AX) / divisor`, `dx = (DX:AX) % divisor` → 50000 ÷ 300
**Result:** AX = 166 (quotient), DX = 200 (remainder)

| Flag | Status | Why |
|------|--------|-----|
| CF, ZF, SF, OF, PF, AF | Undefined | DIV does not set these |

---

## div3.asm — 32-bit division (EDX:EAX dividend)
**Operation:** `eax = (EDX:EAX) / divisor`, `edx = (EDX:EAX) % divisor` → 300,000,000 ÷ 1000
**Result:** EAX = 300000 (quotient), EDX = 0 (remainder)

| Flag | Status | Why |
|------|--------|-----|
| CF, ZF, SF, OF, PF, AF | Undefined | DIV does not set these |

**Why this matters:** Even though the remainder is exactly 0 here, ZF does not reflect that — DIV simply never touches it. Branching on "divided evenly" requires an explicit `test edx, edx` afterward. If the divisor were 0, or the quotient too large for the destination, the CPU raises a `#DE` exception instead of setting any flag.