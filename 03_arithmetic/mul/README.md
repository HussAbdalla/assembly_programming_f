# mul — Effect on FLAGS

> MUL only defines **CF** and **OF**, and they always move together: both 1 if the product spilled into the upper half of the destination register, both 0 if it fit entirely in the lower half. ZF, SF, PF, AF are left undefined by MUL.

## mul1.asm — 8-bit multiplication, no overflow
**Operation:** `ax = al * num2` → 25 × 10
**Result:** AX = `0x00FA` (250), AH = 0x00

| Flag | Status | Why |
|------|--------|-----|
| CF | 0 | AH = 0x00 — product fit entirely in AL |
| OF | 0 | Same rule as CF for MUL |
| ZF, SF, PF, AF | Undefined | Not defined by MUL |

---

## mul2.asm — 16-bit multiplication, overflow
**Operation:** `DX:AX = ax * num2` → 3000 × 200 = 600,000
**Result:** DX:AX = `0x0009:0x27C0`

| Flag | Status | Why |
|------|--------|-----|
| CF | 1 | DX = 0x0009 ≠ 0 — product spilled into upper half |
| OF | 1 | Same rule as CF for MUL |
| ZF, SF, PF, AF | Undefined | Not defined by MUL |

**Why this matters:** Direct contrast with mul1.asm — confirms CF and OF collapse into a single question: "did the result fit in the lower half?"

---

## mul3.asm — 32-bit multiplication, overflow
**Operation:** `EDX:EAX = eax * num2` → 100,000 × 300,000 = 30,000,000,000
**Result:** EDX:EAX = `0x00000006:0xFC23AC00`

| Flag | Status | Why |
|------|--------|-----|
| CF | 1 | EDX = 0x00000006 ≠ 0 — product spilled into upper half |
| OF | 1 | Same rule as CF for MUL |
| ZF, SF, PF, AF | Undefined | Not defined by MUL |

**Why this matters:** Confirms the CF/OF rule holds consistently across byte, word, and dword operand widths.