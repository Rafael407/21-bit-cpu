# Customized 21-bit CPU

A fully functional custom 21-bit CPU designed and simulated in **Logisim Evolution**, implementing Von Neumann architecture with a complete ISA, custom ALU, Booth's multiplication, and a microinstruction-based control unit.

> **Course:** CSE 2114 — Computer Architecture Lab  
> **Institution:** Khulna University of Engineering & Technology (KUET)  
> **Submitted to:** Md. Badiuzzaman Shuvo & Md. Mubtashim Abrar Nihal, Lecturers, CSE, KUET  
> **Author:** Md. Rafid Reza — Roll: 2307120

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Memory Layout](#memory-layout)
- [Registers](#registers)
- [ALU](#alu)
- [Instruction Set](#instruction-set)
- [Instruction Format](#instruction-format)
- [Timing & Clock Cycles](#timing--clock-cycles)
- [Control Unit](#control-unit)
- [Booth's Multiplication](#booths-multiplication)
- [Example Program](#example-program)
- [Project Files](#project-files)
- [How to Run](#how-to-run)
- [Future Enhancements](#future-enhancements)

---

## Overview

This project implements a **custom 21-bit CPU** from scratch in Logisim Evolution. The CPU features:

- **15 instructions** including arithmetic, logic, memory, control flow, and complex multiplication
- **Von Neumann architecture** (unified instruction and data memory)
- **Microinstruction-based control unit** with a 6-bit address / 8-bit data control ROM
- **21 control flags** to manage datapath flow
- **Booth's algorithm** for binary multiplication

---

## Architecture

The CPU follows the classic Von Neumann model:

```
┌─────────────────────────────────────────────────┐
│                     CPU                         │
│  ┌──────────┐   ┌──────────┐   ┌─────────────┐ │
│  │  Control │   │   ALU    │   │  Registers  │ │
│  │   Unit   │──▶│(21-bit)  │◀──│ PC, IR, MAR │ │
│  │  (ROM)   │   │          │   │ MBR, AC, D  │ │
│  └──────────┘   └──────────┘   └─────────────┘ │
└────────────────────────┬────────────────────────┘
                         │ Data/Address Bus (21-bit)
                ┌────────▼────────┐
                │   Main Memory   │
                │ (7-bit addr,    │
                │  21-bit word)   │
                │ ┌─────────────┐ │
                │ │ Code Segment│ │
                │ ├─────────────┤ │
                │ │ Data Segment│ │
                │ └─────────────┘ │
                └─────────────────┘
```

---

## Memory Layout

| Property       | Value                    |
|----------------|--------------------------|
| Address width  | 7 bits (128 locations)   |
| Word size      | 21 bits                  |
| Architecture   | Von Neumann (unified)    |

> ⚠️ **Important:** Since instruction and data share the same memory, the programmer must carefully separate the **code segment** (instructions) from the **data segment** (data values). Mixing them will cause undefined behavior.

---

## Registers

| Register | Size    | Description |
|----------|---------|-------------|
| **PC**   | 7 bits  | Program Counter — holds address of the next instruction |
| **IR**   | 4 bits  | Instruction Register — holds the opcode of the current instruction |
| **MAR**  | 7 bits  | Memory Address Register — holds the memory address to access |
| **MBR**  | 21 bits | Memory Buffer Register — temporarily holds values to/from memory and ALU |
| **AC**   | 21 bits | Accumulator — general-purpose register; holds ALU results and lower word of multiplication |
| **D**    | 21 bits | D Register — stores the upper word of multiplication result |
| **Flag** | 1 bit   | Status Flag Register — stores the Zero Flag from the ALU |

---

## ALU

The 21-bit ALU supports the following operations:

| Operation     | Description                        |
|---------------|------------------------------------|
| AND           | Bitwise AND of two 21-bit values   |
| OR            | Bitwise OR of two 21-bit values    |
| XOR           | Bitwise XOR of two 21-bit values   |
| ADD           | Addition (with carry)              |
| SUB           | Subtraction (with borrow)          |
| INC           | Increment by 1                     |
| DEC           | Decrement by 1                     |
| Left Shift    | Logical left shift by 1 bit        |

### ALU Flags

| Flag         | Description                                           |
|--------------|-------------------------------------------------------|
| **Zero**     | Set when the result is zero                           |
| **Negative** | Set when the result is negative (MSB = 1)             |
| **Overflow** | Set when signed overflow occurs                       |
| **Carry**    | Set when a carry/borrow occurs during add/subtract    |

---

## Instruction Set

### Instruction Format

Each instruction occupies one 21-bit word in memory:

```
┌────────────┬──────────────────┬───────────────────┐
│  Opcode    │   Unused Bits    │  Memory Address   │
│  (4 bits)  │   (10 bits)      │    (7 bits)       │
└────────────┴──────────────────┴───────────────────┘
```

### Full Instruction Table

| Instruction | Opcode | Uses Address? | Execution Microinstructions |
|-------------|--------|---------------|-----------------------------|
| **AND**     | `0000` | Yes           | 1. MBR[addr] → MAR  2. Mem → MBR  3. AC = AC & MBR  4. — |
| **ADD**     | `0001` | Yes           | 1. MBR[addr] → MAR  2. Mem → MBR  3. AC = AC + MBR  4. — |
| **STORE_A** | `0010` | Yes           | 1. MBR[addr] → MAR  2. AC → MBR  3. MBR → Mem  4. — |
| **OR**      | `0011` | Yes           | 1. MBR[addr] → MAR  2. Mem → MBR  3. AC = AC \| MBR  4. — |
| **SUB**     | `0100` | Yes           | 1. MBR[addr] → MAR  2. Mem → MBR  3. AC = AC - MBR  4. — |
| **JMP**     | `0101` | Don't care    | 1. MBR[addr] → PC  2. —  3. —  4. — |
| **Ld_to_A** | `0110` | Yes           | 1. MBR[addr] → MAR  2. Mem → MBR  3. MBR → AC  4. — |
| **HLT**     | `0111` | Don't care    | 1. Freeze control unit clock  2. —  3. —  4. — |
| **JZ**      | `1000` | Yes           | 1. MBR[addr] → PC *(if Zero Flag = 1)*  2. —  3. —  4. — |
| **INC**     | `1001` | Don't care    | 1. AC = AC + 1  2. —  3. —  4. — |
| **DEC**     | `1010` | Don't care    | 1. AC = AC - 1  2. —  3. —  4. — |
| **XOR**     | `1011` | Yes           | 1. MBR[addr] → MAR  2. Mem → MBR  3. AC = AC ^ MBR  4. — |
| **LShift**  | `1100` | Don't care    | 1. AC = AC << 1  2. —  3. —  4. — |
| **STORE_D** | `1101` | Yes           | 1. MBR[addr] → MAR  2. D → MBR  3. MBR → Mem  4. — |
| **MUL**     | `1110` | Yes           | 1. MBR[addr] → MAR  2. Mem → MBR  3. Reset Booth's regs  4. AC = AC × MBR |

---

## Timing & Clock Cycles

Every instruction goes through a **Fetch Cycle** followed by an **Execution Cycle**.

### Fetch Cycle (3 microinstructions)
1. PC → MAR
2. Memory → MBR
3. PC = PC + 1

### Execution Cycle
- Most instructions: **4 microinstructions**
- Total per instruction: **7 clock pulses** (max)

### Multiplication (MUL)
Booth's algorithm runs iteratively over 21 bits:

```
Total = Fetch (3) + Execution setup (3) + Booth's iterations (2 × 21 = 42)
      ≈ 46–50 clock pulses
```

---

## Control Unit

The control unit is ROM-based:

| Property          | Value        |
|-------------------|--------------|
| Address width     | 6 bits       |
| Data width        | 8 bits       |
| Total flags raised| 21           |

The IR register feeds the opcode into the control unit, which sequences the appropriate microinstruction signals for each clock pulse.

---

## Booth's Multiplication

The `MUL` instruction uses **Booth's Algorithm** for signed 21-bit binary multiplication.

- **Inputs:** Accumulator (AC) as multiplicand, value from memory as multiplier
- **Output:**
  - Lower 21 bits → stored in **AC**
  - Upper 21 bits → stored in **D register**
- The Booth's circuit runs for `2 × 21 = 42` internal clock steps before returning the result

---

## Example Program

The following program demonstrates load, OR, store, and add operations.

**Memory image (Logisim hex format):**
```
v2.0 raw
0c0020 060021 040030 020022 040031
```

| Step | Hex Code | Opcode | Address | Operation |
|------|----------|--------|---------|-----------|
| 1    | `0c0020` | `0110` (Ld_to_A)  | `0x20` | Load value from memory[0x20] → AC |
| 2    | `060021` | `0011` (OR)       | `0x21` | AC = AC \| memory[0x21] |
| 3    | `040030` | `0010` (STORE_A)  | `0x30` | Store AC → memory[0x30] |
| 4    | `020022` | `0001` (ADD)      | `0x22` | AC = AC + memory[0x22] |
| 5    | `040031` | `0010` (STORE_A)  | `0x31` | Store AC → memory[0x31] |

---

## Project Files

| File | Description |
|------|-------------|
| `CPU.circ` | Top-level CPU circuit |
| `comp.circ` | Complete computer (CPU + memory) |
| `control_unit.circ` | ROM-based control unit |
| `21-bit-ALU.circ` | Main 21-bit ALU |
| `21-bit-ALU_for_booths.circ` | ALU variant used inside Booth's circuit |
| `Booths_multiplication.circ` | Booth's multiplication circuit |
| `rom_info` | Control ROM contents/configuration |
| `Archi_project.pdf` | Full project report |

---

## How to Run

1. Install [Logisim Evolution](https://github.com/logisim-evolution/logisim-evolution/releases) (v3.x or later recommended)
2. Open `comp.circ` — this is the top-level file containing the full computer
3. Load a program into the main memory (RAM) using Logisim's hex editor (`v2.0 raw` format)
4. Ensure instructions are placed in the **code segment** and data values in the **data segment**
5. Run the simulation using the clock tool (manual step or auto-tick)

> **Tip:** Use a slow clock tick rate when first testing to observe microinstruction execution step by step.

---

## Future Enhancements

- [ ] Pipelining the datapath for higher instruction throughput
- [ ] Cache memory system
- [ ] Division instruction
- [ ] Floating-point arithmetic unit
- [ ] Interrupt handling mechanism
- [ ] Extended ISA with additional instructions
