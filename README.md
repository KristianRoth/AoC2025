# Advent of Code 2025 on the 32-bit CPU

The CPU's board files for the Steam game Virtual Circuit Board (Update 1.0.1), and the programs written for it.

| File | What it is |
|---|---|
| `cpu-32bit-manual-start.vcb` | The CPU as built. Its run switch (a 5×5 latch block near x 1100, y 249) is Latch Off, so it starts when the latch is clicked in the game. Formerly `32-bit-alu-try.vcb`. |
| `cpu-32bit-autorun-aoc1p1.vcb` | The same CPU with the run switch drawn as Latch On, so it runs by itself from the first tick. Its assembly is `days/day1/part1.vcbasm` (Advent of Code day 1, part 1), and the program ends on a Breakpoint. Formerly `CPU.vcb`. |
| `days/day2/cpu-32bit-autorun-aoc2p1.vcb` | The autorun CPU with `days/day2/part1.vcbasm` as its assembly, for checking day 2 part 1 in the game. Persistent Memory is set to `0x0100`–`0x011E`, where the answer is left. |
| `days/day2/cpu-32bit-autorun-aoc2p2.vcb` | The same for `days/day2/part2.vcbasm`. |
| `days/day3/cpu-32bit-autorun-aoc3p1.vcb` | The same for `days/day3/part1.vcbasm`; Persistent Memory `0x0100`–`0x0114`. |

The boards differ only in those 24 latch pixels, in the assembly text they carry, and in the Persistent Memory range.

All five carry two fixes that bring instruction bits to the ALU: the shift fix of 2026-11-17 (50 pixels near x 1120–1129, y 215–224; `shf | n` shifts A left by n) and the xor and neg fix of 2026-11-18 (24 pixels just right of it; `xor` and `neg` work). The fixes and how they were found are in [`findings/cpu-instructions.md`](findings/cpu-instructions.md).

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

Notes on the day 3 solution:

- **One pass per bank.** Each of the 20,000 digits is read once; the program keeps the largest digit so far and the best pair so far, so nothing is compared pairwise. That is about 33 instructions per digit, most of them loading and storing the four variables.
- **One multiplication for the whole input.** The tens digits of all banks are summed separately and multiplied by 10 once at the end, by doubling.
- **Size:** 154 instructions; almost all of the 20,387 words are the input, one word per digit.

## Instruction set

What each instruction really does, as found by test programs, is in [`findings/cpu-instructions.md`](findings/cpu-instructions.md).

Each instruction is one 16-bit word; Jump and Call are followed by a 16-bit address word.

```
1. No-Op:     0000 xxxx xxxx xxxx
2.
3. Load:      0010 AaBbPB  AaBbPB
4. LoadIA:    0011 OOOO OOOO OOOO
5. LoadIB:    0100 OOOO OOOO OOOO
6. Store:     0101 AaBbPB  AaBbPB
7. Op:        0110 xSSS AOXN OOOO
8. 
9.
10.
11. Push:     1010 AaBb PBxx xxxx
12. Pop:      1011 AaBb PBxx xxxx
13. Jump:     1100 zcnq nxxx xxxx - AAAA AAAA AAAA AAAA
14. 
15. Call:     1110 xxxx xxxx xxxx - AAAA AAAA AAAA AAAA
16. Ret:      1111 xxxx xxxx xxxx
```
