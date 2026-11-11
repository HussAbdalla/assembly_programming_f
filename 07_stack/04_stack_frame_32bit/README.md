# 04.asm: Function call and stack frame (32-bit)

## What the program does
This is the 32-bit (i386) version of the stack-frame program. _start
calls work. work builds a stack frame, stores 100 and 200 as two local
variables, reads them back into EAX and EBX, tears the frame down and
returns. _start then exits with code 0 using int 0x80 (eax = 1).

## Stack walkthrough
Let S be the value of ESP when _start begins (0xffffcd40). In 32-bit
mode every stack slot is 4 bytes, not 8.

| Step | Instruction | ESP after | EBP after | What happened |
|---|---|---|---|---|
| Start | entry of _start | S | caller's | nothing pushed yet |
| 1 | call work | S - 4 | unchanged | return address pushed |
| 2 | push ebp | S - 8 | unchanged | caller's EBP saved |
| 3 | mov ebp, esp | S - 8 | S - 8 | frame base set, ESP unchanged |
| 4 | sub esp, 8 | S - 16 | S - 8 | 8 bytes reserved for two locals |
| 5 | mov dword [ebp-4], 100 | S - 16 | S - 8 | 100 stored at S - 12, no push |
| 6 | mov dword [ebp-8], 200 | S - 16 | S - 8 | 200 stored at S - 16, no push |
| 7 | mov eax, [ebp-4] / mov ebx, [ebp-8] | S - 16 | S - 8 | eax = 100, ebx = 200 |
| 8 | mov esp, ebp | S - 8 | S - 8 | locals discarded |
| 9 | pop ebp | S - 4 | caller's | EBP restored |
| 10 | ret | S | caller's | return address popped, back in _start |
| 11 | int 0x80 | S | caller's | exit(0) |

## How ESP changed
The stack grows toward lower addresses, and in 32-bit mode each push
or call moves ESP by 4. call pushed the return address (ESP -4) and
push ebp saved the caller's base pointer (ESP -4). mov ebp, esp made
EBP mark the frame base, so the locals can be reached at fixed offsets
[ebp-4] and [ebp-8] while ESP is free to move. sub esp, 8 reserved
space for the two locals, and they were written with mov, not push, so
ESP did not change while storing them. mov esp, ebp threw the locals
away, pop ebp restored the caller's EBP (ESP +4) and ret popped the
return address (ESP +4), landing back in _start.

## Observations from running it
- The program prints nothing and exits with code 0 (checked with echo $?).
- Compared with the 64-bit version (02.asm), the layout is the same but
  every slot is 4 bytes, so the frame is smaller: ESP moves by 4 per
  push/pop instead of 8, and the locals are at [ebp-4] and [ebp-8].
- The exit uses int 0x80 with eax = 1, not the syscall instruction with
  rax = 60.
- ESP finished at the same value it started with (S), so the stack was
  balanced.