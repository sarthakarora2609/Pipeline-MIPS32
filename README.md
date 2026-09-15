# Pipelined MIPS32 Processor (Verilog)

A 32-bit MIPS processor built in Verilog, using a classic **5-stage pipeline**:
Fetch → Decode → Execute → Memory → Write-back.

## What this project is

This is a hardware description of a simplified MIPS CPU. It's
written at a fairly low, structural level - pipeline registers, muxes, and
logic gates are wired together explicitly, the way you'd draw them on a
processor datapath diagram in a computer-architecture course. It's meant for
learning/simulating how a pipelined CPU actually works, not as a
production-ready chip design.

## What it can do

The processor understands a small set of MIPS instructions:

- **R-type ALU ops** (add, sub, and, or, etc. - via the ALU control unit)
- `lw` - load word from memory
- `sw` - store word to memory
- `bne` - branch if not equal
- `xori` - XOR immediate
- `j` - jump
- `jr` - jump register

Since instructions in a pipeline overlap in time, this design also handles
the three classic pipeline hazard problems:

- **Data forwarding** - if an instruction needs a result that a previous,
  still-in-flight instruction hasn't written back yet, the value is
  forwarded directly instead of waiting.
- **Stalling** - if forwarding alone can't solve it (e.g. right after a
  `lw`), the pipeline is paused for a cycle until the data is ready.
- **Flushing** - when a branch or jump changes the program's direction, the
  wrongly-fetched instructions behind it are discarded.

## Folder contents

```
verilog code/   All the Verilog modules that make up the CPU
Test_Bench_mips_pipeline.v   Testbench that drives the CPU with a clock and reset
instr.txt       The program (machine code, in binary) the CPU runs
```

### Key modules (in `verilog code/`)

| File | What it does |
|---|---|
| `Mips_Pipeline_top_level.v` | Top-level module - wires all five pipeline stages together |
| `Control_Unit.v` | Decodes the opcode into control signals |
| `ALU_32bit.v`, `ALU_Control_Unit.v` | The arithmetic/logic unit and its control |
| `Register_file.v` | The CPU's 32 general-purpose registers |
| `instruction_memory_unit.v` | Holds the program, loaded from `instr.txt` |
| `Data_Memory_Unit.v` | Read/write data memory |
| `Forwarding_Unit.v`, `WB_Forwarding_Unit.v` | Forward results to avoid stalling when possible |
| `Stall_Control_Units.v` | Detects load-use hazards and stalls the pipeline |
| `Flush_Constr_Unit.v` | Flushes wrongly-fetched instructions on a taken branch/jump |
| `JR_Control_Unit.v` | Handles the `jr` (jump register) instruction |
| `Adder_32bit.v`, `Multiplexer_*.v`, `Zero_extention.v`, `signExtension_Shiftbytwo.v` | Small building-block components (adders, muxes, sign/zero extension) used throughout the datapath |

## How to simulate it

The testbench just applies a clock and a reset pulse - the program in
`instr.txt` runs on its own from there. With Icarus Verilog, run this from
the project's top folder (so `instr.txt` is found where the simulator looks
for it):

```sh
iverilog -o mips_sim "verilog code"/*.v Test_Bench_mips_pipeline.v
vvp mips_sim
```

Then inspect register/memory values with a waveform viewer or `$display`
statements to confirm the program executed as expected.

## Status

A learning/simulation project - it has not been run through synthesis or
tested on real hardware.
