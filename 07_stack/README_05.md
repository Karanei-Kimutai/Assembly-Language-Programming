# 05 – Saving and Restoring Registers (64-bit)

## Introduction

This program shows how the stack can be used to save register values and put them back later. `RAX` and `RBX` are pushed onto the stack and then popped back into the same registers. Because of LIFO, the pops have to happen in the opposite order to the pushes.

## How the program works

`_start` loads a full 64-bit value, `0x1122334455667788`, into `RAX` and the smaller value `0x1234` into `RBX`. It pushes `RAX` and then `RBX`. After a `nop`, it pops into `RBX` first and then into `RAX`. The program then exits with `RAX = 60` and `RDI = 0`.

In a real program, the code between the pushes and the pops would be something that uses `RAX` and `RBX` for other work. Here there is only a `nop`, so the exercise is about checking that the save-and-restore works correctly.

## Stack operations and observations

`RSP` began at `0x7fffffffd4a0`.

After `push rax`, `RSP` was `0x7fffffffd498`, and that location held `0x1122334455667788`, so all 8 bytes of `RAX` were saved. After `push rbx`, `RSP` was `0x7fffffffd490` and held `0x1234`.

```
Address            Value
0x7fffffffd490     0x0000000000001234   (RBX, pushed second)  <-- RSP
0x7fffffffd498     0x1122334455667788   (RAX, pushed first)
0x7fffffffd4a0     ...
```

Since `RBX` was on top, it had to be popped first. `pop rbx` took `0x1234` and moved `RSP` to `0x7fffffffd498`. `pop rax` took `0x1122334455667788` and moved `RSP` back to `0x7fffffffd4a0`.

If the pops had been written in the same order as the pushes (`pop rax` then `pop rbx`), the two values would have been swapped. This is what happened in exercise 01, where the first pop received the value that was pushed last.

## Register observations

| Execution point | RAX | RBX | RSP |
|---|---|---|---|
| After loading values | 0x1122334455667788 | 0x1234 | 0x7fffffffd4a0 |
| After `push rax` | 0x1122334455667788 | 0x1234 | 0x7fffffffd498 |
| After `push rbx` | 0x1122334455667788 | 0x1234 | 0x7fffffffd490 |
| After `pop rbx` | 0x1122334455667788 | 0x1234 | 0x7fffffffd498 |
| After `pop rax` | 0x1122334455667788 | 0x1234 | 0x7fffffffd4a0 |

Both registers ended with exactly the values they started with, and `RSP` returned to its original value. Each push and pop moved `RSP` by 8 bytes.

## Execution results

The program ran successfully and exited with status `0`. It does not print any output.

## What I learned

The big lesson here is that the pop order must be the reverse of the push order if you want each value to go back to its own register. Using a large value like `0x1122334455667788` was useful because it showed that a 64-bit push saves the whole register, not just part of it. I can see how this would be used before calling a function that might overwrite registers: push them, do the work, then pop them back in reverse order.
