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
| `shf \| n` | A shifted left by n places (n from 0 to 15, the instruction's four low bits), zeros shifted in | Works since the shift fix of 2026-11-17 (below); before it A came out unchanged. B plays no part. Flags not tested. |

- The `_32` symbols (`sum_32` …) set a bit the CPU ignores. `sum` and `sum_32` do the same.
- There is no multiply or divide, no add-with-carry, and no right shift. Multiplying by a power of two is `shf | n`.
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

## The shift fix (2026-11-17)
The shift did not work on the CPU as first built. Traced in the engine after the owner pointed at the two places.

- The shifter (four stages for 1, 2, 4 and 8 places, around x 790–925, y 306–350) takes its amount from trace colours 0, 1, 2 and 3 of a bus, at (788,347), (788,338), (788,329) and (788,320).
- That bus runs up to the control logic and ends in the row at y 215, x 1083–1119. Nothing anywhere on it drives colours 0–3, so the amount is always 0 and A passes through unchanged.
- The instruction's low twelve bits arrive on a second bus (the row at y 224, x 1083–1121, fed from y 305). The six cells between the two rows (x 1087–1121, y 216–223) put instruction bits 11–6 or bits 5–0 onto colours 11–6 of the upper bus, which is how the two register fields of Load and Store share one decoder. Colours 0–3 of the lower bus, the instruction's four low bits, are never passed on to the upper bus.

**The fix** (50 pixels, drawn into all three CPU boards of this repository on 2026-11-17 at the owner's request; checked in the engine, not yet run in the game):
- extend the upper bus row (y 215) from x 1120 to 1129 and the lower bus row (y 224) from x 1122 to 1129, in the same bus ink;
- draw four vertical traces from y 216 to y 223: colour 0 at x 1123, colour 1 at x 1125, colour 2 at x 1127, colour 3 at x 1129 (the same four colours the shifter taps).

With it, `shf | n` shifts A left by n places (n from 0 to 15, all 32 bits, zeros shifted in; `0x1234F0F5` by 4 gives `0x234F0F50`). B plays no part. `sum`, `lia` and the day 1 and day 2 programs run exactly as before, tick for tick.

## Changes that would help (suggestions only)
- A right shift.
- An add that takes the carry flag in, for numbers wider than 32 bits.
- Working `xor` and `neg`.
- `lia`/`lib` that leave the upper half alone, or a way to load a 16-bit immediate.
