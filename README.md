# MIPS Assembler

A comprehensive two-pass MIPS assembler implemented in C that translates MIPS assembly language programs into machine code. Designed for educational purposes and computer architecture study.

## Overview

This project implements a complete MIPS assembler that reads `.asm` files containing MIPS assembly instructions and generates binary machine code output. The assembler supports a substantial subset of the MIPS instruction set and handles complex features like label resolution and multi-segment programs.

## Features

### Supported Instructions (19 Total)

**R-Type Instructions (Register-based)**
- `add` - Add registers
- `sub` - Subtract registers
- `and` - Bitwise AND
- `or` - Bitwise OR
- `slt` - Set on Less Than
- `sll` - Shift Left Logical
- `srl` - Shift Right Logical
- `sra` - Shift Right Arithmetic

**I-Type Instructions (Immediate-based)**
- `addi` - Add Immediate
- `andi` - AND Immediate
- `ori` - OR Immediate
- `slti` - Set on Less Than Immediate
- `lw` - Load Word
- `sw` - Store Word
- `beq` - Branch on Equal
- `bne` - Branch on Not Equal
- `bltz` - Branch on Less Than Zero
- `blez` - Branch on Less Than or Equal to Zero

**J-Type Instructions (Jump)**
- `j` - Jump
- `jal` - Jump and Link
- `jr` - Jump Register

**Pseudo-Instructions**
- `la` - Load Address
- `li` - Load Immediate
- `nop` - No Operation
- `syscall` - System Call

### Program Structure

- **Two-Pass Assembly**: First pass collects labels and data declarations; second pass generates machine code
- **Label Support**: Full label resolution for jumps and branches with address offset calculations
- **Data/Text Segments**: Separate `.data` (1KB) and `.text` (512B) sections
- **Memory Management**: Uses hash tables for efficient label lookup
- **Binary Output**: Generates `.out` files containing 32-bit machine code instructions

## Project Structure

```
MIPS-Assembler/
├── README.md                                          # This file
├── LICENSE                                            # MIT License
├── test1.asm                                          # Example: Basic arithmetic
├── test2.asm                                          # Example: String handling
└── CIS 580 Computer Architecture.../
    ├── assembler.c          (31KB)                    # Main assembler implementation
    ├── instructionprinter.c                           # Machine code output module
    ├── labelHandler.h                                 # Label management
    ├── preprocessor.h                                 # Comment/whitespace removal
    └── wordHandler.h                                  # Data declaration handling
```

## Building the Assembler

### Prerequisites
- GCC compiler (or compatible C compiler)
- Linux/Unix environment (or Cygwin on Windows)

### Compilation

```bash
cd "CIS 580 Computer Architecture, Fall 2022 Project 1  MIPS Assembler"
gcc -o assembler assembler.c instructionprinter.c labelHandler.c preprocessor.c wordHandler.c
```

## Usage

### Basic Syntax
```bash
./assembler <input_file.asm> <output_file.out>
```

### Example Commands

**Assemble a program:**
```bash
./assembler test1.asm test1.out
```

**Display symbol table:**
```bash
./assembler -symbols test1.asm test1.sym
```

### Output Format

The assembler generates binary output where:
- Text segment instructions: 4 bytes (32 bits) each
- Data segment values: 32-bit words
- Output file: `program-name.out`

## Example Programs

### test1.asm - Basic Arithmetic
```assembly
.data
print_data: .word 0
add_result: .word 0
load_data: .word 1
            .word 2

.text
start: la $1, load_data      # Load address of load_data
       lw $4, 0($1)          # Load first value
       lw $5, 4($1)          # Load second value
       add $4, $4, $5        # Add them
       la $1, add_result     # Load result address
       sw $4, 0($1)          # Store result
```

### test2.asm - String Handling
```assembly
# Store 'Hello world!' on stack
ADDI $sp, $sp, -13
ADDI $t0, $zero, 72           # H
SB $t0, 0($sp)
ADDI $t0, $zero, 101          # e
SB $t0, 1($sp)
# ... (continue for remaining characters)
ADDI $v0, $zero, 4            # Print string syscall
ADDI $a0, $sp, 0
syscall
```

## Implementation Details

### Assembly Process

**Pass 1 (Data Collection):**
- Parse `.data` section and resolve symbol addresses
- Collect all labels in `.text` section with their instruction addresses
- Build label symbol table using hash table (size: 127)
- Track label positions for later offset calculations

**Pass 2 (Code Generation):**
- Parse `.text` section instructions
- For each instruction:
  - Identify instruction type (R/I/J)
  - Extract registers and immediates
  - Calculate branch/jump offsets using label table
  - Generate 32-bit machine code
  - Output to file

### Instruction Encoding

**R-Type Format** (32 bits):
```
[opcode(6)] [rs(5)] [rt(5)] [rd(5)] [shamt(5)] [funct(6)]
```

**I-Type Format** (32 bits):
```
[opcode(6)] [rs(5)] [rt(5)] [immediate(16)]
```

**J-Type Format** (32 bits):
```
[opcode(6)] [address(26)]
```

## Limitations & Known Issues

### Implementation Constraints
1. **ASCII Text Required**: Input files must be plain ASCII text
2. **Line Length**: Maximum 128 bytes per line
3. **Single Sections**: Only one `.data` and one `.text` section allowed
4. **Label Format**: Labels must be on their own line (no inline mnemonics)
5. **Section Headers**: `.data` and `.text` headers must be on separate lines

### Not Implemented
- Exception/interrupt handling
- Full MIPS instruction set (subset only)
- Floating-point instructions
- Memory protection features
- Pipeline simulation

### Pseudo-Instruction Conversion
Some instructions are pseudo-instructions automatically expanded:
- `la` - Expanded to multiple instructions for address loading
- `li` - Converted using immediate load instructions
- `blt`/`ble` - Converted to `slt` + `beq`/`bne`

## Register Reference

| Name | Number | Purpose |
|------|--------|---------|
| `$zero` | 0 | Always zero |
| `$at` | 1 | Assembler temporary |
| `$v0-$v1` | 2-3 | Return values |
| `$a0-$a3` | 4-7 | Function arguments |
| `$t0-$t9` | 8-15, 24-25 | Temporary registers |
| `$s0-$s7` | 16-23 | Saved registers |
| `$gp` | 26 | Global pointer |
| `$sp` | 27 | Stack pointer |
| `$fp` | 28 | Frame pointer |
| `$ra` | 29 | Return address |

## Memory Layout

```
Text Segment:    0x0000 - 0x01FF  (512 bytes)
Data Segment:    0x0800 - 0x0BFF  (1024 bytes)
Stack/Heap:      Above data segment
```

## Testing

Run the included test programs to verify functionality:

```bash
# Test 1: Arithmetic operations
./assembler test1.asm test1.out
cat test1.out

# Test 2: String operations
./assembler test2.asm test2.out
cat test2.out
```

## Educational Use

This assembler is ideal for:
- Computer Architecture courses
- Assembly language learning
- MIPS ISA understanding
- Compiler/interpreter development study
- Understanding machine code generation

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

Developed as part of CIS 580 (Computer Architecture) coursework.

## Author

CodeDroid999

## Contributing

This is an educational project. For improvements or bug reports, please open an issue or submit a pull request.

---

**Last Updated**: January 2023
**Language**: C
**Compiler**: GCC
**Platform**: Linux/Unix
