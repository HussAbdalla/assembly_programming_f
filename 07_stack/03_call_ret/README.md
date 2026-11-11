# 03.asm: call and ret with a return address

## What the program does
The program puts 10 in RAX and calls the function calculate. calculate
adds 5 to RAX (making it 15) and returns. Execution continues at the
after_call label, and the program exits with code 0.

## Stack walkthrough
Let S be the value of RSP when _start begins (<your RSP, e.g. 0x7fffffffdc80>).

| Step | Instruction | RSP after | What happened |
|---|---|---|---|
| Start | entry of _start | S | nothing pushed yet |
| 1 | mov rax, 10 | S | no stack change |
| 2 | call calculate | S - 8 | return address (the address of after_call) pushed |
| 3 | add rax, 5 | S - 8 | RAX = 15, no stack change |
| 4 | ret | S | return address popped into RIP, back at after_call |
| 5 | exit syscall | S | program ends with code 0 |

## How RSP changed
The only instructions that touched the stack were call and ret. call
pushed the 8-byte address of the next instruction (after_call), so
RSP dropped by 8. Inside calculate nothing was pushed, so RSP stayed
at S - 8 and the return address sat at the top of the stack. ret popped
that address into the instruction pointer and RSP rose by 8, back to S.

## Observations from running it
- The program prints nothing and exits with code 0 (checked with echo $?).
- The stack is how ret knows where to go back to: without the pushed
  return address, execution would not resume at after_call.
- RSP ended at the same value it started with, so the stack was balanced.
- Because calculate has no local variables, it needs no stack frame
  (no push rbp / mov rbp, rsp), unlike 02.asm.