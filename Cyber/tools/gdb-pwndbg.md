

cheat sheet:
https://pwndbg.re/CheatSheet.pdf


### Useful commands

`info functions` print functions used

`disassemble <function name>` disassemble a certain function

`break *0xfffffff` - set a breakpoint at certain position

`canary` - show the canary

`x/100x $rsp` - print 100 elements from stack pointer

`search -t string "/bin/sh"`

`cyclic <number>` - generate a sequence of chars to input to a binary

`cyclic -l <sequence of bytes>` - Check what point in the cyclic pattern the sequence occurs

`x/bx` - see value in big endian

