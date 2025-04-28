# MTH410E – RISC-V Architecture and Processor Design

## Multi Cycle Pipelined RISC-V Processor

Write a SystemVerilog code of a multi cycle pipelined RISC-V processor. The designed micro-architecture should include all the instructions in RV32I (It is in the [riscv-spec](https://github.com/riscv/riscv-isa-manual/releases/download/riscv-isa-release-bb8b912-2025-03-21/riscv-unprivileged.pdf)). The processor should work as in-order and single-issue. There is no need for FENCE, FENCE.TSO, PAUSE, ECALL, EBREAK instruction implementation. Additional to the RV32I, [CTZ](https://riscv-software-src.github.io/riscv-unified-db/manual/html/isa/isa_20240411/insts/ctz.html), [CLZ](https://riscv-software-src.github.io/riscv-unified-db/manual/html/isa/isa_20240411/insts/clz.html), and [CPOP](https://riscv-software-src.github.io/riscv-unified-db/manual/html/isa/isa_20240411/insts/cpop.html) must be implemented too.

**Note: This homework is same with the previous one. The only difference is being multi-cycle and pipelined architecture.**

The top file of the processor should be as follows;

```
module riscv_multicycle
  import riscv_pkg::*;
(
    parameter DMemInitFile  = “dmem.mem”;       // data memory initialization file
    parameter IMemInitFile  = “imem.mem”;       // instruction memory initialization file
)   (
    input  logic             clk_i,       // system clock
    input  logic             rstn_i,      // system reset
    input  logic  [XLEN-1:0] addr_i,      // memory adddres input for reading
    output logic  [XLEN-1:0] data_o,      // memory data output for reading
    output logic             update_o,    // retire signal
    output logic  [XLEN-1:0] pc_o,        // retired program counter
    output logic  [XLEN-1:0] instr_o,     // retired instruction
    output logic  [     4:0] reg_addr_o,  // retired register address
    output logic  [XLEN-1:0] reg_data_o,  // retired register data
    output logic  [XLEN-1:0] mem_addr_o,  // retired memory address
    output logic  [XLEN-1:0] mem_data_o,  // retired memory data
    output logic             mem_wrt_o    // retired memory write enable signal
);
  // module body
  // use other modules according to the need.

endmodule
```

The data and instruction memories of the processor should be initialized by DMemInitFile and  IMemInitFile respectively (i.e., use $readmemh in the initial block of the memories).

All the addresses should be based on `0x8000_0000` you can use offset to get for example `0x8000` space after `0x8000_0000`. In other words, you address space for both IMEM and DMEM is from `0x8000_0000` to `0x8000_FFFF` (if you require more space for your implementation you can increase the ending address).

To trace the processor state, you can use the print parts in given testbench.


Some resources:
- [RISC-V: An Overview of the Instruction Set Architecture](https://web.cecs.pdx.edu/~harry/riscv/RISCV-Summary.pdf)
- [RISC-V Specification](https://github.com/riscv/riscv-isa-manual/releases/download/riscv-isa-release-bb8b912-2025-03-21/riscv-privileged.pdf)


Tasks:

1. Complete your RTL in SystemVerilog.
2. Clean all lint problems.
3. Run the example in the test folder.
4. Find other test and check (there will be other test cases to check).
5. Create a PC table for each cycle. state flush and stall states. Example is below. Assume `0x80000010` is a mispredicted branch jumping to `0x80001000`. Name you txt as `pipe.log` and the directory is the directory of `model.log`.

|    | F| D| E|M |WB|
|--  |--|--|--|--|--|
| 1  |0x80000000| - | - | - | - |
| 2  |0x80000004|0x80000000| - | - | - |
| 3  |0x80000008|0x80000004| 0x80000000 | - | - |
| 4  |0x8000000c|0x80000008|0x80000004| 0x80000000 | - |
| 5  |0x80000010|0x8000000c|0x80000008|0x80000004|0x80000000|
| 6  |0x80000014|0x80000010|0x8000000c|0x80000008|0x80000004|
| 7  |0x80000018|0x80000014|0x80000010|0x8000000c|0x80000008|
| 8  |0x80001000|Flushed|Flushed|0x80000010|0x8000000c|
| 9  |0x80001004|0x80001000|Flushed|Flushed|0x80000010|
| 10 |0x80001008|0x80001004|0x80001000|Flushed|Flushed|

