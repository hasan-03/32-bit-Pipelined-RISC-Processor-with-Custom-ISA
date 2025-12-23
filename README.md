# 32-bit 5-Stage Pipelined RISC Processor

This repository contains the Verilog implementation of a **32-bit Pipelined Processor** designed using a Harvard Architecture. The processor utilizes a 5-stage pipeline to maximize instruction throughput and includes advanced features such as **Data Forwarding** and **Hazard Detection** to handle dependencies and control flow efficiently.

## Design Architecture

The processor is divided into five distinct pipeline stages:

1.  **Fetch (IF):** Retrieves the next instruction from the Instruction Memory and handles PC increments.
2.  **Decode (ID):** Decodes the instruction, reads from the Register Bank (Write-First config), and calculates immediate values. The **Hazard Unit** operates here to insert stalls for Load-Use dependencies or flush the pipeline for branches.
3.  **Execute (EX):** The ALU performs arithmetic/logic operations. The **Forwarding Unit** operates here to bypass the register file and forward data from MEM or WB stages directly to the ALU inputs, solving data hazards.
4.  **Memory (MEM):** Accesses Data Memory for `LOAD` and `STORE` operations.
5.  **Writeback (WB):** Writes the final result back into the General Purpose Registers.

### Key Modules
* **Forwarding Unit:** Solves data hazards by forwarding results from the end of the EX or MEM stages to the start of the EX stage, preventing unnecessary stalls.
* **Hazard Unit:** Detects `Load-Use` hazards (stalls pipeline) and Control Hazards (flushes pipeline on taken branches).
* **ALU:** Handles arithmetic, logic, shifts, and generates the `Zero` flag used for branch decisions.

---

## Instruction Set Architecture (ISA)

The processor uses a fixed 32-bit instruction format.

### Instruction Format
Depending on the operation, the 32 bits are interpreted as follows:

| Type | Bits [31:24] | Bits [23:16] | Bits [15:8] | Bits [7:0] |
| :--- | :---: | :---: | :---: | :---: |
| **ALU / Register** | Opcode | Dest Reg (Rd) | Source 1 (Rs1) | Source 2 (Rs2) |
| **LOAD** | `0x02` | *Ignored* | Base (Rs1) | Dest Reg (Rd) |
| **STORE** | `0x01` | *Ignored* | Base (Rs1) | Source (Rs2) |
| **BRANCH** | Opcode | **Offset (Imm)** | Compare 1 (Rs1) | Compare 2 (Rs2) |

### Opcode Table

| Mnemonic | Opcode (Hex) | Operation | Description |
| :--- | :---: | :--- | :--- |
| **Memory** | | | |
| `STORE` | `0x01` | `MEM[Rs1 + Imm] = Rs2` | Store Register to Memory |
| `LOAD` | `0x02` | `Rd = MEM[Rs1 + Imm]` | Load Memory to Register |
| **Arithmetic** | | | |
| `ADD` | `0x07` | `Rd = Rs1 + Rs2` | Addition |
| `SUB` | `0x08` | `Rd = Rs1 - Rs2` | Subtraction |
| `MUL` | `0x09` | `Rd = Rs1 * Rs2` | Multiplication |
| `DIV` | `0x0A` | `Rd = Rs1 / Rs2` | Division |
| `INC` | `0x17` | `Rd = Rs1 + 1` | Increment |
| `DEC` | `0x18` | `Rd = Rs1 - 1` | Decrement |
| **Logic** | | | |
| `AND` | `0x0B` | `Rd = Rs1 & Rs2` | Bitwise AND |
| `OR` | `0x0C` | `Rd = Rs1 \| Rs2` | Bitwise OR |
| `XOR` | `0x0D` | `Rd = Rs1 ^ Rs2` | Bitwise XOR |
| `NOR` | `0x0E` | `Rd = ~(Rs1 \| Rs2)` | Bitwise NOR |
| `NAND` | `0x0F` | `Rd = ~(Rs1 & Rs2)` | Bitwise NAND |
| `NOT` | `0x16` | `Rd = ~Rs1` | Bitwise NOT |
| **Shifts** | | | |
| `SR` | `0x19` | `Rd = Rs1 >> 1` | Logical Shift Right |
| `SL` | `0x20` | `Rd = Rs1 << 1` | Logical Shift Left |
| **Control Flow** | | | |
| `JMP` | `0x25` | `PC = PC + 1 + Offset` | Unconditional Jump |
| `BEQ` | `0x26` | `if (Rs1 == Rs2) Jump` | Branch if Equal |
| `BNE` | `0x27` | `if (Rs1 != Rs2) Jump` | Branch if Not Equal |
| **System** | | | |
| `HALT` | `0x24` | `Stop` | Halts execution (in sim) |

---

## Branching Guide

This processor uses **PC-Relative Branching**. The branch logic is resolved in the **Execute (EX)** stage.

### How to use Branches (`BEQ` / `BNE`)
To write a branch instruction in hex code, you must calculate the **Offset**.
* **Formula:** `Target_Address = Current_PC + 1 + Offset`
* **Therefore:** `Offset = Target_Address - Current_PC - 1`

The Offset is an **8-bit signed integer** located in bits `[23:16]`.
