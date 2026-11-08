# FPGA Digital Design & RISC-V Processor

Seven hardware labs from Northeastern University's **EECE 2310/2311: Digital Design and Computer Architecture** (Spring 2026). Each lab built on the last, starting with a 1-bit half adder and ending with a working single-cycle RISC-V processor running on an FPGA.

Every design was simulated in Quartus, then programmed onto a **DE1-SoC board** and verified in person using the board's switches, pushbuttons, LEDs, and 7-segment displays.

<!-- Optional: add a photo or GIF of the RISC-V processor running on the board -->
<!-- ![RISC-V processor on the DE1-SoC](images/riscv_demo.gif) -->

## Tools & Hardware

- **Software:** Intel Quartus Prime Lite (schematic capture, waveform simulation, pin assignment), Verilog, RARS (RISC-V assembler)
- **Hardware:** Terasic DE1-SoC board (Intel Cyclone V FPGA, 5CSEMA5F31C6)
- **Methods:** truth tables, Karnaugh map minimization, finite state machine design, simulation-based testing, hardware verification

## How the labs build on each other

```
Half/Full Adder ──► 4-bit Adder/Subtractor ──► ALU (Lab 4) ──┐
                                                             ├──► RISC-V Processor (Lab 7)
BCD → 7-Segment Decoder (Lab 3) ──► Decimal ALU display ─────┤
                                                             │
Counters (Lab 5) ──► Program Counter ────────────────────────┘
FSM design (Lab 6)
```

## Labs

| # | Lab | What I built | Report |
|---|-----|--------------|--------|
| 1 | Binary Adders | Half adder and full adder from logic gates; verified with simulation waveforms and on-board switches/LEDs | [PDF](reports/Lab01_Binary_Adders_Quartus_Intro.pdf) |
| 2 | 4-bit Adder/Subtractor | Ripple-carry adder/subtractor using two's complement, with signed and unsigned overflow detection derived from K-maps | [PDF](reports/Lab02_4bit_Adder_Subtractor.pdf) |
| 3 | 7-Segment Display Control | BCD-to-7-segment decoder minimized with seven K-maps, extended with enable and negative-sign logic to drive two active-low displays | [PDF](reports/Lab03_7_Segment_Display_Control.pdf) |
| 4 | Arithmetic Logic Unit | 4-bit ALU (ADD, SUB, AND, OR) with a binary-to-BCD converter showing signed results in decimal on the 7-segment displays | [PDF](reports/Lab04_Arithmetic_Logic_Unit_ALU.pdf) |
| 5 | Digital Counters | Counter system driven by the 50 MHz board clock with start/stop, asynchronous load, and up/down counting | [PDF](reports/Lab05_Digital_Counters_Design.pdf) |
| 6 | Railroad Crossing FSM | Finite state machine for a railroad crossing signal using two track sensors; reduced from five states to four by merging equivalent states, saving a flip-flop | [PDF](reports/Lab06_Railroad_Crossing_FSM.pdf) |
| 7 | RISC-V Single-Cycle Processor | Compact RISC-V CPU executing R-type ADD, SUB, AND, and OR instructions, combining my Lab 4 ALU, a Verilog register file and instruction memory, a program counter, and a control unit | [PDF](reports/Lab07_RISC-V_Single_Cycle_Processor.pdf) |

## Highlight: Compact RISC-V Processor (Lab 7)

The final lab combined earlier work into a working processor:

- **Program counter:** an up-counting `LPM_COUNTER` that advances one instruction per clock
- **Instruction memory:** written in Verilog and loaded with machine code assembled in RARS (`mycode.hex`)
- **Register file:** written in Verilog, with two read ports (`rs1`, `rs2`) and one write port (`rd`)
- **Control unit:** a combinational circuit that picks the ALU operation from just three instruction bits (funct3 bits 12–13 and funct7 bit 30)
- **ALU:** reused directly from Lab 4

On the board, each press of `KEY[0]` executes one instruction and shows the result on the 7-segment displays. A reset returns the PC to 0 and restarts the program.

<!-- ![Single-cycle datapath](images/riscv_datapath.png) -->

## Highlight: Railroad Crossing FSM (Lab 6)

Two sensors (east and west) detect a one-way train, and a signal light stays on while any part of the train is in the crossing. The design handles trains of any length and treats irregular sensor sequences as invalid. After state minimization, the FSM needs only four states.

<!-- ![FSM state diagram](images/fsm_state_diagram.png) -->

## Repository Structure

```
├── README.md
├── images/        # schematics, state diagram, datapath, demo photos
└── reports/       # full lab reports (PDF)
```

## Author

**Sijing Deng** · B.S. Computer Science, Robotics minor · Northeastern University  
[GitHub](https://github.com/SijingDeng24) · [LinkedIn](https://linkedin.com/in/sijingdeng) · deng.siji@northeastern.edu
