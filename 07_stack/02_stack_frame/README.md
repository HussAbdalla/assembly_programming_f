# 02.asm: Function call and stack frame

## What the program does
_start calls the function work. work builds a stack frame, stores 100
and 200 as two local variables, reads them back into RAX and RBX,
tears the frame down and returns. _start then exits with code 0.

## Stack walkthrough
(paste the table above here)

## How RSP changed
call pushed the return address (RSP -8). push rbp saved the caller's
base pointer (RSP -8). mov rbp, rsp copied RSP into RBP, so RBP marks
the frame base and stays fixed while RSP moves. sub rsp, 16 reserved
space for two locals (RSP -16). The locals were written with mov, not
push, so RSP did not change while storing them. mov rsp, rbp threw the
locals away, pop rbp restored the caller's RBP (RSP +8), and ret popped
the return address (RSP +8), landing back on line 6 of _start.

## Observations from running it
- The program prints nothing and exits with code 0.
- Local variables live at [rbp-8] and [rbp-16], below the saved RBP
  and the return address.
- The return address was 0x401005, and ret jumped back to it.
- RSP finished at 0x7fffffffdc80, the same as at the start, so the
  stack was balanced.
- Full gdb trace saved in 02_gdb_trace.txt.