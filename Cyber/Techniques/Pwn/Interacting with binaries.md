`gcc` to make your elf

`file cat` - show info about the binary
`readelf` to parse the header
`objdump` to parse the header and disassemble the source code
`nm` to view your elf's symbols
- `nm -D cat`
	- show imported symbols
- `nm -a cat`
	- show all symbols
`patchelf` - to change some ELF properties
`objcopy` - to swap out elf sections
`strip` - to remove helpful information such as symbols
`kaitai struct` -  to look through your elf interactively

www.intezer.com/blog/.../section segments, symbols, relocations, dynamic linking
