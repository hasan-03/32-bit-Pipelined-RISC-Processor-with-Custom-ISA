# 32-bit Pipelined RISC Processor

This project presents the design and implementation of a **32-bit 5-Stage Pipelined Processor** developed in Verilog using the Xilinx Vivado design suite. Unlike standard single-cycle architectures, this design utilizes a deep pipeline to increase instruction throughput and efficiency. It features a custom **Instruction Set Architecture (ISA)** that supports arithmetic, logic, memory access, and complex control flow including loops and conditional branching.

To handle the complexities of pipelining, the processor includes dedicated hardware for **Hazard Detection** (to prevent data corruption) and **Data Forwarding** (to eliminate unnecessary stalls).

## Processor Modules

1. **inst_memory & data_memory**:
   The design uses a Harvard Architecture approach with separate memory modules for Instructions and Data. This allows simultaneous fetching of code and reading/writing of data, which is essential for pipelined execution.

2. **reg_bank**:
   A high-speed register file containing general-purpose registers (R0-R255). It features "Write-First" logic (writing on the negative clock edge) to resolve Write-After-Write hazards automatically within a single cycle.

3. **ALU (Arithmetic Logic Unit)**:
   A combinational circuit responsible for arithmetic and logic execution. It supports standard operations including:
   - **Arithmetic:** Addition, Subtraction, Multiplication.
   - **Control:** Decrement (optimized for Loop Counters).
   - **Logic:** Zero-flag generation for Branch comparisons.

4. **forwarding_unit**:
   A specialized unit that "short-circuits" the pipeline. It detects if an instruction needs data that is still moving through the pipeline (in Execute or Memory stages) and forwards it immediately to the ALU, skipping the need to wait for a write-back.

5. **hazard_unit**:
   The "traffic cop" of the processor. It detects dangerous conditions (like `Load-Use` dependencies) and inserts "bubbles" (stalls) or flushes the pipeline (on Branch Taken) to prevent errors.

6. **processor**:
   The top-level module that connects all stages (IF, ID, EX, MEM, WB). It manages the pipeline registers (`IF/ID`, `ID/EX`, `EX/MEM`, `MEM/WB`) that act as buffers between clock cycles.

## How Instructions are Executed

In this pipelined architecture, up to five instructions are active simultaneously, each in a different stage of execution:

1. **Fetch (IF)**: The **Program Counter (PC)** fetches the next instruction from Instruction Memory. If a Branch is taken, the PC is updated immediately, and the incorrect instruction in the pipeline is "flushed" (deleted).
2. **Decode (ID)**: The instruction is broken down. The **Hazard Unit** checks for conflicts. If a `Load-Use` conflict is found, the pipeline "stalls" (pauses) here.
   - *Optimization:* Branch Immediate offsets are read from bits `[23:16]` to allow larger register address space.
3. **Execute (EX)**: The ALU performs the calculation. The **Forwarding Unit** checks if the operands are available from previous instructions (Forwarding from MEM or WB) and bypasses the Register File if necessary.
4. **Memory (MEM)**: Used for `LOAD` and `STORE` instructions to access the Data Memory.
5. **Write-Back (WB)**: The final result is written back to the Register Bank.

## Instruction Set Architecture (ISA) - Opcode Table

The processor uses a fixed 32-bit instruction format.

| Mnemonic | Opcode (Hex) | Description |
|----------|--------------|----------------------------------------|
| `STORE`  | `0x01`       | Store register value to memory (`MEM[Rs1+Imm] = Rs2`) |
| `LOAD`   | `0x02`       | Load value from memory to register (`Rd = MEM[Rs1+Imm]`) |
| `ADD`    | `0x07`       | Add two registers (`Rd = Rs1 + Rs2`) |
| `SUB`    | `0x08`       | Subtract two registers (`Rd = Rs1 - Rs2`) |
| `MUL`    | `0x09`       | Multiply two registers (`Rd = Rs1 * Rs2`) |
| `DEC`    | `0x18`       | Decrement value (`Rd = Rs1 - 1`) |
| `HALT`   | `0x24`       | Halt the processor |
| `JMP`    | `0x25`       | Unconditional Jump (`PC = PC + 1 + Offset`) |
| `BEQ`    | `0x26`       | Branch if Equal (`if Rs1 == Rs2 Jump`) |
| `BNE`    | `0x27`       | Branch if Not Equal (`if Rs1 != Rs2 Jump`) |

## Performance & Clock Frequency

- **Architecture:** 5-Stage Pipeline
- **Hazard Resolution:** Hardware Forwarding & Stalling
- **Simulated Frequency:** 100 MHz (Configurable to 1 GHz+ in simulation testbench)
- **Throughput:** 1 Instruction Per Cycle (IPC) ideally.
