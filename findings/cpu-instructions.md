# What the CPU's instructions do

Found on 2026-11-17 with small test programs run in the `vcb` engine on `cpu-32bit-autorun-aoc1p1.vcb` (the engine reproduces the game tick for tick). The symbols are those at the top of the day programs. Anything not listed here was not tested.

## The machine
- **Registers:** A and B are 32 bits wide. `rgA` and `rgB` name their low 16 bits, `rgAH` and `rgBH` their high 16 bits.
- **Memory:** 16-bit words at 16-bit addresses.
- **Start:** the CPU begins at address `0xFF00`. Only 256 words follow before the address wraps, so a longer program starts with `jmp; MAIN` there and lives elsewhere.
- **End:** the CPU stops on its Breakpoint when it executes the word 0 (`non`).
- **Speed:** about 240 ticks per instruction on average (day 2 part 1: 43.5 million ticks).

## Loading and storing
| Instruction | Effect |
|---|---|
| `lia n` | A = n (12 bits). **Clears rgAH.** |
| `lib n` | B = n (12 bits). **Clears rgBH.** |
| `load X rgB` | X = the word at address B. X is `rgA`, `rgAH`, `rgB` or `rgBH`; only those 16 bits change. |
| `load X rgA` | The same with the address in A. So `lia v; load rgA rgA` loads a variable into A with rgAH 0, and `lib v; load rgB rgB` does it for B, without touching the other register. |
| `store X rgB`, `store X rgA` | The word at address B (or A) = X. |

- Naming two registers in one load (`rgA | rgAH`) puts the same word in both. Naming two in one store or push writes the OR of them. Neither is useful.
- An address above `0x0FFF` cannot be an immediate. Keep it in a variable and load it.
- Reading a variable through a pointer: `lib P; load rgB rgB; load rgA rgB`.

## Arithmetic and logic
All work on the full 32 bits: A = A op B.

| Instruction | Result | Flags afterwards |
|---|---|---|
| `sum` | A + B | zero: result is 0. carry: carry out of bit 31. nega: bit 31 of the result. |
| `sub` | A - B | zero: equal. carry: A >= B (no borrow). nega: A < B when both are below 2^31. |
| `and`, `or` | bitwise | **All three flags are cleared**, also when the result is 0. |
| `xor`, `neg` | A becomes 0 | Not working. |
| `shf` | A unchanged, for every operand value tried | Not working. |

- The `_32` symbols (`sum_32` …) set a bit the CPU ignores. `sum` and `sum_32` do the same.
- There is no multiply, divide or shift, and no add-with-carry.
- Adding numbers wider than 32 bits: keep them as 16-bit words, add two words in the 32-bit registers, store the low half, and the carry is what is left in rgAH (`psh rgAH; pop rgB` moves it into B for the next word).

## Jumps
`jmp <conditions>; <address>`. With no condition it always jumps; with several it jumps when any holds.

| Condition | Holds when |
|---|---|
| `zero`, `carry`, `nega` | The flag from the last `sum` or `sub` is set. Loads, stores, pushes and pops leave the flags alone. |
| `eq`, `neq` | A and B are equal (or not) **right now**, all 32 bits. No operation is needed first. |

## Stack and calls
- `psh X` pushes one 16-bit register half; `pop X` pops into one. `psh rgA; pop rgB` copies A's low half into B and leaves rgBH alone.
- `call; <address>` and `ret` work and nest.
- `psh rgSP` pushes the address of the code being executed (the program counter), `psh rgSB` pushed `0xE000` (the stack). Popping into them was not tested.

## Changes that would help (suggestions only)
- A working shift (left by one would already give multiply by doubling).
- An add that takes the carry flag in, for numbers wider than 32 bits.
- Working `xor` and `neg`.
- `lia`/`lib` that leave the upper half alone, or a way to load a 16-bit immediate.
