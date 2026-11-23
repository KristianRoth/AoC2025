# Advent of Code 2025 on the 32-bit CPU

Every puzzle of Advent of Code 2025 solved by a program running on a 32-bit CPU built in the Steam game Virtual Circuit Board (Update 1.0.1). The CPU is a drawing of wires and gates; the game simulates it tick by tick.

## The solutions and their speed

| Solution | Answer | Ticks to the Breakpoint | In the engine | In the game |
|---|---|---|---|---|
| [Day 1 part 1](days/day1/part1.vcbasm) | 1031 | 43,146,512 | 0.3 s | 9 s |
| [Day 1 part 2](days/day1/part2.vcbasm) | 5831 | 70,339,472 | 0.6 s | 14 s |
| [Day 2 part 1](days/day2/part1.vcbasm) | 54641809925 | 43,495,312 | 0.2 s | 9 s |
| [Day 2 part 2](days/day2/part2.vcbasm) | 73694270688 | 75,753,616 | 0.2 s | 15 s |
| [Day 3 part 1](days/day3/part1.vcbasm) | 16993 | 170,904,144 | 0.5 s | 34 s |
| [Day 3 part 2](days/day3/part2.vcbasm) | 168617068915447 | 419,669,712 | 1.6 s | 84 s |
| [Day 4 part 1](days/day4/part1.vcbasm) | 1505 | 481,327,888 | 2.3 s | 1.6 min |
| [Day 4 part 2](days/day4/part2.vcbasm) | 9182 | 1,339,421,264 | 6.8 s | 4.5 min |
| [Day 5 part 1](days/day5/part1.vcbasm) | 798 | 800,901,776 | 2.7 s | 2.7 min |
| [Day 5 part 2](days/day5/part2.vcbasm) | 366181852921027 | 258,090,384 | 0.7 s | 52 s |
| [Day 6 part 1](days/day6/part1.vcbasm) | 3261038365331 | 455,100,048 | 1.2 s | 1.5 min |
| [Day 6 part 2](days/day6/part2.vcbasm) | 8342588849093 | 613,674,000 | 2.7 s | 2.0 min |
| [Day 7 part 1](days/day7/part1.vcbasm) | 1660 | 117,191,312 | 0.6 s | 23 s |
| [Day 7 part 2](days/day7/part2.vcbasm) | 305999729392659 | 250,336,656 | 1.0 s | 50 s |
| [Day 8 part 1](days/day8/part1.vcbasm) | 62186 | 1,110,321,360 | 7.8 s | 3.7 min |
| [Day 8 part 2](days/day8/part2.vcbasm) | 8420405530 | 3,875,322,704 | 27 s | 13 min |
| [Day 9 part 1](days/day9/part1.vcbasm) | 4771508457 | 205,937,872 | 1.3 s | 41 s |
| [Day 9 part 2](days/day9/part2.vcbasm) | 1539809693 | 1,329,065,552 | 3.5 s | 4.4 min |
| [Day 10 part 1](days/day10/part1.vcbasm) | 475 | 1,010,771,664 | 3.1 s | 3.4 min |
| [Day 10 part 2](days/day10/part2.vcbasm) | 18273 | 10,976,582,736 | 14 s | 37 min |
| [Day 11 part 1](days/day11/part1.vcbasm) | 652 | 22,973,904 | 0.2 s | 5 s |
| [Day 11 part 2](days/day11/part2.vcbasm) | 362956369749210 | 269,092,752 | 1.5 s | 54 s |
| [Day 12](days/day12/part1.vcbasm) | 427 | 464,586,704 | 0.8 s | 1.5 min |

- **Ticks** are exact and the same in the game (the game's counter shows one more). All 23 programs together take 24,404,007,344 ticks.
- **In the engine** is one run in the `vcb` engine of the VirtualCircuitBoardCPU repository, which reproduces the game tick for tick, measured on 2026-11-23. These times are rough and change from run to run.
- **In the game** is worked out from the ticks at 5 million ticks a second; it was not measured in the game.
- Every program stops with its answer in memory from `0x0100` on, as binary words and as decimal digits one per word from `0x0110`. The header of each file says exactly where.
- Each folder also has `partN-example.vcbasm`, the same program with the puzzle's example as input, and `cpu-32bit-autorun-aocNpM.vcb`, a board that carries the program for running it in the game.
- **Day 12** decides every region of the puzzle input for certain, but does not solve the puzzle's small example: it reports the example's three regions as undecided (the header of its file explains why).

The rules the puzzles were solved under are in [`RULES.md`](RULES.md): only the form of the input is changed outside the CPU, and every decision and every sum happens in the program.

## How the CPU is set up

- **Two 32-bit registers, A and B.** Arithmetic and logic work on A with B: add, subtract, and, or, xor, not, and a shift to the left. There is no multiply, no divide and no shift to the right.
- **Memory of 16-bit words at 16-bit addresses.** Program, data and variables share it. A register's two halves are loaded and stored as two separate words.
- **It starts at `0xFF00`** and runs until it executes the word 0, which stops it on a Breakpoint. Only 256 words follow `0xFF00`, so a program jumps from there to its code at `0x1000`.
- **A stack** for `psh`, `pop`, `call` and `ret` starts at `0xE000` and grows upwards.
- **A program is assembly text** in the game's Assembly Editor: the game assembles it into the CPU's memory when a simulation starts. The puzzle input is part of that text, as data words.
- **The answer is read from memory.** With the game's Persistent Memory set to the answer's addresses, the words stay in the project after the run.
- **Two boards:** `cpu-32bit-manual-start.vcb` is the CPU as built, started by clicking its run switch in the game; `cpu-32bit-autorun-aoc1p1.vcb` has the switch drawn on, so it runs from the first tick. The boards in the day folders are the second one with another program and Persistent Memory range.
- **Two fixes** were drawn into every board: one that makes the shift work and one for xor and not. They and everything else found out about the CPU by test programs are in [`findings/cpu-instructions.md`](findings/cpu-instructions.md).

To run a solution in the engine, from `engine/` in the VirtualCircuitBoardCPU repository:

```
./target/release/vcb run ../AoC2025/cpu-32bit-autorun-aoc1p1.vcb --asm ../AoC2025/days/day5/part1.vcbasm --ticks 2000000000 --watch 0x100
```

## Instruction set

How to read this: an instruction is one 16-bit memory word. The names in the first column are the symbols every program defines at its top (copy them from any day's file). `jmp` and `call` are followed by a second word, the address. How each behaviour was found, and the fixes to the boards, are in [`findings/cpu-instructions.md`](findings/cpu-instructions.md).

### The machine

| Part | What it is |
|---|---|
| A, B | Two 32-bit registers. `rgA` and `rgB` name their lower 16 bits, `rgAH` and `rgBH` their upper 16 bits. |
| Memory | 16-bit words at 16-bit addresses (`0x0000`–`0xFFFF`). |
| Flags | `zero`, `carry`, `nega`: set by `sum` and `sub`, kept until the next one. `eq`, `neq`: not stored, they compare A with B at the moment of the jump. |
| Stack | Used by `psh`, `pop`, `call`, `ret`. It starts at `0xE000`. |
| Start | The CPU begins at `0xFF00`. Only 256 words follow, so a program puts `jmp; MAIN` there and lives elsewhere. |
| Stop | The word `0` (`non`) stops the CPU on its Breakpoint. |

### Instructions

`n` is a number from 0 to 4095. `X` is one of `rgA`, `rgAH`, `rgB`, `rgBH`. Ticks are measured in the engine.

| Write | Word (bits) | What it does | Flags | Ticks |
|---|---|---|---|---|
| `lia n` | `0011 nnnn nnnn nnnn` | A = n. **The upper half of A becomes 0.** | unchanged | 256 |
| `lib n` | `0100 nnnn nnnn nnnn` | B = n. **The upper half of B becomes 0.** | unchanged | 256 |
| `load X rgB` | `0010 dddd dd00 0100` | X = the memory word at the address in B. Only those 16 bits of the register change. | unchanged | 256 |
| `load X rgA` | `0010 dddd dd01 0000` | The same, with the address in A. | unchanged | 256 |
| `store X rgB` | `0101 ssss ss00 0100` | The memory word at the address in B = X. | unchanged | 256 |
| `store X rgA` | `0101 ssss ss01 0000` | The same, with the address in A. | unchanged | 256 |
| `sum` | `0110 0100 0000 0000` | A = A + B, all 32 bits. | zero: result is 0. carry: carried out of bit 31. nega: bit 31 of the result. | 256 |
| `sub` | `0110 0010 0000 0000` | A = A - B, all 32 bits. | zero: A was equal to B. carry: A >= B (no borrow). nega: A < B (for values below 2^31). | 256 |
| `and` | `0110 0000 1000 0000` | A = A and B, bit by bit. | **not usable** (see below) | 256 |
| `or` | `0110 0000 0100 0000` | A = A or B, bit by bit. | not usable | 256 |
| `xor` | `0110 0000 0010 0000` | A = A xor B, bit by bit. | not usable | 256 |
| `neg` | `0110 0000 0001 0000` | A = every bit of A flipped (NOT). For -A, add 1 afterwards. | not usable | 256 |
| `shf \| k` | `0110 0001 0000 kkkk` | A = A shifted left by k places (k from 0 to 15), zeros shifted in. There is no right shift. | not tested | 256 |
| `psh X` | `1010 xxxx xx00 0000` | Push the 16 bits of X on the stack. | unchanged | 576 with its `pop` |
| `pop X` | `1011 xxxx xx00 0000` | Pop 16 bits from the stack into X. The other half of the register is unchanged. | unchanged | |
| `jmp; addr` | `1100 0000 0000 0000`, then the address | Continue at addr. | unchanged | 320 |
| `jmp c; addr` | `1100 cccc c000 0000`, then the address | Continue at addr if condition c holds; otherwise go on. Several conditions: jump if any holds. | unchanged | 320 taken, 192 not |
| `call; addr` | `1110 0000 0000 0000`, then the address | Remember where to return and continue at addr. Calls nest. | unchanged | 1,088 with its `ret` |
| `ret` | `1111 0000 0000 0000` | Return to just after the `call`. | unchanged | |
| `non` | `0000 0000 0000 0000` | Stops the CPU (Breakpoint). | | |

The register bits `dddddd`, `ssssss` and `xxxxxx` are, from the left: `rgAH`, `rgA`, `rgBH`, `rgB`, then two for the program counter and the stack, which programs leave alone. The programs build `load` and `store` with the two macros from day 1:

```
macro  load  { ld  | a? | (b? >> 6) }     # load X Y: the register X, the address register Y
macro  store { sto | a? | (b? >> 6) }
```

### Jump conditions

| Write | Bit | Jumps when |
|---|---|---|
| `zero` | `1000 0000 0000` | the last `sum` or `sub` gave 0 |
| `carry` | `0100 0000 0000` | the last `sum` carried out, or the last `sub` did not borrow (A >= B) |
| `nega` | `0010 0000 0000` | the last `sum` or `sub` gave a result with bit 31 set (after `sub`: A < B) |
| `eq` | `0001 0000 0000` | A equals B right now, all 32 bits; no `sub` needed |
| `neq` | `0000 1000 0000` | A differs from B right now |

### The idioms every program is built from

`v` is a variable: a `pointer` with an address below `0x1000`.

| To do this | Write | Instructions |
|---|---|---|
| A = variable v | `lia v; load rgA rgA` | 2 |
| B = variable v | `lib v; load rgB rgB` | 2 |
| variable v = A | `lib v; store rgA rgB` | 2 |
| v = v + 1 | `lia v; load rgA rgA; lib 1; sum; lib v; store rgA rgB` | 6 |
| B = A (lower half) | `psh rgA; pop rgB` | 2 |
| A = A + A | `lib 0; psh rgA; pop rgB; sum` (for A below 65,536) | 4 |
| A = the word a pointer variable p points at | `lia 0; lib p; load rgB rgB; load rgA rgB` | 4 |
| the word p points at = A | `lib p; load rgB rgB; store rgA rgB` | 3 |
| if a == b: goto L | `lia a; load rgA rgA; lib b; load rgB rgB; jmp eq; L` | 5 |
| if a < b: goto L | `lia a; load rgA rgA; lib b; load rgB rgB; sub; jmp nega; L` | 6 |
| if a >= b: goto L | the same with `jmp carry; L` | 6 |
| goto the address in A | `psh rgA; pop rgSP` (`rgSP` is the program counter) | 2 |
| add two numbers wider than 32 bits | word by word from the lowest; after each `sum` the carry is in the upper half: `psh rgAH; pop rgB` moves it into B for the next word (see day 2) | |

### Macros for the common operations

A macro body may hold several instructions separated by `;` (the game's compact sample computer does it, and the engine's assembler follows). These cover the rows of the table above that make up most of every program. `v` is a variable, `p` a variable that holds an address, `L` a label.

```
macro getA   { lia a?; ld | rgA | (rgA >> 6) }
macro getB   { lib a?; ld | rgB | (rgB >> 6) }
macro putA   { lib a?; sto | rgA | (rgB >> 6) }
macro getAat { lia a?; ld | rgA | (rgA >> 6); ld | rgA | (rgA >> 6) }
macro putAat { lib a?; ld | rgB | (rgB >> 6); sto | rgA | (rgB >> 6) }
macro inc    { lia a?; ld | rgA | (rgA >> 6); lib 1; sum; lib a?; sto | rgA | (rgB >> 6) }
macro copyAB { psh rgA; pop rgB }
macro goto   { jmp; a? }
macro if_eq  { jmp eq; a? }
macro if_neq { jmp neq; a? }
macro if_lt  { sub; jmp nega; a? }
macro if_ge  { sub; jmp carry; a? }
macro do     { call; a? }
```

| Write | Meaning | Instructions |
|---|---|---|
| `getA v` | A = v | 2 |
| `getB v` | B = v | 2 |
| `putA v` | v = A (B is used up) | 2 |
| `getAat p` | A = the word p points at | 3 |
| `putAat p` | the word p points at = A (B is used up) | 3 |
| `inc v` | v = v + 1 (A and B are used up) | 6 |
| `copyAB` | B = A (lower half) | 2 |
| `goto L` | jump to L | 2 |
| `if_eq L`, `if_neq L` | jump to L when A == B, A != B | 2 |
| `if_lt L`, `if_ge L` | jump to L when A < B, A >= B (A becomes A - B) | 3 |
| `do L` | call the subroutine L | 2 |
| `if_zero L`, `if_carry L`, `if_nega L` | jump to L on the flag the last `sum` or `sub` left (`sub` on its own row, then `if_nega L`: A < B) | 2 |
| `carryB` | B = the upper half of A, the carry of the last `sum` (`psh rgAH; pop rgB`) | 2 |
