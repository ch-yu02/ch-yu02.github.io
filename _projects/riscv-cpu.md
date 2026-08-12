---
layout: page
title: Five-Stage Pipelined RISC-V CPU
description: A Verilog implementation of a 32-bit five-stage RISC-V processor with forwarding, hazard detection, branch flushing, and subword memory operations.
img: assets/img/projects/riscv-cpu/riscv-cpu-architecture.png
importance: 1
category: work
---

**August 2026 · Computer Organization and Design, Zhejiang University · Verilog / ModelSim / Vivado / Nexys Video**

I implemented and integrated a 32-bit, five-stage pipelined RISC-V processor in Verilog. The design executes the course-required RV32I instruction subset across the standard **IF–ID–EX–MEM–WB** pipeline and handles the main timing problems introduced by pipelining: data forwarding, load-use stalls, and control-hazard flushing. I also extended the memory path to support byte and halfword load/store instructions.

[Source code on GitHub](https://github.com/ch-yu02/My-ISEE-Course-Reference/tree/main/2526%E7%A7%8B%E5%86%AC/ComputerOrganization/experiment/lab30_Risc5CPU)

{% include figure.liquid loading="eager" path="assets/img/projects/riscv-cpu/riscv-cpu-architecture.png" title="Five-stage RISC-V CPU architecture" class="img-fluid rounded z-depth-1" %}

<div class="caption">
  Overall datapath and pipeline-control structure.
</div>

## Processor architecture

The processor is organized into five stages separated by four pipeline registers:

- **IF** maintains the program counter, fetches instructions, and selects between sequential and branch/jump targets.
- **ID** decodes instructions, reads the register file, generates immediates, evaluates branches, and detects load-use hazards.
- **EX** performs ALU operations and selects forwarded operands when dependencies exist.
- **MEM** accesses data memory.
- **WB** selects the ALU or memory result and writes it back to the register file.

The base design supports arithmetic, logic, shift, conditional branch, `jal`/`jalr`, `lw`/`sw`, `lui`, `auipc`, and `nop` operations required by the lab. Pipeline registers carry both data and control signals between stages, while dedicated `Stall`, `IFWrite`, and `Flush` paths modify the normal flow when an instruction cannot safely advance.

## Pipeline hazard handling

The main design work was making dependent instructions execute correctly without turning the pipeline into a sequential processor.

For ordinary RAW dependencies, the **EX-stage forwarding unit** compares the current source registers against destination registers in the MEM and WB stages. The newest available value has priority, and either `ALUResult_mem` or `RegWriteData_wb` is routed directly to the ALU inputs instead of waiting for register-file writeback. Store data passes through the same forwarding path, so a dependent `sw` can also receive the latest value.

A second bypass path is implemented inside the register file. If the WB stage writes a register in the same cycle that ID reads it, the new write data is returned directly rather than the stale stored value.

A **load-use dependency** is different because the loaded data is not yet available when the following instruction reaches ID/EX. The hazard detector therefore freezes the PC and IF/ID register and clears ID/EX for one cycle, inserting a bubble. For branches and jumps, the target is resolved early in ID; when control flow changes, the prefetched instruction in IF/ID is flushed.

## Subword memory operations

After completing the base processor, I extended the memory datapath to support **`lb`, `lh`, `lbu`, `lhu`, `sb`, and `sh`** in addition to word accesses.

The extension carries `funct3` into MEM and uses the low two address bits as a byte offset. Stores generate a four-bit byte-write mask and shift the source data into the correct byte lanes. Loads select the requested byte or halfword from the 32-bit RAM output and apply either sign extension or zero extension before writeback. This keeps the main pipeline structure unchanged while moving alignment and width handling into the memory stage.

## Verification

I wrote module-level testbenches for the ALU, decoder, and instruction-fetch logic, followed by a top-level CPU test program designed to exercise the interaction between pipeline stages. The program includes jumps and branches, consecutive register dependencies, memory accesses, and an explicit `lw` followed by a dependent shift instruction to trigger the load-use stall.

{% include figure.liquid path="assets/img/projects/riscv-cpu/cpu-simulation-waveform.png" title="Top-level CPU simulation waveform" class="img-fluid rounded z-depth-1" %}

<div class="caption">
  ModelSim top-level simulation. The test sequence exercises jumps, branches, forwarding, memory access, and a load-use stall.
</div>

The top-level waveform matched the expected instruction sequence: forwarding supplied results that had not yet reached WB, branch and jump decisions redirected the PC correctly, and the load-use case asserted `Stall`, held the PC for one cycle, and resumed with the correct operand. The completed design was then integrated into the provided Vivado project and tested on a **Nexys Video FPGA board** using the course's single-step observation interface.

This project was a useful exercise in reasoning about **when a value becomes available**, not just what an instruction computes. Most of the implementation complexity came from the feedback paths and pipeline-control conditions needed to preserve architectural correctness while multiple instructions were in flight.
