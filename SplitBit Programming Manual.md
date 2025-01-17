# General Description:

SplitBit is a small 8 bit CPU.
It is a Harvard Architecture machine with a separate 64k memory space for its Program and another for its Data.

It has six registers:
 - The A and B Registers are the 8 bit operands for the ALU. All ALU operations use them as operands. They can also be used as general purpose accumulators.
 - The Q Register is the 8 bit ALU output register, all ALU operations store their result in it.
 - The Program Counter is a 16 bit pointer into the Program Memory. It points to the current operation the CPU is executing. It initializes at address 0x0000.
 - The Data Pointer is a 16 bit pointer into the Data Memory. It points to the current byte of data that the CPU can read or write to, and can be set arbitrarily by the programmer to any value in the Data Memory. It initializes at location 0x0000.
 - The Stack Pointer is a 16 bit pointer into the Data Memory. It points to the current element of the stack, and is only ever modified by the use of the push and pop instructions. It initializes at location 0xFFFF.
 - The Status register is an 8 bit register whose various bits are used as flags. Only three of these flags are used in the current implementation.
	 - Bit 0 is the Carry/Borrow Flag. Any arithmetic operation either sets or clears it depending on whether or not the result causes Q to overflow/underflow. It is a 1 if a carry/underflow occurred, and a 0 otherwise.
	 - Bit 1 is the Stack Collision Flag. It is set if the Data Pointer's value ever meets or exceeds the Stack Pointer's value. This condition also sets the Halt Flag.
	 - Bit 7 is the Halt Flag. It is set by the HALT instruction, or if there is a stack collision.

## List of Instructions:

### Arithmetic and Logic Operations: 9 Instructions
Hex Code | Mnemonic | Description
-- | -- | --
00 | ADD |  Adds A, B, and the Carry Flag, the result is stored in Q.
01 | SUB | Subtracts B and the Carry Flag from A, the result is stored in Q.
02 | AND | Bitwise and of A and B, the result is stored in Q.
03 | OR | Bitwise or of A and B, the result is stored in Q.
04 | XOR | Bitwise xor of A and B, the result is stored in Q.
05 | NOTA | Bitwise inversion of A, the result is stored in Q.
06 | NOTB | Bitwise inversion of B, the result is stored in Q.
07 | SHL | A and B form a circular shift register. Rotate this register left.
08 | SHR | A and B form a circular shift register. Rotate this register right.

### Branch and Subroutine Operations: 7 Instructions
Hex Code | Mnemonic | Description
-- | -- | --
10 | BRI | Branch Immediately. Loads the immediate next two bytes of Program Memory into the Program Counter, first the most significant byte, then the least.
11 | BRQ | Branch on Q. If Q is zero, loads the immediate next two bytes of Program Memory into the Program Counter.
12 | BRA | Branch on A. If A is zero, loads the immediate next two bytes of Program Memory into the Program Counter.
13 | BRB | Branch on B. If B is zero, loads the immediate next two bytes of Program Memory into the Program Counter.
14 | BRC  | Branch if Carry is set.
17 | CALL | Call subroutine. Stores all the registers to the Stack, A, B, Q, the Data Pointer, and the Program Counter, then performs an immediate branch.
1F | RET  | Restores all registers from the Stack, then immediately branches to the Return Address by setting the Program Counter to the next instruction after the last CALL.


### Register Operations: 11 Instructions
| Hex Code | Mnemonic | Description         |
| -------- | -------- | ------------------- |
| 20       | RSTA     | Resets A to 0.      |
| 21       | RSTB     | Resets B to 0.      |
| 22       | INCA     | Adds 1 to A.        |
| 23       | INCB     | Adds 1 to B.        |
| 24       | DECA     | Subtracts 1 from A. |
| 25       | DECB     | Subtracts 1 from B. |
| 26       | LDA	  | Loads Data to A via the Data Pointer. |
| 27	   | LDB      | Loads Data to B via the Data Pointer.
| 28	   | INIA     | Loads the next byte of Program Memory to A. |
| 29       | INIB     | Loads the next byte of Program Memory to B. |
| 2F	   | CCF      | Clears the Carry Flag. |

### Stack Operations: 7 Instructions
Hex Code | Mnemonic | Description
-- | -- | --
30 | PSHQ | Stores Q into Data Memory at the location referenced by the Stack Pointer then decrements the Stack Pointer.
31 | PSHA | Stores A into Data Memory at the location referenced by the Stack Pointer then decrements the Stack Pointer.
32 | PSHB | Stores B into Data Memory at the location referenced by the Stack Pointer then decrements the Stack Pointer.
33 | PSHD | Stores the Data Pointer to the stack, with the low byte on top. Increments the Stack Pointer by two.
34 | POPA | Reads the location referenced by the Stack Pointer from Data Memory into A then increments the Stack Pointer.
35 | POPB | Reads the location referenced by the Stack Pointer from Data Memory into B then increments the Stack Pointer.
36 | POPD | Restores the Program Counter from the stack, increments the Stack Pointer by two.

### Data Operations: 10 Instructions
Hex Code | Mnemonic | Description
-- | -- | --
40 | INCD | Increments the Data Pointer.
41 | DECD | Decrements the Data Pointer.
42 | LDA | Loads the byte referenced from Data Memory by the Data Pointer into A.
43 | LDB | Loads the byte referenced from Data Memory by the Data Pointer into B.
44 | STQ | Stores Q into the byte referenced by the Data Pointer in Data Memory.
45 | STA | Stores A into the byte referenced by the Data Pointer in Data Memory.
46 | STB | Stores B into the byte referenced by the Data Pointer in Data Memory.
47 | SETD | Loads the next two bytes of Program Memory into the Data Pointer.
48 | DPUP | Offset Data Pointer up by the value of the immediate next byte of Program Memory.
49 | DPDN | Offset Data Pointer down by the value of the immediate next byte of Program Memory.


### Output Operations: 3 Instructions
Hex Code | Mnemonic | Description
-- | -- | --
D0 | OUTQ | Writes the value of Q to an Output specified by the next byte of Program Memory.
D1 | OUTA | Writes the value of A to an Output specified by the next byte of Program Memory.
D2 | OUTB | Writes the value of B to an Output specified by the next byte of Program Memory.

### Input Operations: 2 Instructions
Hex Code | Mnemonic | Description
-- | -- | --
E0 | INA | Writes the value of an Input to A. The input port is specified the next byte of Program Memory.
E1 | INB | Writes the value of an Input to B. The input port is specified the next byte of Program Memory.

### Special Operations: 2 Instructions
Hex Code | Mnemonic | Description
-- | -- | --
F0 | NOP | Perform no Operation, increment the Program Counter.
FF | HALT | Stops CPU Execution.

## Input and Output In the Emulator:

The current implementation has Input 0 and Output 0 hooked to stdout and stdin respectively, allowing programs to read to and from the console.

### Example Program: Hello World
```
; This is a basic hello world program for the SplitBit CPU.
; We'll create a loop that outputs each byte of our string to Output 0, the text console.

#Program

Start:
  LDA         ; Load a byte of the string into A.
  BRA End     ; If A is zero, branch out of the loop.
  OUTA 0x00   ; Output the value in A to Port 0, the text console.
  INCD        ; Increment the Data Pointer to the next byte of the string.
  BRI Start   ; Branch immediately to the start of the loop.

End:
  INIA 0x0A   ; We'll load a linefeed into A and output it to make it look nice.
  OUTA 0x00   ; Output it to the text console.
  HALT        ; Terminate the program.

#Data

"Hello, World!"
```

### Structure of a SplitBit Binary File:
The Program and Data values are both stored in a single file for loading into the system. The Program Segment must come first, then the Data Segment. The system will look for a three letter header, PRG for program and DAT for data. After the header is a two byte value representing the length of the segment. The system loads the memories with the bytes from the file in sequence starting from address 0x0000.

Here's an example hex dump of the hello world program stored in the proper format:

```
50 52 47 00 0F 26 12 00 0A D1 00 40 10 00 00 28 0A D1 00 FF 44 41 54 00 0E 48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 00
```
