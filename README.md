# Virtual-Machine
A simple virtual machine with the highly coveted and useful feature of heap banks where it takes a single command line argument being the path to the file containing the RISK-XVII assembly code and produces the output of it.

**The Architecture**
- 0x0000 - 0x3ff: Instruction Memory - Contains 2
10 of bytes for text segment.
- 0x0400 - 0x7ff: Data Memory - Contains 2
10 of bytes for global variables, and function stack.
- 0x0800 - 0x8ff: Virtual Routines - Accesses to these address will cause special operations to be
called.
- 0xb700 +: Heap Banks - Hardware managed 128 x 64 bytes banks of dynamically allocate-able
memory.

The machine also has a total of 32 registers, as well as a PC (program counter) that points to the address of the current instruction in memory. Each of the general-purpose registers can store 4 bytes (32 bits) of data that can be used directly as operands for instructions. All registers are generalpurpose except for the first one, which has an address of 0. This register is called the zero register, as any read from it will return a value of 0. Writes to the zero register are ignored.


**RISK-XVII Instruction-Set Architecture**

An Instructions-Set Architecture(ISA) specifies a set of instructions that can be accepted and executed by the target machine. A program for the target machine is an ordered sequence of instructions. The virtual machine will operate on a home-brewed ‘RISK-XVII’ instruction set architecture. During marking, you will be provided with binaries in this ISA to run on your virtual machine. RISKXVII is a reduced version of the well-known RV32I instruction set architecture, and your virtual machine should be able to execute binary programs compiled for RV32I, as long as they do not include instructions that were not specified by ‘RISK-XVII’. There are in total 33 instructions defined in RISK-XVII, they can be classified into three groups by their functionality:

1. Arithmetic and Logic Operations - e.g. add, sub, and, or, slt, sll

2. Memory Access Operations - e.g. sw, lw, lui

3. Program Flow Operations - e.g. jal, beq, blt

These instructions provide access to memory and perform operations on data stored in registers, as well as branching to different locations within the program to support conditional execution. Some instructions also contain data, i.e., an immediate number, within their encoding. This type of instruction is typically used to introduce hard-coded values such as constants or memory address offsets.

Instructions in the RISK-XVII instruction set architecture are encoded into 4 bytes of data. Since each instruction can access different parts of the system, six types of encoding formats were designed to best utilize the 32 bits of data to represent the operations specified by each instruction: R, I, S, SB, U, UJ. The exact binary format of each encoding type can be found in the table below.:

<img width="662" alt="Screen Shot 2023-05-13 at 3 06 04 pm" src="https://github.com/dotori0409/Virtual-Machine/assets/71887438/fc90ecd8-128e-440e-9ec8-6d558be18af3">

- opcode: used in all encoding to differentiate the operation, and even the encoding type itself.
- rd, rs1, rs2: register specifiers. rs1 and rs2 specify registers to be used as the source operand, while rd specifies the target register. Note that since there are 32 registers in total, all register specifiers are 5 bits in length.
- func3, func7: these are additional opcodes that specify the operation in more detail. For example, all arithmetic instructions may use the same opcode, but the actual operation, e.g. add, logic shift, are defined by the value of func3.
- imm: immediate numbers. These value can be scrambled within the instruction encoding. For example, in SB, the 11st bit of the actual value was encoded at the 7th bit of the instruction, while the 12rd bit was encoded at the 31th bit


An RISK-XVII program can be illustrated as below:
[Instruction 1 (32 bits)]
[Instruction 2 (32 bits)]
[Instruction 3 (32 bits)]
[...]
[Instruction n (32 bits)]


**Virtual Routines**

Virtual routines are operations mapped to specific memory addresses such that a memory access operation at that address will have different effects. This can be used to allow programs running in the virtual machine to communicate with the outside world through input/output (I/O) operations.

1. 0x0800 - Console Write Character
A memory store command to this address will cause the virtual machine to print the value being stored as a single ASCII encoded character to stdout.

2. 0x0804 - Console Write Signed Integer
A memory store command to this address will cause the virtual machine to print the value being stored as a single 32-bit signed integer in decimal format to stdout.

3. 0x0808 - Console Write Unsigned Integer
A memory store command to this address will cause the virtual machine to print the value being stored as a single 32-bit unsigned integer in lower case hexadecimal format to stdout.

4. 0x080C - Halt
A memory store command to this address will cause the virtual machine to halt the current running program, then output CPU Halt Requested to stdout, and exit, regardless the value to be stored.

5. 0x0812 - Console Read Character
A memory load command to this address will cause the virtual machine to scan input from stdin and treat the input as an ASCII-encoded character for the memory load result.

6. 0x0816 - Console Read Signed Integer
A memory load command to this address will cause the virtual machine to scan input from stdin and parse the input as a signed integer for the memory load result.

7. 0x0820 - Dump PC
A memory store command to this address will cause the virtual machine to print the value of PC in lower case hexadecimal format to stdout.

8. 0x0824 - Dump Register Banks
A memory store command to this address will force the virtual machine to perform an Register Dump. See Error Handling.

9. 0x0828 - Dump Memory Word
A memory store command to this address will cause the virtual machine to print the value of M[v] in lower case hexadecimal format to stdout. v is the value being stored interpreted as an 32-bit unsigned integer.

10. 0x0830, 0x0834 - Heap Banks
These addresses are used by a hardware-enabled memory allocation system.

11. 0x0850 and above - Reserved
Our program will not call these addresses. You are free to use them for your own test cases and debugging purposes.


**Heap Banks**

The virtual machine program should manage memory allocation requests and ensure that the ownership of each block is always unique to malloc requests unless it is not used (free). An error is returned if it is not possible to fulfill such a memory request. The mapped address of the initial byte of the first bank is 0xb700, and there are a total of 8192 bytes of memory that can be dynamically
allocated.

1. 0x0830 - malloc
A memory store command to this address will request a chunk of memory with the size of the value being stored to be allocated. The pointer of the allocated memory (starting address) will be stored in R[28]. If the memory cannot be allocated, R[28] should be set to zero.

2. 0x0834 - free
A memory store command to this address will free a chunk of memory starting at the value being stored. If the address provided was not allocated, an illegal operation error should be raised.


**Error Handling**

Register Dump
When an register dump was requested, the virtual machine should print the value of all registers, including PC, in lower case hexadecimal format to stdout. The output should be one register per line in the following format:

PC = 0x00000001;
R[0] = 0xffffffff;
R[1] = 0xffffffff;
R[...] = 0xffffffff;
R[31] = 0xffffffff

Not Implemented
If an unknown instruction was detected, your virtual machine program should output Instruction Not Implemented: and the hexadecimal value of the encoded instruction to stdout, followed by a Register Dump and terminate. For example, when if 0xffffffff were found in the instruction memory, your program should output:

Instruction Not Implemented: 0xffffffff
