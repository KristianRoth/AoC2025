# Rules for solving Advent of Code on the CPU

Standing rules for every day, agreed on 2026-11-17. Day 2 part 1 is the first puzzle solved under them.

## The rules
1. **Input.** A host step may change the representation of the puzzle input and nothing else: numbers become binary data words in the program file (for numbers wider than a word, 16-bit words, low word first), with a terminator. No sorting, filtering or other processing outside the CPU. The step is throwaway work and is not kept in the repository.
2. **The CPU's job.** Every decision and every sum of the puzzle happens in the CPU's program. Constants that do not depend on the input (powers of ten, for example) may be in the program as data.
3. **Algorithm and run time.** The program should use an efficient algorithm. Tens of minutes of engine time is fine; days are not.
4. **Hardware.** The CPU as it is in `cpu-32bit-manual-start.vcb` and `cpu-32bit-autorun-aoc1p1.vcb` (with the owner's fixes of 2026-11-17 and 2026-11-18 for the shift, xor and neg). Only the program changes. A real bug in the CPU is reported to the owner, who decides. Changes may be suggested as an exercise; expect them to be rejected.
5. **Result.** The run ends on the Breakpoint. The answer is left in memory at fixed addresses in a format the program's header documents: binary words, decimal digits one per word, or both.
6. **Checking.** Any work or checking outside the CPU is allowed: test programs in the engine, the puzzle's example, a reference computation on the host. A wrong answer is fixed in the program.
7. **Generality.** A program must work for the owner's input and the puzzle's example. Limits taken from those (digit counts, number of entries) may be built in, and are stated in the program's header.
8. **Findings.** What is learned about the CPU by test programs goes into `findings/`, so later programs can start from it.
9. **Deliverables per part.** `days/dayN/partM.vcbasm` in the style of day 1, usable in the game's Assembly Editor as it is (so nothing follows a macro definition on its line), using the README's macros for the common operations, written as "How a solution is written" below says: an explanation and the whole program in pseudo code at the top, pseudo code before each section, and a comment on the code rows; a row in the README's list of solutions (answer, ticks, time); `days/dayN/partM-example.vcbasm` with the puzzle's example as input; a local commit in this repository. Never pushed by the assistant.
10. **Two parts.** After part 1 the answer is reported and work stops. The owner submits it and pastes part 2, which is solved under the same rules.

## How a program is run
From `engine/` in the VirtualCircuitBoardCPU repository:

```
./target/release/vcb run ../AoC2025/cpu-32bit-autorun-aoc1p1.vcb --asm ../AoC2025/days/dayN/partM.vcbasm --ticks <limit> --watch <result addresses>
```

## Decision log
- Input form → numbers as binary words; the CPU does no text parsing.
- Who computes → only encoding outside the CPU.
- Run time → an efficient algorithm; tens of minutes is acceptable.
- Hardware → as built; suggestions welcome as an exercise, most likely rejected.
- Result → in memory in a well-defined format; binary words or one decimal digit per word are both fine.
- Checking → anything outside the CPU is allowed.
- Generality → the owner's input and the example.
- Deliverables → the program, the example program, the README rows, a local commit. No encoding script in the repository.
- CPU behaviour → the assistant may run any test program and writes its findings to a folder here.
- Part 2 → the owner submits part 1 and pastes part 2.

## How a solution is written

Every solution from day 2 on follows the same form, so it can be read without running it. (The day 1 programs came first and are shorter on comments.)

- **An explanation at the top of the file:** the puzzle in two sentences, the idea the solution rests on with a worked example, where the answer is left in memory, how the input is encoded, the built-in limits, and a memory map.
- **The whole program in pseudo code**, about twenty lines, in the same header.
- **Pseudo code before each section** of the assembly (each loop, each subroutine), saying what that block does in terms of the variables.
- **A comment on the code rows**, at the end of the line, in terms of the variables: `# SRC = SRC + 1`, `# word 1: A = carry + C_1 + S_1`, `# HI < C: past the end of the range`. Table and data rows say what the words are in decimal.
- **Names over numbers:** every variable is a `pointer` with a name, every jump target a label.
- **Macros for the common operations** (the programs of days 2 to 4 were rewritten with them on 2026-11-18, storing the same words as before; day 1 is the owner's own and stays as it is): a program defines the ones it uses from [Macros for the common operations](README.md#macros-for-the-common-operations) after its instruction symbols and uses them wherever the operation occurs, so a row reads `getA ROW; getB GRID; if_lt DONE` and not ten instructions. An operation the program itself repeats often gets its own macro in the same way.

A comment explains what the line means for the program, not what the instruction does; what the instructions do is in [`findings/cpu-instructions.md`](findings/cpu-instructions.md).
