# 06.asm: Push/pop order and 32-bit values on the stack

## What the program does
This is the 32-bit (i386) version of the push/pop order program. It
loads 0x11223344 into EAX and 0x1234 into EBX, pushes EAX then EBX,
then pops into EBX then EAX, so each register gets its own value back.
It then exits with code 0 using int 0x80 (eax = 1).

## Stack walkthrough
Let S be the value of ESP when _start begins (<your ESP>). In 32-bit
mode every stack slot is 4 bytes, not 8.

| Step | Instruction | ESP after | What happened |
|---|---|---|---|
| Start | entry of _start | S | nothing pushed yet |
| 1 | mov eax / mov ebx | S | no stack change |
| 2 | push eax | S - 4 | 0x11223344 stored at the top |
| 3 | push ebx | S - 8 | 0x00001234 on top, EAX's value below it |
| 4 | pop ebx | S - 4 | EBX = 0x1234 (last pushed, first popped) |
| 5 | pop eax | S | EAX = 0x11223344 |
| 6 | int 0x80 | S | exit(0) |

## How ESP changed
Each push subtracted 4 from ESP, because in 32-bit mode push writes a
full 32-bit slot, even for a small value like 0x1234 (the upper bytes
are zero). Each pop added 4. ESP went S -> S-4 -> S-8 on the pushes and
S-8 -> S-4 -> S on the pops.

## Observations from running it
- The program prints nothing and exits with code 0 (checked with echo $?).
- The pops were in the reverse order of the pushes (ebx first, then eax),
  so each register got its original value back (LIFO).
- Compared with the 64-bit version (05.asm), the logic is identical but
  every slot is 4 bytes instead of 8, so ESP moves by 4 per push/pop.
- The small value 0x1234 still used a full 4-byte slot on the stack.
- The exit uses int 0x80 with eax = 1, not the syscall instruction with
  rax = 60.
- ESP finished at the same value it started with (S), so the stack was
  balanced.