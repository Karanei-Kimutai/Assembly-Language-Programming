# 06 – Saving and Restoring Registers (32-bit)

## Introduction

This is the 32-bit version of exercise 05. `EAX` and `EBX` are saved on the stack and restored by popping them back in reverse order. I wanted to confirm that the idea works the same way in 32-bit mode and to see how much `ESP` moves each time.

## How the program works

The program sets `EAX = 0x11223344` and `EBX = 0x1234`, then runs `push eax` followed by `push ebx`. After a `nop`, it does `pop ebx` and then `pop eax`. Finally it exits using the 32-bit system call: `EAX = 1`, `EBX = 0`, `int 0x80`.

## Stack operations and observations

`ESP` started at `0xffffc650`.

`push eax` lowered `ESP` by 4 to `0xffffc64c` and stored `0x11223344` there. `push ebx` lowered it another 4 to `0xffffc648` and stored `0x00001234`.

```
Address       Value
0xffffc648    0x00001234   (EBX, pushed second)  <-- ESP
0xffffc64c    0x11223344   (EAX, pushed first)
0xffffc650    ...
```

`pop ebx` read `0x1234` from the top and raised `ESP` to `0xffffc64c`. `pop eax` read `0x11223344` and raised `ESP` back to `0xffffc650`.

## Register observations

| Execution point | EAX | EBX | ESP |
|---|---|---|---|
| After loading values | 0x11223344 | 0x1234 | 0xffffc650 |
| After `push eax` | 0x11223344 | 0x1234 | 0xffffc64c |
| After `push ebx` | 0x11223344 | 0x1234 | 0xffffc648 |
| After `pop ebx` | 0x11223344 | 0x1234 | 0xffffc64c |
| After `pop eax` | 0x11223344 | 0x1234 | 0xffffc650 |

Both registers were restored to their original values and `ESP` ended where it started.

## Comparison with 05.asm

| | 64-bit (05) | 32-bit (06) |
|---|---|---|
| Starting stack pointer | 0x7fffffffd4a0 | 0xffffc650 |
| After first push | 0x7fffffffd498 (−8) | 0xffffc64c (−4) |
| After second push | 0x7fffffffd490 (−8) | 0xffffc648 (−4) |
| Total moved by two pushes | 16 bytes | 8 bytes |
| Value pushed from first register | 0x1122334455667788 | 0x11223344 |

The difference comes from the register size. `RAX` is 64 bits (8 bytes), so a push in 64-bit mode reserves 8 bytes. `EAX` is 32 bits (4 bytes), so a push in 32-bit mode only needs 4. That is also why the 64-bit program could store a much longer value like `0x1122334455667788`, while the 32-bit version uses `0x11223344`. The stack addresses themselves are also shorter in the 32-bit program, since it runs with 32-bit addresses.

## Execution results

The program ran successfully and exited with status `0`. It does not produce any terminal output.

## What I learned

The register-saving technique doesn't depend on 32-bit or 64-bit mode. It only depends on popping in the reverse order of pushing. What changes between the two modes is how far the stack pointer moves for each push and pop: 4 bytes here compared with 8 bytes in exercise 05. Putting the two programs side by side made the reason clear, because the stack pointer always moves by the size of the register being saved.
