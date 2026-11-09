# 01 – Basic PUSH and POP Operations

## Introduction

This was the first stack exercise. The program puts two values into registers, pushes both onto the stack, and then pops them off again into two *different* registers. The point is to see the Last In, First Out (LIFO) behaviour of the stack and to watch the stack pointer move as values go on and come off.

## How the program works

The program starts at `_start` by loading `100` into `RAX` and `200` into `RBX`. It then pushes `RAX` first and `RBX` second. A `nop` sits after the pushes so there is a clear point where both values are on the stack.

Next the program pops twice. The first pop goes into `RCX` and the second into `RDX`. Note that the values do not go back into `RAX` and `RBX`, so this exercise is only about the order in which values come off the stack. Finally, the program puts `60` (the `exit` system call) into `RAX`, clears `RDI` so the exit code is 0, and calls `syscall`.

## Stack operations and observations

At the start of the program `RSP` was `0x7fffffffd4a0`.

When `push rax` ran, `RSP` went down by 8 to `0x7fffffffd498` and the value `0x64` (100) was stored at that address. When `push rbx` ran, `RSP` went down by another 8 to `0x7fffffffd490` and `0xc8` (200) was stored there. So the stack really does grow towards lower addresses, and each 64-bit push takes up 8 bytes.

After both pushes the top of the stack looked like this:

```
Address            Value
0x7fffffffd490     0xc8   (200, from RBX)  <-- RSP (top of stack)
0x7fffffffd498     0x64   (100, from RAX)
0x7fffffffd4a0     ...    (original top of stack)
```

The first `pop rcx` took the value at the top, `0xc8`, and moved `RSP` back up to `0x7fffffffd498`. The second `pop rdx` took `0x64` and moved `RSP` back to `0x7fffffffd4a0`, which is exactly where it started.

So 200 was pushed last but came out first, and 100 was pushed first but came out last. That is LIFO.

## Register observations

| Execution point | RAX | RBX | RCX | RDX | RSP |
|---|---|---|---|---|---|
| Program start | 0x0 | 0x0 | 0x0 | 0x0 | 0x7fffffffd4a0 |
| After `mov rax, 100` / `mov rbx, 200` | 0x64 | 0xc8 | 0x0 | 0x0 | 0x7fffffffd4a0 |
| After `push rax` | 0x64 | 0xc8 | 0x0 | 0x0 | 0x7fffffffd498 |
| After `push rbx` | 0x64 | 0xc8 | 0x0 | 0x0 | 0x7fffffffd490 |
| After `pop rcx` | 0x64 | 0xc8 | 0xc8 | 0x0 | 0x7fffffffd498 |
| After `pop rdx` | 0x64 | 0xc8 | 0xc8 | 0x64 | 0x7fffffffd4a0 |

Checking the arithmetic: `0x...d4a0 - 8 = 0x...d498`, and `0x...d498 - 8 = 0x...d490`. The two pops add the 8 bytes back each time, so the total change in `RSP` at the end is zero.

One thing I noticed is that pushing does not clear or change `RAX` and `RBX`. Pushing only copies the value onto the stack.

## Execution results

The program ran successfully. It does not print anything to the terminal, and its exit status was `0`.

## What I learned

Before this exercise I thought of the stack as just a list, but seeing the actual addresses made it clearer. Every push moved `RSP` down by 8 and every pop moved it back up by 8. Popping into `RCX` and `RDX` showed that the stack does not keep track of which register a value came from. It just gives back whatever is on top, so `RCX` ended up with 200 and `RDX` with 100, which is the reverse of the push order.
