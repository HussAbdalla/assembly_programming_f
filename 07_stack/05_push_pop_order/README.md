# 05.asm: Push/pop order and 64-bit values on the stack

## What the program does
The program loads a large 64-bit value (0x1122334455667788) into RAX and
a small value (0x1234) into RBX. It pushes RAX, then RBX, then pops into
RBX, then RAX, so each register gets its own value back. It then exits
with code 0.

## Stack walkthrough
Let S be the value of RSP when _start begins (<your RSP, e.g. 0x7fffffffdc80>).

| Step | Instruction | RSP after | What happened |
|---|---|---|---|
| Start | entry of _start | S | nothing pushed yet |
| 1 | mov rax / mov rbx | S | no stack change |
| 2 | push rax | S - 8 | 0x1122334455667788 stored at the top |
| 3 | push rbx | S - 16 | 0x0000000000001234 on top, RAX's value below it |
| 4 | pop rbx | S - 8 | RBX = 0x1234 (last pushed, first popped) |
| 5 | pop rax | S | RAX = 0x1122334455667788 |
| 6 | exit syscall | S | program ends with code 0 |

## How RSP changed
Each push subtracted 8 from RSP, because push always writes a full
64-bit slot, even for a small value like 0x1234 (the upper bytes are
zero). Each pop added 8. RSP went S -> S-8 -> S-16 on the pushes and
S-16 -> S-8 -> S on the pops.

## Observations from running it
- The program prints nothing and exits with code 0 (checked with echo $?).
- The pops were in the reverse order of the pushes (rbx first, then rax),
  so each register got its original value back. This is the correct use
  of LIFO. In 01.asm the pops went into different registers, so the
  values ended up swapped.
- The small value 0x1234 still used a full 8-byte slot on the stack.
- RSP finished at the same value it started with (S), so the stack was
  balanced.