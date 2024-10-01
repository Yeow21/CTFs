Level 1:
To interact with any level you will send raw bytes over stdin to this program.
To efficiently solve these problems, first run it to see the challenge instructions.
Then craft, assemble, and pipe your bytes to this program.

For instance, if you write your assembly code in the file asm.S, you can assemble that to an object file:

  as -o asm.o asm.S

I used the command mov rdi, 0x1337 to set the rdi register then used the command above to assemble

