# Advent of Code 2025 on the 32-bit CPU

The CPU's board files for the Steam game Virtual Circuit Board (Update 1.0.1), and the programs written for it.

| File | What it is |
|---|---|
| `cpu-32bit-manual-start.vcb` | The CPU as built. Its run switch (a 5×5 latch block near x 1100, y 249) is Latch Off, so it starts when the latch is clicked in the game. Formerly `32-bit-alu-try.vcb`. |
| `cpu-32bit-autorun-aoc1p1.vcb` | The same CPU with the run switch drawn as Latch On, so it runs by itself from the first tick. Its assembly is `days/day1/part1.vcbasm` (Advent of Code day 1, part 1), and the program ends on a Breakpoint. Formerly `CPU.vcb`. |
| `days/day2/cpu-32bit-autorun-aoc2p1.vcb` | The autorun CPU with `days/day2/part1.vcbasm` as its assembly, for checking day 2 part 1 in the game. Persistent Memory is set to `0x0100`–`0x011E`, where the answer is left. |

The boards differ only in those 24 latch pixels, in the assembly text they carry, and in the Persistent Memory range.

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
