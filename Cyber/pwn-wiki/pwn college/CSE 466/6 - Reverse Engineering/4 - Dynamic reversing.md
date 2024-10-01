#### Analyse a program at runtime

### Program Tracing

`ltrace` - traces library calls
`strace` - traces system calls

Running the program with multiple inputs might get you further e.g:
- Running a program with input A
- Running again with input B
- See if you can reverse the algorithm from looking at input and output
- Does not scale to complex algorithms


### Debugging

#### GDB:
Best friend or worst enemy

When it runs up, you can use `cat ~/.gdbinit`
Ours should include enabling of gdb debugging environment i.e pwndbg

Others:
`set disassembly-flavor intel`
`set history save on`


`nano cat.gdb`
can set commands to run on breakpoints. 

Check on google how to add conditionals / if statements

for example break after a syscall and print a memory address:


```
disp/5i $rip 

break *address
commands
	x/s $rsp
	c
end
```
This works for static compiled but for PIE binaries of course the address will be a random place in memory.

To run this script:

`gdb -x cat.gdb cat.elf`

Then when you run the script, it will skip to the input at *address and run commands after that.

`printf "READ SYSCALL READ IN %d BYTES", $rax`


### Errata: Dealing with PIE

gdb tries to help by loading them always at 0x000055555554000 or 0x7ffff7ffc000

Easiest way to deal with this is to put this in your .gdbinit:

	set $base = 0x7ffffff7ffc000
Then:
	`break *($base + 0x1023)`

in gdb:
`info proc map` will show where the base address will be


### Timeless debugging

Frees you from having to think of breakpoints ahead of time
1. record execution
2. rewind execution
3. replay execution

#### relevant tools
**gdb** - built in record-replay func
**rr** is highly preformant record replay engine
**qira** timeless debugger made for rev


