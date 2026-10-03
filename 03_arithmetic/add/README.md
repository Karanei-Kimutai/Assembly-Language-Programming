## Effect of `ADD` on the EFLAGS Register

**File:** `add1.asm`

**Notes and what happens in the code:**

We first load the 8-bit decimal value `120` from the `num1` memory address into the `AL` register. We then add `10`, stored at `num2`, to `AL`. This gives us `130`, which is finally stored in the `result` memory address.

`AL` is an **8-bit register**, so the result is stored as the 8-bit binary value `10000010`.

**GDB EFLAGS Display:** `[ PF AF SF IF OF ]`

| Flag               | Status      | Why                                                                                                                                                                                   |
| :----------------- | :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **ZF (Zero)**      | Cleared (0) | The result is `130`, so it is not zero.                                                                                                                                               |
| **SF (Sign)**      | Set (1)     | The result is `10000010`. Since the most significant bit is `1`, the CPU treats the result as negative when interpreting it as a signed 8-bit value.                                  |
| **CF (Carry)**     | Cleared (0) | `120 + 10 = 130`, which is still within the 8-bit unsigned range of `0–255`, so there is no carry out of the most significant bit.                                                    |
| **OF (Overflow)**  | Set (1)     | Both numbers being added are positive, but their signed result would be greater than the maximum 8-bit signed value of `127`. The result therefore overflows into the negative range. |
| **PF (Parity)**    | Set (1)     | The result is `10000010`, which contains two `1`s. Since that is an even number of `1`s, the parity flag is set.                                                                      |
| **AF (Auxiliary)** | Set (1)     | Looking at the lower 4 bits, `1000 + 1010` produces a carry into the next nibble, so the auxiliary carry flag is set.                                                                 |

---

**File:** `add2.asm`

**Notes and what happens in the code:**

This time, we load the 16-bit decimal value `32000` from `num1` into the `AX` register. We then add `500` from `num2`. The result is `32500`, which is stored in the `result` memory location.

Since `AX` is a **16-bit register**, we are working with a 16-bit result here.

**GDB EFLAGS Display:** `[ IF ]`

| Flag               | Status      | Why                                                                                                                                |
| :----------------- | :---------- | :--------------------------------------------------------------------------------------------------------------------------------- |
| **ZF (Zero)**      | Cleared (0) | The result is `32500`, so it is not zero.                                                                                          |
| **SF (Sign)**      | Cleared (0) | The 16-bit binary representation of `32500` starts with `0`, so the result is positive when interpreted as a signed value.         |
| **CF (Carry)**     | Cleared (0) | `32000 + 500 = 32500`, which is below the 16-bit unsigned maximum of `65,535`, so no carry occurs out of the most significant bit. |
| **OF (Overflow)**  | Cleared (0) | `32500` is still within the 16-bit signed range of `-32,768` to `32,767`, so there is no signed overflow.                          |
| **PF (Parity)**    | Cleared (0) | The lowest byte of the result is `11110100`, which contains five `1`s. Since five is odd, the parity flag is cleared.              |
| **AF (Auxiliary)** | Cleared (0) | The lower 4 bits do not produce a carry into the next nibble, so the auxiliary carry flag remains cleared.                         |
