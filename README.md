# Advent of Code 2025 on the 32-bit CPU

The CPU's board files for the Steam game Virtual Circuit Board (Update 1.0.1), and the programs written for it.

| File | What it is |
|---|---|
| `cpu-32bit-manual-start.vcb` | The CPU as built. Its run switch (a 5×5 latch block near x 1100, y 249) is Latch Off, so it starts when the latch is clicked in the game. Formerly `32-bit-alu-try.vcb`. |
| `cpu-32bit-autorun-aoc1p1.vcb` | The same CPU with the run switch drawn as Latch On, so it runs by itself from the first tick. Its assembly is `days/day1/part1.vcbasm` (Advent of Code day 1, part 1), and the program ends on a Breakpoint. Formerly `CPU.vcb`. |
| `days/day2/cpu-32bit-autorun-aoc2p1.vcb` | The autorun CPU with `days/day2/part1.vcbasm` as its assembly, for checking day 2 part 1 in the game. Persistent Memory is set to `0x0100`–`0x011E`, where the answer is left. |
| `days/day2/cpu-32bit-autorun-aoc2p2.vcb` | The same for `days/day2/part2.vcbasm`. |
| `days/day3/cpu-32bit-autorun-aoc3p1.vcb` | The same for `days/day3/part1.vcbasm`; Persistent Memory `0x0100`–`0x0114`. |
| `days/day3/cpu-32bit-autorun-aoc3p2.vcb` | The same for `days/day3/part2.vcbasm`; Persistent Memory `0x0110`–`0x011E`. |

The boards differ only in those 24 latch pixels, in the assembly text they carry, and in the Persistent Memory range.

All six carry two fixes that bring instruction bits to the ALU: the shift fix of 2026-11-17 (50 pixels near x 1120–1129, y 215–224; `shf | n` shifts A left by n) and the xor and neg fix of 2026-11-18 (24 pixels just right of it; `xor` and `neg` work). The fixes and how they were found are in [`findings/cpu-instructions.md`](findings/cpu-instructions.md).

These boards run in the game, and in the `vcb` engine of the VirtualCircuitBoardCPU repository, which reproduces the game tick for tick. That engine's fixture `engine/tests/boards/cpu-aoc1p1.vcb` is the autorun board with the program already assembled into its memory by the game; its README explains how to run a program.

## Programs

The rules the puzzles are solved under are in [`RULES.md`](RULES.md).

`days/` holds the assembly programs for the CPU, one folder per day, in the game's assembly language:

| File | Program |
|---|---|
| `days/day1/part1.vcbasm` | Advent of Code day 1, part 1 |
| `days/day1/part2.vcbasm` | Advent of Code day 1, part 2 |
| `days/day2/part1.vcbasm` | Advent of Code day 2, part 1. The header says where the answer is left |
| `days/day2/part1-example.vcbasm` | The same program with the puzzle's example as input (answer 1227775554) |
| `days/day2/part2.vcbasm` | Advent of Code day 2, part 2. The header says where the answer is left |
| `days/day2/part2-example.vcbasm` | The same program with the puzzle's example as input (answer 4174379265) |
| `days/day3/part1.vcbasm` | Advent of Code day 3, part 1. The header says where the answer is left |
| `days/day3/part1-example.vcbasm` | The same program with the puzzle's example as input (answer 357) |
| `days/day3/part2.vcbasm` | Advent of Code day 3, part 2. The header says where the answer is left |
| `days/day3/part2-example.vcbasm` | The same program with the puzzle's example as input (answer 3121910778619) |

## How a solution is written

Every solution from day 2 on follows the same form, so it can be read without running it. (The day 1 programs came first and are shorter on comments.)

- **An explanation at the top of the file:** the puzzle in two sentences, the idea the solution rests on with a worked example, where the answer is left in memory, how the input is encoded, the built-in limits, and a memory map.
- **The whole program in pseudo code**, about twenty lines, in the same header.
- **Pseudo code before each section** of the assembly (each loop, each subroutine), saying what that block does in terms of the variables.
- **A comment on the code rows**, at the end of the line, in terms of the variables: `# SRC = SRC + 1`, `# word 1: A = carry + C_1 + S_1`, `# HI < C: past the end of the range`. Table and data rows say what the words are in decimal.
- **Names over numbers:** every variable is a `pointer` with a name, every jump target a label.

A comment explains what the line means for the program, not what the instruction does; what the instructions do is in [`findings/cpu-instructions.md`](findings/cpu-instructions.md).

## Performance

Measured in the `vcb` engine on `cpu-32bit-autorun-aoc1p1.vcb`, to the Breakpoint. The tick count is the same in the game (the game's counter shows one more). The engine runs about 6 million ticks per second on the machine these were measured on.

| Program | Answer | Ticks | Engine time | Instructions run (about) | Program and data |
|---|---|---|---|---|---|
| Day 1 part 1 | 1031 | 43,146,512 | 7.4 s | 169,000 | 8,399 words |
| Day 1 part 2 | 5831 | 70,339,472 | 11.6 s | 275,000 | 8,427 words |
| Day 2 part 1, example | 1227775554 | 10,383,248 | 1.7 s | 41,000 | 633 words |
| Day 2 part 1 | 54641809925 | 43,495,312 | 7.1 s | 170,000 | 777 words |
| Day 2 part 2, example | 4174379265 | 19,838,992 | 3.3 s | 77,000 | 810 words |
| Day 2 part 2 | 73694270688 | 75,753,616 | 12.3 s | 296,000 | 954 words |
| Day 3 part 1, example | 357 | 682,000 | 0.1 s | 2,700 | 251 words |
| Day 3 part 1 | 16993 | 170,904,144 | 26.7 s | 668,000 | 20,387 words |
| Day 3 part 2, example | 3121910778619 | 1,384,016 | 0.2 s | 5,400 | 270 words |
| Day 3 part 2 | 168617068915447 | 419,669,712 | 65.0 s | 1,639,000 | 20,406 words |

"Instructions run" is the tick count divided by 256, the cost of most instructions:

| Instruction | Ticks |
|---|---|
| `lia`, `lib`, `load`, `store`, `sum`, `sub`, `and`, `or`, `xor`, `neg`, `shf` | 256 |
| `psh` and `pop` together | 576 |
| `jmp`, taken | 320 |
| `jmp` with a condition, not taken | 192 |
| `call` and `ret` together | 1,088 |

Notes on the day 2 solutions:

- **They never test an ID.** The input ranges hold about 2.6 million IDs. The programs generate only the invalid ones: about 1,700 are added in part 1. Checking every ID would take thousands of times longer on this CPU, which has no multiply or divide.
- **Where the time goes.** Reaching the first invalid ID of a range is done by walking up in decimal steps (`STEP_UP`), at most nine steps per decimal digit. Each step taken is one 48-bit addition, one comparison and one copy, about 60 instructions. Part 2 runs longer than part 1 because it has 13 families of repeated blocks per range in place of 5.
- **Most instructions only move data.** Loading a variable takes two instructions (`lia v; load rgA rgA`) and so does storing one. In part 2, 73% of the instructions in the code are such pairs or the carry move.
- **Code size:** part 1 has 355 instructions and part 2 has 419; the rest of the words are the tables and the input.

Notes on the day 3 solutions:

- **One pass per bank.** Each of the 20,000 digits is read once; the program keeps the largest digit so far and the best pair so far, so nothing is compared pairwise. That is about 33 instructions per digit, most of them loading and storing the four variables.
- **One multiplication for the whole input.** The tens digits of all banks are summed separately and multiplied by 10 once at the end, by doubling.
- **Size:** part 1 has 154 instructions; almost all of the 20,387 words are the input, one word per digit.
- **Part 2 keeps a stack.** Choosing 12 digits is dropping the rest: each digit goes on a stack after the smaller digits on top of it are thrown away, while drops are left. Every digit is pushed once and thrown away at most once, so a bank is still one pass (plus one pass to find its end), about 82 instructions per digit.
- **Part 2 never builds a big number.** The chosen digits are added column by column into the 15 result cells, and one round of carrying at the end turns the column sums into the decimal answer. So a 15-digit sum needs only 16-bit arithmetic, and the result exists in decimal only.
- **Part 2's size:** 172 instructions.

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

### Things that surprise

- **`lia` and `lib` clear the upper half** of their register. Load the upper half (`load rgAH …`) after the lower one, never before.
- **`lib` is needed for every store**, so storing uses up B: `lib v; store rgA rgB`.
- **After `and`, `or`, `xor` and `neg` the flags are wrong.** `zero` is 0 even when the result is 0. To test such a result, compare it: `lib 0; jmp eq; …`.
- **An address above `0x0FFF` cannot be written in an instruction** (`lia`/`lib` take 12 bits). Keep it in a variable and load it.
- **Naming two registers in one instruction does not do two things.** A load puts the same word in both; a store or push writes the OR of both.
- **`sum_32`, `sub_32` and the other `_32` symbols** set a bit the CPU ignores. They do the same as the plain ones; every operation is 32 bits wide.
- **No multiply, divide, right shift or add-with-carry.** Multiply by a power of two with `shf`, by a small constant with repeated `sum`.

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
| add two numbers wider than 32 bits | word by word from the lowest; after each `sum` the carry is in the upper half: `psh rgAH; pop rgB` moves it into B for the next word (see day 2) | |
