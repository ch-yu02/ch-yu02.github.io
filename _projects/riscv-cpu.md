---
layout: page
title: Five-Stage Pipelined RISC-V CPU
description: A Verilog implementation of a 32-bit five-stage RISC-V processor with forwarding, hazard detection, branch flushing, and subword memory operations.
img: assets/img/projects/riscv-cpu/riscv-cpu-architecture.png
importance: 1
category: work
_styles: |
  article {
    font-size: 1.1rem;
    line-height: 1.75;
  }

  article blockquote {
    font-size: inherit;
  }
---

**August 2026 · Computer Organization and Design, Zhejiang University · Verilog · ModelSim · Vivado · Nexys Video**

I implemented and integrated a 32-bit, five-stage pipelined RISC-V processor in Verilog. The design executes the course-required RV32I subset through the standard **IF–ID–EX–MEM–WB** pipeline and handles the main hazards introduced by pipelining: data forwarding, load-use stalls, and control-flow flushing. I also extended the memory datapath to support byte and halfword load/store operations.

[Source code on GitHub](https://github.com/ch-yu02/My-ISEE-Course-Reference/tree/main/2526%E7%A7%8B%E5%86%AC/ComputerOrganization/experiment/lab30_Risc5CPU)

<div style="width: 84%; margin-inline: auto;">
{% include figure.liquid loading="eager" path="assets/img/projects/riscv-cpu/riscv-cpu-architecture.png" title="Five-stage RISC-V CPU architecture" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  Five-stage datapath and control paths.
</div>

> **Pipeline:** IF → ID → EX → MEM → WB

## Processor architecture

The processor is divided into five stages separated by four pipeline registers:

- **IF** maintains the program counter, fetches instructions, and selects between sequential execution and a branch/jump target.
- **ID** decodes instructions, reads the register file, generates immediates, evaluates branches, and detects load-use hazards.
- **EX** performs ALU operations and selects forwarded operands when dependencies are present.
- **MEM** accesses data memory.
- **WB** selects either the ALU result or memory data and writes it back to the register file.

The base design supports the arithmetic, logic, shift, conditional branch, `jal`/`jalr`, `lw`/`sw`, `lui`, `auipc`, and `nop` operations required by the lab. Pipeline registers carry both data and control signals between stages, while dedicated `Stall`, `IFWrite`, and `Flush` paths alter the normal flow when an instruction cannot advance safely.

## Pipeline hazard handling

The main design challenge was preserving correctness across dependent instructions without reducing the pipeline to sequential execution.

For ordinary RAW dependencies, the **EX-stage forwarding unit** compares the current source registers with destination registers in the MEM and WB stages. The newest available value takes priority: either `ALUResult_mem` or `RegWriteData_wb` is routed directly to the ALU inputs instead of waiting for register-file writeback. Store data uses the same forwarding path, so a dependent `sw` can also receive the latest value.

A second bypass path sits inside the register file. If WB writes a register in the same cycle that ID reads it, the newly written value is returned directly instead of the stale stored value.

A **load-use dependency** requires a different response because the loaded data is not yet available when the following instruction reaches the next stage. The hazard detector therefore freezes the PC and IF/ID register for one cycle and clears ID/EX, inserting a bubble. Branch and jump targets are resolved early in ID; when control flow changes, the prefetched instruction in IF/ID is flushed.

## Subword memory operations

After completing the base processor, I extended the memory datapath to support **`lb`, `lh`, `lbu`, `lhu`, `sb`, and `sh`** in addition to word accesses.

The extension carries `funct3` into MEM and uses the low two address bits as a byte offset. Stores generate a four-bit byte-write mask and shift source data into the appropriate byte lanes. Loads select the requested byte or halfword from the 32-bit RAM output and apply sign or zero extension before writeback. This keeps the overall pipeline unchanged while localizing alignment and width handling to the memory stage.

## Verification

I wrote module-level testbenches for the ALU, decoder, and instruction-fetch logic, then verified the complete processor with a top-level test program designed to exercise interactions between pipeline stages. The sequence includes jumps and branches, consecutive register dependencies, memory accesses, and an explicit `lw` followed by a dependent shift to trigger the load-use stall.

<div style="width: 86%; margin-inline: auto;">
{% include figure.liquid path="assets/img/projects/riscv-cpu/cpu-simulation-waveform.png" title="Top-level CPU simulation" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  Top-level ModelSim verification.
</div>

The waveform matched the expected instruction sequence. Forwarding supplied results before they reached WB, branch and jump decisions redirected the PC correctly, and the load-use case asserted `Stall`, held the PC for one cycle, and then resumed with the correct operand.

The completed design was integrated into the provided Vivado project and tested on a **Nexys Video FPGA board** using the course's single-step observation interface.

This project made pipeline timing concrete: the difficult part was not simply determining _what_ an instruction computes, but tracking _when_ each value becomes available and routing it correctly while several instructions are in flight.
