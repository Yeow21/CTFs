
### Static tools:

*static* - reverse engineering at rest. 

### Simple tools:

**kaitai struct** - [File parser and explorer](https://ide.kaitai.io)
**nm** - List symbols used/provided by ELF files 
**strings** - Dumps ascii strings found in a file
**objdump** - analyses sec features used by an executable [[objdump]]


### Advanced Dissassemblers

#### Commercial
**IDA Pro** - Gold standard of dissassemblers
**Binary Ninja** - IDA's main competitor

#### Free
**Binary Ninja Cloud** - A version of Binary ninja that runs in browser

**Open source**
**Angr management** - An academic binary analysis framework
**Ghidra** - A reversing tool created by the NSA
**Cutter** - A reversing tool created by the radare2 open source project


### getting started with Binary Ninja

Variables will be determined by the disassembler and given names e.g:
`var_108` - This is where it is located in relation to the start of the function. before this, there was `push rbp` which is 0x100, so var_108 is the next one based on it's location. These can be renamed.

Binary ninja can do High level analysis - this will try to recover as close to C as possible. Can give almost source code.

Can be dangerous as it might abstract details.

Use assembly view first.