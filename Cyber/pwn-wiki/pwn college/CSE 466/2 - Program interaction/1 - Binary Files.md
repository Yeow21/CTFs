

https://kaitai.io/
Web based IDE for looking at structured files such as exe, png, zip etc
### What is an elf file?

Executable and Linkable format
- Defines a program as it will be loaded and executed in memory
- stores data about a program
	- what architecture?
	- other os
		- PE
		- MacO
- All similar
	- allow a compiler to create and define a program

### what does it contain?
- How it should be loaded and executed
- what the data is

### program headers
- Define segments
	- part of an elf file loaded into memory
- All elfs start with bytes 
	- 7f 45 4c 46
	- This spells ELF
	- Then
		- defines a bunch of stuff
		- defines entry point.
- headers:
	- define segments loaded into memory
		- offset filesize
			- position relative to binary entrypoint and size
		- VirtAddr memsiz
			- Where in memory
			- memory size
		- PhysAddr
			- flags
				- R - Read
				- E - Executable
				- W - Writable
					- These show permissions
	- Load
	- INTERP
		- The library that is going help load the file

### Section headers
- Represent a different view into the elf file
- Lot's more semantic info
	- less important for loading process
- These define what the segments have inside them
	- from a meaning perspective
	- only really important to reversing/debugging tools
- Sections
	- .text
		- executable code
		- assembled assembly code
	- .plt, .got
		- resolve library calls
		- find and store the function in the GOT
	- .data
		- 3 types of data sections
			- .data
				- global writable data that has an initial value
				- e.g hard coded array
			- .rodata
				- similar but read only data
				- e.g hard coded strings
			- .bss
				- used for storing uninitialized writable data
					- array defined that doesn't have initial values
					- there is no reason for it to take space
					- It's allocated with all 0's

### Symbols
- Names that rely to constructs of other library such as functions and variables exported by libraries
- used to find libs of correct versions
	- https://intezer.com/blog/malware-analysis/executable-linkable-format-101-part-2-symbols/

`readelf -a /usr/bin/cat` 


### interacting with your ELF

`gcc` to make your elf

`readelf` to parse the header
`objdump` to parse the header and disassemble the source code
`nm` to view your elf's symbols
`patchelf` - to change some ELF properties
`objcopy` - to swap out elf sections
`strip` - to remove helpful information such as symbols
`kaitai struct` -  to look through your elf interactively

www.intezer.com/blog/.../section segments, symbols, relocations, dynamic linking
