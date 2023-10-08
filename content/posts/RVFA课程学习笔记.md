---
title: "RVFA课程学习笔记"
date: 2023-10-03T12:59:38+08:00
url: "/RVFA_COURSE/"
draft: false
---
### timeline   
**1980**:   start around, RISC-V is the fifth generation of a research project.   
**2010**:   RISC-V is the result of an evolving project.   
**2011**:   The first version of the RISC-V ISA was released.   

### name   
the number just after "RV":
  ***XLEN***: in the Unprivileged Specification document.
represents ***the width of registers and not the width of the instructions***. Not all instruction width are allowed, only multiple of 16 bits wide.  
for example, the 32 of RV32I.    
***RV64G***:   
RV64IMAFDZicsr_Zifencei   

### intruction   
#### encode   
RV32I/RV64I:   
For 32-bit(or higher) instructions, the first 2 bits will be 11.   
The other three quadrants (00, 01, 10) are used by the 16-bit Compressed Instructions.   
#### length   
RISC-V instructions can be any multiple of 16 bits wide (although most are 32 bits wide) and the information on their length is contained in their major opcode.   
#### major opcode   
the first 7 bits of the instruction, identifying the intruction.   
#### minor opcode  
name funct7 or funct3.    
***note***: LUI, AUIPC and JAL contain no minor opcode.   
***funct7*** occupies the last 7 bits of the R type instruction   
***funct3*** always occupies bits 12 to 14   
#### formats   
instruction type formats defined by the ***encoding of immediate***.   
R, I, S, B, U, J   
***R*** type:   
  "Register" type - this format is used by arithmetic register-register operations such as ADD, SUB, boolean operations, and shifts.   
***I*** type:   
  "Immediate" type - this format is used by arithmetic, logic, and shift operations with immediates, jump and link register, environment calls, and LOAD instructions. (have the longest immediate bits, 11 bits, from 20 to 31)   
***S*** type:    
  "Store" type - this format is used by the STORE instructions.   
***B*** type:    
  "Branch" type - this format is used by the BRANCH instructions.   
***U*** type:    
  "Upper Immediate" type - this format is used by instructions that use upper immediates (LUI and AUIPC).   
***J*** type:    
  "Jump" type - this format is used by the JAL (jump and link) instruction.   

### register   
More: RISC processors usually have more than eight general-purpose registers.   

### memory   
Memory access is limited to Load and Store instructions.   

### feature   
modular: base(I must exist) + extensions   

### spec
ISA specifications:    
priviledged + unprivileged   
  unprivileged: Base ISA and ISA extensions   
  privileged: interrupts, exceptions, virtual memory management, and physical memory protection etc   

### tips      
1. The first OpenRISC ISA is the first open ISA.   
2. There are instruction set specifications for 32-bit and 64-bit address spaces, but the RISC-V design allows for **any bit-width** address spaces.