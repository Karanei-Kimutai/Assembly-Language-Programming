## Effect of `DIV` on the EFLAGS Register

**File:** `div1.asm`

**Notes and what happens in the code:**

We first load the 16-bit dividend `100` into the `AX` register. We then load the 8-bit divisor `7` into the `BL` register. When we divide `AX` by `BL`, we get a quotient of `14` and a remainder of `2`.

For an 8-bit `DIV`, the quotient is stored in `AL` and the remainder is stored in `AH`. So, after the division, `AL` contains `14` and `AH` contains `2`.

**GDB EFLAGS Display:** `[ AF IF ]`

| Flag               | Status                           | Why                                                                                                                                                                                  |
| :----------------- | :------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ZF (Zero)**      | Undefined                        | The `DIV` instruction does not define the Zero Flag, so its value after the division cannot be reliably interpreted.                                                                 |
| **SF (Sign)**      | Undefined                        | The Sign Flag is not defined by the `DIV` instruction.                                                                                                                               |
| **CF (Carry)**     | Undefined                        | The Carry Flag is not defined by the `DIV` instruction.                                                                                                                              |
| **OF (Overflow)**  | Undefined                        | The Overflow Flag is not defined by the `DIV` instruction.                                                                                                                           |
| **PF (Parity)**    | Undefined                        | The Parity Flag is not defined by the `DIV` instruction.                                                                                                                             |
| **AF (Auxiliary)** | Undefined (Displayed Set by GDB) | The Auxiliary Carry Flag is also not defined by `DIV`. The `AF` shown by GDB does not represent a result of the division and may simply be the value left by an earlier instruction. |

---

## Effect of `DIV` on the EFLAGS Register

**File:** `div2.asm`

**Notes and what happens in the code:**

This time, we prepare a 32-bit dividend by loading `50000` into `AX` and clearing `DX` to `0`. This gives us the `DX:AX` dividend pair.

We then load the 16-bit divisor `300` into the `BX` register. We divide `DX:AX` by `BX`, giving us a quotient of `166` and a remainder of `200`.

For a 16-bit `DIV`, the quotient is stored in `AX` and the remainder is stored in `DX`.

**GDB EFLAGS Display:** `[ AF IF ]`

| Flag               | Status                           | Why                                                                                                                                                                      |
| :----------------- | :------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ZF (Zero)**      | Undefined                        | The `DIV` instruction does not define the Zero Flag, so its value after the division cannot be reliably interpreted.                                                     |
| **SF (Sign)**      | Undefined                        | The Sign Flag is not defined by the `DIV` instruction.                                                                                                                   |
| **CF (Carry)**     | Undefined                        | The Carry Flag is not defined by the `DIV` instruction.                                                                                                                  |
| **OF (Overflow)**  | Undefined                        | The Overflow Flag is not defined by the `DIV` instruction.                                                                                                               |
| **PF (Parity)**    | Undefined                        | The Parity Flag is not defined by the `DIV` instruction.                                                                                                                 |
| **AF (Auxiliary)** | Undefined (Displayed Set by GDB) | The Auxiliary Carry Flag is not defined by `DIV`. The `AF` shown by GDB does not come from the division itself and may simply be a value left by an earlier instruction. |
