## Effect of `SUB` on the EFLAGS Register

**File:** `sub1.asm`

**Notes and what happens in the code:**

We first load the 8-bit decimal value `50` from the `num1` memory address into the `AL` register. We then subtract `80`, stored at `num2`, from `AL`. This gives us `-30`, which is finally stored in the `result` memory address.

Since `AL` is an **8-bit register**, the result is represented in 8-bit two's complement as `11100010`.

**GDB EFLAGS Display:** `[ CF PF SF IF ]`

| Flag               | Status      | Why                                                                                                                                                  |
| :----------------- | :---------- | :--------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ZF (Zero)**      | Cleared (0) | The result is `-30`, so it is not zero.                                                                                                              |
| **SF (Sign)**      | Set (1)     | The 8-bit result is `11100010`. Since the most significant bit is `1`, the CPU treats the result as negative when interpreting it as a signed value. |
| **CF (Carry)**     | Set (1)     | `50 - 80` requires a borrow because `80` is larger than `50` when treated as an unsigned value.                                                      |
| **OF (Overflow)**  | Cleared (0) | The result, `-30`, is still within the 8-bit signed range of `-128` to `127`, so there is no signed overflow.                                        |
| **PF (Parity)**    | Set (1)     | The result is `11100010`, which contains four `1`s. Since that is an even number of `1`s, the parity flag is set.                                    |
| **AF (Auxiliary)** | Cleared (0) | Looking at the lower 4 bits, `0010 - 0000` does not require a borrow from the upper nibble, so the auxiliary carry flag remains cleared.             |

---


**File:** `sub2.asm`

**Notes and what happens in the code:**

This time, we load the 16-bit decimal value `1000` from the `num1` memory address into the `AX` register. We then subtract `2000`, stored at `num2`, from `AX`. This gives us `-1000`, which is finally stored in the `result` memory location.

Since `AX` is a **16-bit register**, the result is represented using 16-bit two's complement.

**GDB EFLAGS Display:** `[ CF PF SF IF ]`

| Flag               | Status      | Why                                                                                                                                      |
| :----------------- | :---------- | :--------------------------------------------------------------------------------------------------------------------------------------- |
| **ZF (Zero)**      | Cleared (0) | The result is `-1000`, so it is not zero.                                                                                                |
| **SF (Sign)**      | Set (1)     | The result is negative, so the most significant bit of the 16-bit result is `1`.                                                         |
| **CF (Carry)**     | Set (1)     | `1000 - 2000` requires a borrow because `2000` is larger than `1000` when treated as an unsigned value.                                  |
| **OF (Overflow)**  | Cleared (0) | The result, `-1000`, is well within the 16-bit signed range of `-32,768` to `32,767`, so there is no signed overflow.                    |
| **PF (Parity)**    | Set (1)     | The lowest byte of the result is `00011000`, which contains two `1`s. Since that is an even number of `1`s, the parity flag is set.      |
| **AF (Auxiliary)** | Cleared (0) | Looking at the lower 4 bits, `1000 - 0000` does not require a borrow from the upper nibble, so the auxiliary carry flag remains cleared. |
