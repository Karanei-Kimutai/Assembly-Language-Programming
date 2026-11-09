# 04 – Stack Frames and Local Variables (32-bit)

## Introduction

This is the 32-bit version of exercise 02. The function `work` builds a stack frame using `EBP` and `ESP`, stores 100 and 200 as local variables, reads them back into `EAX` and `EBX`, and returns. I mainly wanted to compare how the frame looks when every stack slot is 4 bytes instead of 8.

## How the program works

`_start` calls `work` straight away. The function starts with the 32-bit prologue:

```asm
push ebp
mov ebp, esp
sub esp, 8      ; two 4-byte locals
```

It stores `100` at `[ebp - 4]` and `200` at `[ebp - 8]`, loads them into `EAX` and `EBX`, and then cleans up with `mov esp, ebp`, `pop ebp` and `ret`. Back in `_start`, the program exits through the 32-bit Linux system call interface: `EAX = 1` (exit), `EBX = 0` (exit code), and `int 0x80`.

## Stack operations and observations

**The call.** `ESP` was `0xffffc650` before the call. `call work` pushed the return address `0x08049005`, which is the address of `mov eax, 1` in `_start`, and `ESP` dropped by 4 to `0xffffc64c`.

**Saving EBP.** `push ebp` stored the old `EBP` (`0x0`) and moved `ESP` to `0xffffc648`.

**New frame.** `mov ebp, esp` set `EBP` to `0xffffc648`. Then `sub esp, 8` moved `ESP` to `0xffffc640`, which is enough space for two 32-bit values.

**Local variables.** After the two stores the frame looked like this:

```
Address       Value        Meaning
0xffffc640    0x000000c8   [ebp - 8] = 200   <-- ESP
0xffffc644    0x00000064   [ebp - 4] = 100
0xffffc648    0x00000000   saved EBP         <-- EBP
0xffffc64c    0x08049005   return address
0xffffc650    ...          caller's stack
```

**Cleaning up.** `mov esp, ebp` brought `ESP` back to `0xffffc648`. `pop ebp` restored `EBP` to `0x0` and moved `ESP` to `0xffffc64c`. `ret` popped `0x08049005` and `ESP` returned to `0xffffc650`, its original value.

## Register observations

| Execution point | EAX | EBX | ESP | EBP |
|---|---|---|---|---|
| Before `call work` | 0x0 | 0x0 | 0xffffc650 | 0x0 |
| After `call work` (top = 0x08049005) | 0x0 | 0x0 | 0xffffc64c | 0x0 |
| After `push ebp` | 0x0 | 0x0 | 0xffffc648 | 0x0 |
| After `mov ebp, esp` | 0x0 | 0x0 | 0xffffc648 | 0xffffc648 |
| After `sub esp, 8` | 0x0 | 0x0 | 0xffffc640 | 0xffffc648 |
| After loading locals | 0x64 | 0xc8 | 0xffffc640 | 0xffffc648 |
| After `mov esp, ebp` | 0x64 | 0xc8 | 0xffffc648 | 0xffffc648 |
| After `pop ebp` | 0x64 | 0xc8 | 0xffffc64c | 0x0 |
| After `ret` | 0x64 | 0xc8 | 0xffffc650 | 0x0 |

The total drop in `ESP` was 4 + 4 + 8 = 16 bytes (`0xffffc650 - 0x10 = 0xffffc640`).

## Comparison with 02.asm

The structure is identical, but the sizes are halved:

| Step | 64-bit (02) | 32-bit (04) |
|---|---|---|
| `CALL` (return address) | RSP − 8 | ESP − 4 |
| Push base pointer | RSP − 8 | ESP − 4 |
| Space for two locals | `sub rsp, 16` | `sub esp, 8` |
| Local offsets | `[rbp-8]`, `[rbp-16]` | `[ebp-4]`, `[ebp-8]` |
| Total frame size | 32 bytes | 16 bytes |

In the 64-bit program the return address and saved base pointer each take 8 bytes because addresses are 64 bits wide. In the 32-bit program they only need 4 bytes. The locals are also `dword` (4 bytes) here instead of `qword` (8 bytes), which is why the offsets are spaced 4 apart.

## Execution results

The program ran successfully and exited with status `0`. Nothing is printed to the terminal.

## What I learned

Doing the same exercise in 32-bit made it obvious that the frame layout is the same idea in both modes: return address, then saved base pointer, then locals below that. The only real difference is the slot size, 4 bytes instead of 8. I also had to remember that the 32-bit exit is different (`eax = 1`, `int 0x80`) from the 64-bit one (`rax = 60`, `syscall`).
