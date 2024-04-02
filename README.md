# RARC-API
API of my architecture, RARC, from Turing Complete on steam. Read the API in the architecture file. It's very messy and un-organized.
The actual terms come around at line 70. Remember that it's very non-understandable as I never really entered this project with a plan.

RARC ARCHITECTURE
- Uses two RAMs along with an HDD to save and load values, along with doing operations on them.

Output 1 from program is OPCODE #1
Output 2 is ADDRESS 1
Output 3 is ADDRESS 2
Output 4 is DESTINATION

The OPCODE can determine what to do with the values, if to save/load between register and RAM, and if to save/load between RAM and HDD.

API

Code Format:
OPCODE Arg1 Arg2 DEST/ADDRESS


OPCODE

The OPCODE determines the level of communication between storage modes, whether or not the ALU/COND is being used, the IM values, and the operation/condition.

The last two bits determine level of communication.
(128) (64)

  0    0............ I/O - Registers
  1    0............ RAM - Registers
  0    1............ HDD - RAM
  1    1............ Unused

I/O-Reg and RAM-Reg may use the ALU and the COND. The HDD must transfer to the ram first before using the ALU/COND. Only I/O-Reg may use FUNCT, if needed in the first place.

The 3rd last bit(/last 3 bits) determines the ALU/COND/FUNCT usage.

(128) (64) (32)

  x    x    0...... ALU
  x    x    1...... COND
  1    1    1...... FUNCT

Explanation: The ALU can perform operations on 8-bit inputs, for example dividing, adding, subtracting, NOTing, etc. The COND determines whether or not the first argument is equal to, greater than, etc. to the second argument, and outputs an on/off signal. Functions use the STACK in order to have the call and ret commands used.


The 4th and 5th bigs determine using IM values. IM values are values taken directly from the program, not any register or input.

(16) (8)

 0    0......... NONE
 0    1......... ARG2
 1    0......... ARG1
 0    0......... BOTH

The first 3 bits determine the operation/condition to use. The 3rd-last bit must be ON to use condition.

(4) (2) (1)

 0   0   0....... ADD / =
 0   0   1....... SUB / /=
 0   1   0....... AND / <
 0   1   1....... OR  / <=
 1   0   0....... NOT / >
 1   0   1....... XOR / >=
 1   1   0....... DIV / N/A
 1   1   1....... UNUSED

*NOT only takes Arg1

OPCODE TERMS
*By default, all OPCODE terms deal with the I/O-Register level. Add another term before for RAM-Register level.
rh....... Deals with the RAM-HDD level. There is a special set of terms for this, specified below.
copy..... Copy a value from one register to the other. Arg2 is replaced with the term "to"
write.... Write a value to a register/RAM. Arg2 is replaced with the term "to"
imArg1... Use to write an immediate value on Arg1. Use by adding +imArg1 to any ALU or COND term.
imArg2... Use to write an immediate value on Arg2. Use by adding +imArg2 to any ALU or COND term.
funct.... Use when dealing with a function in scripts. Addon call or ret depending on if you are returning or calling a function.
call..... Use when calling a function. Arg1 and Arg2 should be blank. Dest/Address should be the line # or label name to jump to.
ret...... Use when returning from a function. Arg1, Arg2, and Dest/Address should be blank.

(alu).... The following are terms used with the ALU:
addi..... Adds an immediate value Arg1 and a regular Arg2 together. (equivalent to add+imArg1)
add...... Add Arg1 and Arg2.
sub...... Subtract Arg2 from Arg1.
div...... Divide Arg1 by Arg2.
and...... Bitwise AND Arg1 and Arg2.
not...... Bitwise NOT Arg1.
or....... Bitwise OR Arg1 and Arg2.
xor...... Bitwise XOR Arg1 and Arg2.

(cond)... The following are terms used with the COND:
JET...... Output 1 if Arg1 is equal to Arg2.
JNE...... Output 1 if Arg1 is not equal to Arg2.
JLT...... Output 1 if Arg1 is less than Arg2.
JLE...... Output 1 if Arg1 is less than or equal to Arg2.
JGT...... Output 1 if Arg1 is greater than Arg2.
JGE...... Output 1 if Arg1 is greater than or equal to Arg2.
JMP...... Output 1 at all times. Example command: JMP _ _ pastFunct

Arguments (outputs 2/3)
Arguments tell the terms used within operations. These can be taken from the HDD, RAM, Registers, or I/O.

Terms:
r0-r5.... Registers 0-5, represented by the numbers 0-5 respectively.
in....... The system input. Represented by 6.
cnt...... The counter. It takes the current line # of the program. Represented by 7. Calling cnt from Arg1 will output the current tick. Calling from Arg2 will output the current byte the program is on.
......... *If you wish to write the program byte to a register, then you must do addi _ cnt (r0-5)
to....... Another term for 0. Use this in Arg2 of write or copy statements.
_........ Placeholder for nothing. Use this when nothing is to be added for the argument (ex. in funct statements)

Dest./Address (output 4)
This is the distination for any line of code used. This can be to a register, I/O, RAM, or HDD.

Terms:
r0-r5.... Registers 0-5, represented by the numbers 0-5 respectively.
all...... This outputs to all registers. Only use on the DEST/ADDRESS argument. Useful for clearing all registers.
out...... System output. Represented by 6.
cnt...... The counter. It sets the result of Arg1 and Arg2 to the counter. Mostly only used on Register/IO level. Represented by 7.
_........ Placeholder for nothing. Use this when nothing is to be added for the argumetnt (ex. in funct statements)

THE STACK
You can use the stack to temporarily store values in the order you input. Any stack commands must be added onto funct. Pushing a value adds a value to the top of the stack. Popping a value pulls a value from the top of the stack.

Terms:
stck... Defines whether or not you are using a stack. This should be part of the OPCODE.
psh.... Pushes a value onto the stack. Add this onto the OPCODE.
pll.... Pulls a value from the stack, to the destination of your choosing. Add this onto the OPCODE.

When pushing onto the stack, Arg1 is the value you are pushing to the stack. Leave all other arguments blank. The push value can only be from a register. When pulling from the stack, the DEST/ADDRESS is the destination of the value you have gotten from the stack. Leave all other arguments blank.

RAM-REGISTER INFO

When using RAM - register layer, bits 1, 2 and 3 no longer determine operation. Bit 3 now determines whether or not you are using RAM 1 or RAM 2. Bit 6 determines whether or not you are saving/loading. Bits 1 and 2 do nothing.

Functions also cannot be used while dealing with RAM.

RAM-related terms:
ra1...... This defines working with RAM 1. Must be added onto the OPCODE.
ra2...... This defines working with RAM 2. Must be added onto the OPCODE.
rread.... This defines reading from the RAM. This must be added onto the OPCODE. When reading from RAM, the RAM ADDRESS is Arg1. Leave Arg2 blank. The destination is DEST/ADDRESS.
rwrite... This defines writing onto the RAM. This must be added onto the OPCODE. When writing, the register to write to is Arg1. DEST/ADDRESS is the RAM ADDRESS. Leave Arg2 blank.
rim1..... This determines if you are using Arg1 as the immediate input. If not, Arg1 is the register/input to write from Leave Arg2 blank.
sTck..... Onle to be used with rwrite, put this in the DEST/ADDRESS bit to write the address to the current tick

*as of now, the RAM address can only be taken from a register. More functionality may or may not be added as needed.

Six example lines of RAM reading and writing are found here.

rwrite+rim1+ra1 24 _ r5.... This writes 24 to the RAM on the address of R5.
rwrite+ra2 in _ r3......... This writes the input to the RAM on the address of r3.
rwrite+ra1 r2 _ r4......... This writes r2 to the RAM on the address of r4.

rread+rim1+ra1 24 _ r3..... This reads the address of 24 from the RAM to r3.
rread+ra2 r5 _ r1.......... This reads the address of r5 on the RAM to r1.
rread+ra1 r2 _ out......... This reads the address of r2 on the RAM to the output.

HDD USAGE
When using the HDD-RAM level, The whole OPCODE is used in a different way. Bit 6 determines read/write. While writing to HDD, it takes the RAM address of Arguments 1 and 2, and also all of the registers, and combines it into a 64-bit byte to put into the HDD. The DEST/ADDRESS is the HDD address you would like to go to. (0-255) 
(32)
 0......... WRITE
 1......... READ

Terms
hddwrite..... The function to write something to the HDD.
hddshift..... Shifts the place of the HDD, as according to DEST/ADDRESS. Normally use it to shift back after writing (shift it by [255-<write shift #>]+1)
hddread...... The function to read something from the HDD (or allocate from the HDD)
fullRead..... This option sets the HDD to read address value from r0 and output it to 8 storage registers. From there, you can allocate or trash them.
allocate..... This option allows you to allocate a value to any destination (using the dest/address argument)
v0-7......... Add this option onto the hddread to specify which value you wish to allocate.
trash........ Add this on to the end of an allocation argument to trash the value.
clear........ This term clears the current byte the HDD is on. Add this onto the OPCODE.

BASIC FUNCTIONS

write 0 to r0
funct+call _ _ clearRAML

label clearRAML
rwrite+rim1+ra1 0 _ r0
rwrite+rim1+ra2 0 _ r0
add+imArg1 1 r0 r0
JET+imArg1 255 r0 clearRAMD
JNE+imArg1 255 r0 clearRAML

label clearRAMD
funct+ret _ _ _

This function clears both RAMs at once. Takes a while, however, and it will delete register 0.
-------------

funct+call _ _ writeHDD

label writeHDD
hddwrite x x x
write x to all
hddshift _ _ x
write x to r0
rwrite+ra1 r0 _ x
write 0 to all
hddwrite 0 

This function writes RAM data to the HDD and saves the RAM data's spot in the HDD.  (The X's are values that should be changed)
-------------

funct+call _ _ readHDD

label readHDD
hddread+fullRead _ _ 0
hddread+allocate+v0 _ _ r0
rwrite+ra1 r0 _ _
hddread+fullRead _ _ r0
hddshift _ _ 242

This function writes HDD data gathered from a designated storage indicator slot to the HDD reserved registers, to be allocated or trashed.
-------------

write 0 to r0
funct+call _ _ clearHDD
JMP _ _ pastFunct

label clearHDD
hddwrite+clear _ _ _
hddshift _ _ 1
addi 1 r0 r0
JET+imArg2 r0 255 cHDDD
JNE+imArg2 r0 255 clearHDD

label cHDDD
hddshift _ _ 1
funct+ret

label pastFunct
# code here

This function fully clears the hard drive, although it takes a solid 2 minutes. No way around that.
-------------
