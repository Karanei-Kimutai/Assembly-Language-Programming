# 02 – Stack Frames and Local Variables (64-bit)

## Introduction

This program shows how a function sets up its own stack frame and keeps local variables in it. The function `work` reserves 16 bytes on the stack, stores two values (100 and 200) there, reads them back into registers, and then cleans up the frame before returning to `_start`.

## How the program works

The only thing `_start` does before exiting is `call work`. Inside `work`, the usual prologue runs:

```asm
push rbp        ; save the caller's base pointer
mov rbp, rsp    ; new base pointer = current top of stack
sub rsp, 16     ; reserve 16 bytes for two 8-byte locals
```

The function then writes `100` to `[rbp - 8]` and `200` to `[rbp - 16]`, and loads them back into `RAX` and `RBX`. To finish, it runs `mov rsp, rbp` and `pop rbp` to undo the frame and `ret` to return. Back in `_start`, the program calls the `exit` system call with exit code 0.

## Stack operations and observations

**The call.** Before `call work`, `RSP` was `0x7fffffffd4a0`. After the call, `RSP` was `0x7fffffffd498` and the value at the top of the stack was `0x401005`. That is the address of `mov rax, 60`, the instruction directly after `call work` in `_start`. So `CALL` pushed the return address just like a normal 8-byte push.

**Saving RBP.** `push rbp` moved `RSP` down to `0x7fffffffd490` and stored the old `RBP`, which was `0x0` at that point.

**Setting up the frame.** `mov rbp, rsp` made `RBP` equal to `0x7fffffffd490`, so both pointers pointed at the saved `RBP`. Then `sub rsp, 16` moved `RSP` down to `0x7fffffffd480`, leaving a 16-byte gap between `RSP` and `RBP` for the local variables.

**Local variables.** After both `mov qword` instructions the frame looked like this:

```
Address            Value       Meaning
0x7fffffffd480     0xc8        [rbp - 16] = 200   <-- RSP
0x7fffffffd488     0x64        [rbp - 8]  = 100
0x7fffffffd490     0x0         saved RBP          <-- RBP
0x7fffffffd498     0x401005    return address
0x7fffffffd4a0     ...         caller's stack
```

**Cleaning up.** `mov rsp, rbp` put `RSP` straight back to `0x7fffffffd490`, which throws away the 16 bytes of locals in one step. `pop rbp` restored `RBP` to `0x0` and moved `RSP` to `0x7fffffffd498`, so the return address was on top again. `ret` popped `0x401005` into the instruction pointer, `RSP` went back to `0x7fffffffd4a0`, and execution continued at `mov rax, 60` in `_start`.

I also noticed that the values 100 and 200 were still in memory at `0x...d488` and `0x...d480` after `mov rsp, rbp`. Cleaning up the frame does not erase anything. It only moves the stack pointer, so that area is treated as free space.

## Register observations

| Execution point | RAX | RBX | RSP | RBP |
|---|---|---|---|---|
| Before `call work` | 0x0 | 0x0 | 0x7fffffffd4a0 | 0x0 |
| After `call work` (top = 0x401005) | 0x0 | 0x0 | 0x7fffffffd498 | 0x0 |
| After `push rbp` | 0x0 | 0x0 | 0x7fffffffd490 | 0x0 |
| After `mov rbp, rsp` | 0x0 | 0x0 | 0x7fffffffd490 | 0x7fffffffd490 |
| After `sub rsp, 16` | 0x0 | 0x0 | 0x7fffffffd480 | 0x7fffffffd490 |
| After loading locals | 0x64 | 0xc8 | 0x7fffffffd480 | 0x7fffffffd490 |
| After `mov rsp, rbp` | 0x64 | 0xc8 | 0x7fffffffd490 | 0x7fffffffd490 |
| After `pop rbp` | 0x64 | 0xc8 | 0x7fffffffd498 | 0x0 |
| After `ret` (back in `_start`) | 0x64 | 0xc8 | 0x7fffffffd4a0 | 0x0 |

`RSP` moved down by 8 (call), 8 (push rbp) and 16 (locals), for a total of 32 bytes (`0x...d4a0 - 0x20 = 0x...d480`). On the way out it came back up by 16, 8 and 8, so it finished where it started.

## Execution results

The program ran successfully and exited with status `0`. It does not print any output.

## What I learned

The main thing I took away is why `RBP` is useful. Once `mov rbp, rsp` runs, `RBP` stays fixed for the whole function, so the locals are always at `[rbp - 8]` and `[rbp - 16]` no matter what happens to `RSP` afterwards. If the function pushed more things, `RSP` would keep changing and offsets from it would keep shifting, but offsets from `RBP` would not. It was also good to see the return address sitting right above the saved `RBP`. That made it clear why the frame has to be cleaned up properly before `ret`, otherwise `ret` would jump to the wrong place.
