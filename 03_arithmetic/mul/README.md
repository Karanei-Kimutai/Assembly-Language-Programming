## Effect of `MUL` on the EFLAGS Register

**File:** `mul1.asm`

**Notes and what happens in the code:**

We first load the 8-bit decimal value `25` into the `AL` register. We then multiply `AL` by the 8-bit decimal value `10` stored at the `num2` memory address. This gives us a product of `250`.

Since `MUL` with an 8-bit operand produces a 16-bit result, the product is stored in the `AX` register. In this case, the result is `00000000 11111010`, so `AH` contains `0` while `AL` contains `250`. The value in `AX` is then moved to the `result` memory address.

**GDB EFLAGS Display:** `[ IF ]`

| Flag               | Status      | Why                                                                                                                                                           |
| :----------------- | :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **ZF (Zero)**      | Undefined   | The `MUL` instruction does not define the Zero Flag, so its value after the multiplication should not be used as an indication of whether the result is zero. |
| **SF (Sign)**      | Undefined   | The Sign Flag is not defined by the `MUL` instruction.                                                                                                        |
| **CF (Carry)**     | Cleared (0) | The product is `250`, which fits entirely in the lower 8 bits. The upper half of the result, `AH`, is `0`, so the Carry Flag is cleared.                      |
| **OF (Overflow)**  | Cleared (0) | For `MUL`, the Overflow Flag has the same status as the Carry Flag. Since `AH` is `0`, the Overflow Flag is cleared.                                          |
| **PF (Parity)**    | Undefined   | The Parity Flag is not defined by the `MUL` instruction.                                                                                                      |
| **AF (Auxiliary)** | Undefined   | The Auxiliary Carry Flag is not defined by the `MUL` instruction.                                                                                             |

---


**File:** `mul2.asm`

**Notes and what happens in the code:**

This time, we load the 16-bit decimal value `3000` from the `num1` memory address into the `AX` register. We then multiply `AX` by the 16-bit value `200` stored at `num2`. This gives us a 32-bit product of `600,000`.

Since a 16-bit `MUL` produces a 32-bit result, the CPU stores the lower 16 bits in `AX` and the upper 16 bits in `DX`. The result is therefore split between the two registers before being moved to the `result` memory space.

**GDB EFLAGS Display:** `[ IF ]` *(See note below)*

| Flag               | Status    | Why                                                                                                                                        |
| :----------------- | :-------- | :----------------------------------------------------------------------------------------------------------------------------------------- |
| **ZF (Zero)**      | Undefined | The Zero Flag is not defined by the `MUL` instruction.                                                                                     |
| **SF (Sign)**      | Undefined | The Sign Flag is not defined by the `MUL` instruction.                                                                                     |
| **CF (Carry)**     | Set (1)   | The product `600,000` does not fit entirely in the lower 16 bits. The upper 16 bits stored in `DX` are non-zero, so the Carry Flag is set. |
| **OF (Overflow)**  | Set (1)   | For `MUL`, the Overflow Flag has the same status as the Carry Flag. Since `DX` contains non-zero data, the Overflow Flag is set.           |
| **PF (Parity)**    | Undefined | The Parity Flag is not defined by the `MUL` instruction.                                                                                   |
| **AF (Auxiliary)** | Undefined | The Auxiliary Carry Flag is not defined by the `MUL` instruction.                                                                          |
