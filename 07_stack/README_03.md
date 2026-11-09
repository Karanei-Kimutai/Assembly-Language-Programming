# 03 – Function Calls and Returns

## Introduction

This program focuses on how `CALL` and `RET` use the stack. `_start` puts 10 into `RAX` and calls a small function, `calculate`, which adds 5 to it and returns. The interesting part is how the CPU knows where to go back to after the function finishes.

## How the program works

Execution begins at `_start` with `mov rax, 10`, followed by `call calculate`. The function only has two instructions, `add rax, 5` and `ret`. After returning, execution continues at the label `after_call`, which holds a `nop`. The program then exits through the `exit` system call with code 0.

There is no stack frame in this function (no `push rbp` or `sub rsp`), so the only thing the stack is used for here is the return address.

## Stack operations and observations

Before the call, `RSP` was `0x7fffffffd4a0`. The `call calculate` instruction is at address `0x401005` and is 5 bytes long, so the next instruction (`after_call`) is at `0x40100a`.

When `CALL` ran, `RSP` dropped by 8 to `0x7fffffffd498`, and the value now on top of the stack was `0x40100a`, which is exactly the address of `after_call`. At the same time, the instruction pointer jumped to `0x401015`, the start of `calculate`.

```
Before CALL                     Inside calculate
                                0x7fffffffd498: 0x40100a  <-- RSP (return address)
0x7fffffffd4a0: ...  <-- RSP    0x7fffffffd4a0: ...
```

Inside the function `add rax, 5` doesn't touch the stack at all, so `RSP` stayed at `0x7fffffffd498`.

When `RET` ran, it popped `0x40100a` off the stack into the instruction pointer. `RSP` went back up to `0x7fffffffd4a0` and the next instruction to run was the `nop` at `after_call`.

## Register observations

| Execution point | RAX | RSP | Top of stack | Next instruction |
|---|---|---|---|---|
| Program start | 0x0 | 0x7fffffffd4a0 | 0x1 | `mov rax, 10` (0x401000) |
| Before `call calculate` | 0xa (10) | 0x7fffffffd4a0 | 0x1 | `call` (0x401005) |
| Start of `calculate` | 0xa (10) | 0x7fffffffd498 | 0x40100a | `add rax, 5` (0x401015) |
| Before `ret` | 0xf (15) | 0x7fffffffd498 | 0x40100a | `ret` (0x401019) |
| After `ret` | 0xf (15) | 0x7fffffffd4a0 | 0x1 | `nop` at `after_call` (0x40100a) |

`RAX` went from 10 to 15 inside the function and kept that value after returning, which matches the comment in the source (`RAX should now be 15`). The stack pointer changed by exactly 8 bytes on the `CALL` and 8 bytes back on the `RET`.

## Execution results

The program ran successfully with exit status `0`. It does not produce any terminal output. The result (15) is only visible in `RAX` while the program runs, and `RAX` is then overwritten with 60 for the exit system call.

## What I learned

This exercise made `CALL` and `RET` much less mysterious for me. `CALL` is basically "push the address of the next instruction, then jump", and `RET` is "pop an address and jump to it". Because the return address is just a value on the stack, the function has to leave `RSP` pointing at it when `ret` runs. In this program that was easy because `calculate` never pushed anything, but I can see now why the bigger functions in the other exercises have to clean up their frames first.
