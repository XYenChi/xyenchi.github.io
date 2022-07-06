---
title: "RVFA课程学习笔记"
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
The R-type instruction format is the only one to encode three different registers.   
Encode 3 registers, 0 immediate, 2 minor opcode fields, 1 major opcode field.   
***I*** type:   
  "Immediate" type - this format is used by arithmetic, logic, and shift operations with immediates, jump and link register, environment calls, and LOAD instructions. (have the longest immediate bits, 11 bits, from 20 to 31)   
not the only instruction to encode immediate but R type.   
1. generic register-immediate instruction   
    12 bits immediate, 5 bits rs1, 3 bits funct3, 5 bits rd and 7 bits opcode.   
2. generic LOAD instruction   
    12 bits immediate, 5 bits rs1, 3 bits funct3, 5 bits rd and 7 bits opcode.   
3. JALR instruction   
    12 bits immediate, 5 bits rs1, 3 bits funct3, 5 bits rd and 7 bits opcode.   
4. FENCE instruction   
    4 bits fm, 1 bit PI, PO, PR, PW, SI, SO, SR and SW, 5 bits rs1, 3 bits funct3, 5 bits rd and 7 bits opcode.   
5. Environment instruction   
    12 bits immediate, 5 bits rs1, 3 bits funct3, 5 bits rd and 7 bits opcode.
    **EBREAK** returns control to the debugging environment.   
    **ECALL** requests privileged information to the execution environment in ways defined by the applicable EEI.    
6. pseudoinstructions   
    NOP, LI, MV, SEXT.W, SEQZ, The FENCE instruction without arguments is an alias for FENCE iorw iorw, JR, JALR and RET.   
***S*** type:    
  "Store" type - this format is used by the STORE instructions.    
same structure as B    
***B*** type:    
  "Branch" type - this format is used by the BRANCH instructions.   
same structure as S   
***U*** type:    
  "Upper Immediate" type - this format is used by instructions that use upper immediates (LUI and AUIPC).   
same structure as J   
20 bits immediates, 5 bits rd and 7 bits opcode.   
AUIPC rd imm "add upper immediate to PC" stores the value of PC in rd and adds the shifted imm to it.   
LUI rd imm "load upper immediate" set rd to the shifted imm.   
***J*** type:    
  "Jump" type - this format is used by the JAL (jump and link) instruction.    
same structure as U   
JAL rd imm "jump and link":   
  save PC+4, the return address associated with the JAL instruction, in rd.   
  Add imm to PC.   
#### summary    
The J-type instructions have the same structure as the U-type instructions, only differing by how the immediate is encoded.

### register   
More: RISC processors usually have more than eight general-purpose registers.   

### memory   
Memory access is limited to Load and Store instructions.   
#### virtual memory   
1. single address space   
2. divided into pages, 4 kilobytes long sections of continuous memory.   
3. isolates software running above it requiring addresses to be translated by the operating system.   
### feature   
modular: base(I must exist) + extensions   

### spec
ISA specifications:    
priviledged + unprivileged   
  unprivileged: Base ISA and ISA extensions   
  privileged: interrupts, exceptions, virtual memory management, and physical memory protection etc   

### software stack   
ABI(specification) -> application   
SBI(specification) -> supervisor(application)   
HBI(specification) -> hypervisor(application)   
AEE   
SEE   
HEE   

### RISC-V ISA privilegde mode   
**M** (machine) mode   
**S** (supervisor) mode   
**U** (user) mode   

### traps   
#### contained trap   
Handled by the **software that raise it**.   
Execution resumes after resolution.   

#### requested trap  
request to execution environment.  

#### invisible trap   
raised and handled by the execution environment.   
not visible by the software running inside it.   

#### fatal trap   
cause the execution environment to stop exeception.   

### CSR   
#### Acronyms   
**WPRI**: write preserve, read ignore.   
**WLRL**: write legal, read legal.   
**WARL**: write any, read legal.   
#### Machine Level ISA   
##### mstatus   
register.   
always 64 bits wide. RV32 machine implement it as two separate CSRs (mstatus and mstatush)   
**xIE**: x mode **I**nterrupt **E**nabled   
**xPIE**: x mode **P**revious **I**nterrupt **E**nabled   
**xPP**: x mode **P**revious  **P**rivilege    
x can be M or S mode   
##### mip   
**M**achine **I**nterrupt **P**ending WARL register   
##### mie   
**M**achine **I**nterrupt **E**nable WARL register   
##### mepc   
**M**achine **E**xception **P**rogram **C**ounter WARL register   
##### mcause   
**M**achine **C**ause WLRL register   
##### mtval   
**M**achine **T**rap **Val**ue register   
##### mconfigptr   
**M**achine **Config**uration **P**oin**t**e**r**    
address
##### I-type format instructions to enable trap handling    
**MRET**: M mode return   
**WFI**: Wait for Interrupt   
#### Supervisor level   
##### sstatus, sip, sie, scause   
same as mstatus, mip, mie, mcause.   
##### sepc, stval
same as mepc and mtval, except they contain virtual rather than physical memory addresses.   
##### satp
**S**upervisor **A**ddress **T**ranslation and **P**rotection register has no Machine Level equivalent    
it is used for memory address translation.   
**MODE**   
**ASID**   
**A**ddress **S**pace **ID**entifier   
**PPN**   
**P**hysical **P**age **N**umber   
#### Hypervisor   
#### memory model   
##### RVWMO   
**R**ISC-V **W**eak **M**emory **O**rdering   
##### PMP   
**P**hysical **M**emory **P**rotection   


### Assembly
#### interrupt handler   
__attribute\__((interrupt("machine")))   
__attribute\__((interrupt("supervisor")))   
#### call convention   
register **s0** is callee register, the function that is called should be responsible for (re)storing it.   

### GCC   
#### compiler flags   
##### code model
**-mcmodel**:   
-mcmodel=medlow:   
address range is within 2 GiB in the absolute range between -2 GiB and +2 GiB   
-mcmodel=medany:   
an address range within 2 GiB but position-independent   
**linker relaxation**:   
used by the linker to efficiently load symbol addresses.   
-mrelax:enable linker relaxations   
-mno-relax:disable linker relaxations   

### LLVM   
### operate system   
#### AMO
**R-type** instruction format.   

atomic swap instrution can be used to atomically acquire and release locks.   
atomic load-and-reserve and store-conditional instructions can be used to implement semaphores.   

#### High-Level Atomic API   
##### Mutex   
A mutex is a lock that provides mutual exclusion to a shared resource.   
A spinlock is a lock that uses busy waiting to synchronize access to a shared resource.    
##### Semaphores   
Data structure. A semaphore is one synchronization mechanism used in most OSes to control access to shared resources.    

##### Monitors
##### Send/Recv
#### Low-Level Atomic Ops   
##### Load/store
##### Interrupt disable/enable
##### Test&Set
##### Other atomic instructions
#### Interrupts(I/O), Multiprocessors and CPU scheduling
#### Synchronization   
Load-linked/store-conditional(**LL/SC**):   
Generally, LR will return the current value of an address, and a subsequent SC to the same address will store a new value only if no updates have occurred to the same address since the last LR.   
Load-reserved/store-conditional(**LR/SC**):   
If the SC.W **succeeds**, the instruction writes the word in rs2 to memory, and it **writes zero** to rd.    
If the SC.W **fails**, the instruction does not write to memory, and it **writes a nonzero value** to rd.
### general purpose operate system
##### SMAP   
Supervisor Mode Access Prevention   
SMAP is a hardware-based feature that prevents the kernel from accessing user-level memory.   
##### SMEP   
Supervisor Mode Execution Prevention   
Unlike SMAP, which can be configured, SMEP is always enabled in RISC-V when paged virtual memory is enabled.   
### tips      
1. The OpenRISC ISA is the first open ISA.   
2. There are instruction set specifications for 32-bit and 64-bit address spaces, but the RISC-V design allows for **any bit-width** address spaces.   
3. AUIPC+JALR or LUI+JALR combo make absolute addressing achievable, so a program can unconditionally jump to any absolute address in the whole memory address space. not recommend.   
4. RISC-V support **hypervisor nesting**(running a hypervisor on top of another hypervisor) with the Hypervisor extension.   
