# RISC-V Microprocessor

*This repository is done as part of the [RISC-V MYTH Workshop](https://github.com/stevehoover/RISC-V_MYTH_Workshop).*

## Table of Contents
- [RISC-V Microprocessor](#risc-v-microprocessor)
  - [Table of Contents](#table-of-contents)
  - [From application to hardware, a high level introduction](#from-application-to-hardware-a-high-level-introduction)
  - [Code compilation](#code-compilation)
    - [Disassemble the compiled file](#disassemble-the-compiled-file)
    - [Debugging/Simulating the compiled file](#debuggingsimulating-the-compiled-file)
  - [Initial RISC-V Core Implementation](#initial-risc-v-core-implementation)
    - [PC](#pc)
    - [Instruction Memory](#instruction-memory)
    - [Decoder](#decoder)
    - [Register File](#register-file)
    - [ALU](#alu)
  - [Pipelining RISC-V Core & Misc](#pipelining-risc-v-core--misc)
  - [Final Implementation](#final-implementation)
  - [Further Work](#further-work)

## From application to hardware, a high level introduction

Let's first go back to how the computer works, starting at an user application. Take for example any application, such as a notepad, the application is almost certain to have been coded using a relatively high level language such as C, C++, Java, Python, etc. The code is very much readable to humans but not so much for computers. At the computer level, all these code need to be translated into 1s and 0s. In order to do so, the code first has to go through a complier, which translates the code into instructions, then a assembler, which translate the code into binary 1s and 0s. Once we have these 1s and 0s, the hardware could then execute the program.

From a design perspective, register-transfer-level (RTL) is the design abstraction used to model the hardware circuits and effectively allows for simulation of the execution of 1s and 0s. RTL is used in hardware description languages like Verilog and VHDL to create human understandable representations of the hardware. For this repository, we will be using TL-Verilog as the hardware description language.

## Code compilation

We will first start by generating some RISC-V compatible instructions for testing of our RISC-V core.

Let's assume you have written a simple application in the C programming language, we could first compile the code using bash command below:

```bash
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o <output file> <input file>
```

where,
- `-Ofast` refers to the level of optimization used
- `-mabi` refers to the target ABI
- `-march` refers to the target architecture
- `-o` refers to the output file

Once compiled the output file should contain the relavant instructions need to run the application using a RISC-V core.

### Disassemble the compiled file

The output instructions can be viewed on screen using the bash command below:

```bash
riscv64-unknown-elf-objdump -d <compiled file> | less
```

One trick is that when using `less`, you could find a text by typing forward slash follow by the text. Say if I typed `/main` to search for the string "main". Then to find the next occurrence, hit `Enter`. To quit `less`, hit `q`.

### Debugging/Simulating the compiled file

We will be using `spike` for debugging and simulating the compiled file for now. To start the simulation, we will run the command below:

```bash
spike pk <compiled file>
```

The output of the compiled application, if any, will be displayed on screen as if executed by a RISC-V core.

We could further debug the compiled application by executing the compiled file instruction by instruction. In order to do so, we will need to execute the command with a `-d` option.

```bash
spike -d pk <compiled file>
```

Running this command will return a prompt on screen. Let's say if we want to executed till program counter reaches instruction 100b0, we could run the following command:

```
until pc 0 100b0
```

*As a side note, the zero right after `pc` refers to CPU core 0*

And then, to execute the next instruction, we could just press enter.

At any point in time, we could examine the value of any register by typing the following command:

```
reg 0 <register>
```

To fully exit the debug prompt, just press `q` at any point.

## Initial RISC-V Core Implementation

Now that we are comfortable with our compiled instructions, we will move on to creating our first RISC-V core that attempts to implement most of RV32I Base Integer Instruction Set.

We will first start by implementing the RISC-V core without any consideration of pipelining, i.e., we will implement the core as shown below:

![](/risc_1.png)
(Courtesy of [Steve Hoover](https://github.com/stevehoover))

### PC

The program counter implementation is relatively straight forward since all instructions are 32 bits, PC increment by 4 bytes each time.

### Instruction Memory

Instruction memory store contains all the instructions, when given a PC as input, it outputs the actual instruction.

### Decoder

The decoder decodes the instruction into its components, namely, `opcode`, `rs1`, `rs2`, `rd`, `funct3`, `funct7` and `imm`. The different instruction formats are shown below:

![](/risc_2.png)
(The RISC-V Instruction Set Manual)

### Register File

The register file provides ability to read 2 registers or write 1 register given the correct inputs.

### ALU

The arithmetic-logic unit performs the necessary computation based on the opcode of the current instruction. The specifics can be found within [the specifications listed on RISC-V website](https://riscv.org/technical/specifications/).

## Pipelining RISC-V Core & Misc

Once we have a working RISC-V Core, we could start think about pipelining the RISC-V core. There are two harzards, branching hazard and read after write (RAW) hazard, that needs to be resolved when performing pipelining.

To resolve the read after write hazardm we will be implementing a register bypass, and to resolve the branching hazard we will be invalidating the cycles that are invalid.

On top of the pipelining, jumps (JAL or JALR) will be implemented.

The pipelined core will look like something below:

![](/risc_3.png)
(Courtesy of [Steve Hoover](https://github.com/stevehoover))

## Final Implementation

The final implementation can be found [here](/Day3_5/risc-v_solutions.tlv).

## Further Work

- [ ] Complete the implementation of RV32I instruction set
