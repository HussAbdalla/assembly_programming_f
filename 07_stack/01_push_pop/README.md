# 01.asm: Basic push and pop

## What the program does
The program loads 100 into RAX and 200 into RBX, pushes both onto the
stack, pops them back into RCX and RDX, and exits with code 0 using the
exit syscall (rax = 60, rdi = 0).

## Stack walkthrough
| Step | Instruction | RSP after | Change | What happened |
|---|---|---|---|---|
| Start | entry of _start | 0x7fffffffdc80 | n/a | nothing pushed yet |
| 1 | push rax | 0x7fffffffdc78 | -8 | 100 stored at the top |
| 2 | push rbx | 0x7fffffffdc70 | -8 | 200 on top, 100 below it |
| 3 | pop rcx | 0x7fffffffdc78 | +8 | rcx = 200 |
| 4 | pop rdx | 0x7fffffffdc80 | +8 | rdx = 100 |

## How RSP changed
The stack grows toward lower addresses. Each push subtracts 8 from RSP
(one 64-bit value) and each pop adds 8. RSP went dc80 -> dc78 -> dc70
on the pushes, then dc70 -> dc78 -> dc80 on the pops. At the start, RSP
already pointed at data the kernel placed there (argc and argv), so the
program's pushes went below that.

## Observations from running it
- The program prints nothing and exits with code 0 (checked with echo $?).
- I pushed RAX then RBX but popped into RCX then RDX, so RCX got 200 and
  RDX got 100. The last value pushed came off first (LIFO).
- x/2gx $rsp after both pushes showed 0xc8 (200) at the top and 0x64
  (100) below it.
- RSP finished at 0x7fffffffdc80, the same as at the start, so every
  push had a matching pop and the stack was balanced.