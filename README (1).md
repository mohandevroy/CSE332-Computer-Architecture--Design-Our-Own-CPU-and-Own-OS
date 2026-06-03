# 🖥️ MIPS Single-Cycle Processor — CSE332 Group 4 (FALL 2025)

A complete hardware implementation of a **32-bit MIPS Single-Cycle Processor** designed in Verilog, accompanied by a custom **MIPS Assembler** written in C++. The processor supports a rich instruction set including arithmetic, logic, memory, branching, and jump-and-link operations, simulated and verified using ModelSim on WSL (Windows Subsystem for Linux).

---

## 📌 Project Overview

This project implements the full digital design pipeline for a MIPS-based CPU:

- **RTL design** of all processor datapath and control components in Verilog HDL
- **Extended instruction support** beyond the basic MIPS ISA — including `JAL`, `JR`, `LUI`, shift operations, and more
- **A custom MIPS assembler** capable of translating `.s` assembly programs into binary machine code loadable by the processor
- **End-to-end testing** using ModelSim waveform simulation with real MIPS programs (Min/Max/Mean, factorial, array operations)

---

## 🏗️ Architecture

The processor follows a classic **single-cycle MIPS** architecture where every instruction completes in one clock cycle.

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  Instruction │────▶│   Control    │────▶│   Datapath   │
│  Memory(ROM) │     │    Unit      │     │              │
└─────────────┘     └──────────────┘     └──────┬───────┘
                                                 │
                                         ┌───────▼───────┐
                                         │  Data Memory  │
                                         │    (RAM)      │
                                         └───────────────┘
```

### Top-Level Module: `MIPS_SCP`

| Sub-module        | File               | Description                                      |
|-------------------|--------------------|--------------------------------------------------|
| `Datapath`        | `datapath.v`       | PC logic, register file, ALU wiring, mux network |
| `Controlunit`     | `control.v`        | Decodes opcode/funct → control signals            |
| `alu32`           | `alu32.v`          | 32-bit ALU with 15 operations                    |
| `registerfile32`  | `regfile32.v`      | 32×32-bit register file                          |
| `ram`             | `ram.v`            | 128-word synchronous data memory                 |
| `rom`             | `rom.v`            | 256-word instruction memory (loaded from file)   |
| `adder`           | `adder.v`          | Parameterized adder (PC+4, branch target)        |
| `mux2` / `mux4`   | `mux2.v`, `mux4.v` | Parameterized multiplexers                       |
| `signext`         | `signext.v`        | 16→32-bit sign extender                          |
| `slt2`            | `sl2.v`            | Left-shift-by-2 (branch offset scaling)          |
| `flopr_param`     | `flopr_param.v`    | Parameterized flip-flop register (PC register)   |

---

## 🧠 Supported Instructions

### R-Type
| Instruction | Operation          |
|-------------|--------------------|
| `ADD`/`ADDU`| Addition           |
| `SUB`/`SUBU`| Subtraction        |
| `AND`       | Bitwise AND        |
| `OR`        | Bitwise OR         |
| `XOR`       | Bitwise XOR        |
| `NOR`       | Bitwise NOR        |
| `SLT`/`SLTU`| Set-less-than      |
| `SLL`/`SRL`/`SRA` | Shift immediate |
| `SLLV`/`SRLV`/`SRAV` | Shift variable |
| `JR`        | Jump register      |

### I-Type
| Instruction | Operation               |
|-------------|-------------------------|
| `ADDI`/`ADDIU` | Add immediate        |
| `ANDI`/`ORI`/`XORI` | Logic immediate |
| `SLTI`/`SLTIU` | Set-less-than imm   |
| `LUI`       | Load upper immediate    |
| `LW` / `SW` | Load/store word         |
| `BEQ` / `BNE` | Branch equal/not-equal |

### J-Type
| Instruction | Operation              |
|-------------|------------------------|
| `J`         | Unconditional jump     |
| `JAL`       | Jump and link (`$ra` ← PC+4) |

---

## 📁 Project Structure

```
Group_4_CSE332_Project_FALL2025/
│
├── *.v                          # Final Verilog source files (top-level)
│   ├── MIPS_SCP.v               # Top-level processor module
│   ├── MIPS_SCP_tb.v            # Testbench
│   ├── control.v                # Control unit
│   ├── datapath.v               # Datapath
│   ├── alu32.v                  # 32-bit ALU
│   ├── regfile32.v              # Register file
│   ├── ram.v / rom.v            # Data & instruction memories
│   └── ...                      # Supporting modules
│
├── memfile.txt                  # Binary instruction memory image
│
└── Root/
    ├── Project 0/               # Baseline: ModelSim + WSL environment check
    ├── Project 1/               # JAL & JR instruction verification
    ├── Project 2/               # Min/Max/Mean algorithm test
    ├── Project 3/               # Full final integration test
    ├── MIPSVerilogWOJAL/        # Baseline Verilog without JAL/JR (reference)
    ├── MipsAssembler/           # Original assembler (CMake, C++)
    ├── UpgradedMIPS32Assembler/ # Extended assembler with data section support
    │   ├── mips.h               # Assembler core (instruction encoding)
    │   ├── README.md            # Assembler usage guide
    │   ├── *.s                  # Sample MIPS assembly programs
    │   ├── output/              # Generated machine code (.out, .bin)
    │   └── logs/                # Simulation logs
    ├── images/                  # Waveform and simulation screenshots
    └── video recording/         # Demo recordings (ModelSim, WSL)
```

---

## 🛠️ Tools & Environment

| Tool        | Purpose                                      |
|-------------|----------------------------------------------|
| ModelSim    | Verilog simulation and waveform analysis      |
| WSL (Ubuntu)| Linux environment for assembler and toolchain |
| CMake + GCC | Build system for the C++ MIPS assembler       |
| Verilog HDL | Hardware description language for the CPU     |

---

## 🔧 Building the Assembler

Navigate into the assembler directory and build with CMake:

```bash
cd Root/UpgradedMIPS32Assembler/MipsAssembler
mkdir -p build
cd build
cmake ..
make
cd ..
```

### Running the Assembler

```bash
./build/MipsAssembler <input.s> <output.out> <log.txt>
```

**Example — assemble a factorial program:**
```bash
./build/MipsAssembler input.s/factorial.s output/factorial.out logs/factorial.log
```

The assembler produces:
- `factorial.out` — machine code with addresses (text section)
- `factorial.out.no_address.text.bin` — raw binary for the text section
- `factorial.out.no_address.data.bin` — raw binary for the data section

The `.text.bin` file can be used directly as the `memfile.txt` input for the Verilog ROM.

### Supported Instructions (Assembler)
The assembler supports **46 MIPS instructions** including `lw`, `sw`, `add`, `sub`, `slt`, `and`, `or`, `xor`, `nor`, `sll`, `srl`, `sra`, `mult`, `div`, `addi`, `ori`, `lui`, `beq`, `bne`, `j`, `jal`, `jalr`, `jr`, `syscall`, pseudo-instructions (`li`, `la`, `move`, `nop`), and more.

---

## 🧪 Test Programs

The project was validated using real MIPS programs assembled and loaded into the processor:

| Test                  | Description                                               |
|-----------------------|-----------------------------------------------------------|
| **Min/Max/Mean**      | Finds minimum, maximum, sum, and average of 3 values      |
| **Factorial**         | Recursive factorial using `JAL`/`JR` for subroutine calls |
| **Increment Array**   | Iterates over an array and increments each element        |
| **JAL/JR Test**       | Validates jump-and-link and jump-register behavior        |
| **Hello MIPS**        | Basic syscall and register operation sanity check         |

---

## 📽️ Simulation & Demo

Simulation was performed in **ModelSim** with waveform inspection. Video recordings of each milestone are included under `Root/video recording/`:

- `WSL_Implement.mkv` — Setting up the WSL + ModelSim environment
- `JALJR.mkv` — Demonstrating JAL and JR instruction execution
- `DataSupport.mkv` — Verifying data memory (LW/SW) operations
- `mmm.mkv` — Min/Max/Mean program running on hardware

---

## 🔑 Key Design Decisions

- **JAL** writes `PC+4` into register `$ra` (register 31) and jumps to the target; implemented by adding a `Jal` control signal and a dedicated MUX in the register write-back path.
- **JR** redirects the PC to the value in `rs`; implemented via a `Jr` control signal that bypasses normal PC selection to use `dataone` (the value read from `rs`).
- **BNE** is handled within the control unit by an extra `BNE` flag XOR'd with the `Zero` flag before driving `PCSrc`.
- **LUI** is implemented directly in the ALU (operation `4'b1110`) by placing the 16-bit immediate into the upper half of the 32-bit result.
- The **ROM** loads its instruction image from `memfile.txt` at simulation start using `$readmemb`, making it straightforward to swap test programs.

---

## 👥 Group 4 — CSE332, FALL 2025
    Mohan Dev Roy
> Computer Organization and Architecture  
> Department of Computer Science & Engineering

---

## 📄 License

This project was developed for academic purposes as part of the CSE332 course curriculum. Not intended for commercial use.
