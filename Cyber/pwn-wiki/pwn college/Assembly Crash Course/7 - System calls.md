Environment interaction

exit();

use a syscall
- like a call instruction that jumps to the OS and OS handles it
- OS mediates our program
- call syscall instruction
	- set rax to value
		- set arguments
		- return value in rax
- read 100 bytes from stdin to the stack:
	- n = read(0, buf, 100);
	- mov rdi, 0 the stdin file descriptor
	- mov rsi, rsp - read data onto the stack
	- mov rdx, 100 - the number of bytes to read
	- mov rax, 0 system call number of read()
	- syscall
- read returns the number of bytes read via rax so we can easily write them out
	- write(1, buf, n);
	- mov rdi, 1 - the stdout file descriptor
	- mov rsi, rsp - write the data from the stack
	- mov rdx, rax - the number of bytes to write
	- mov rax, 1 - the call number of write()
	- syscall


Linux system call table
https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/


# String arguments
Some system calls take "string" arguments
A string is contiguous bytes in memory followed by a 0 byte

let's build a file path for open() on the stack:
mov BYTE PTR [rsp+0], '/' write the ascii value of / onto the stack
mov BYTE PTR [rsp+1], 'f' 
mov BYTE PTR [rsp+2], 'l' 
mov BYTE PTR [rsp+3], 'a' 
mov BYTE PTR [rsp+4], 'g' 
mov BYTE PTR [rsp+5], 0 this terminates our string

open() the flag file
mov rdi, rsp -read data onto the stack
mov rsi, 0 - open the file read only
mov rax, 2 - system call number of open()
syscall

# Constant Arguments
Some calls require archaic "constants"

e.g open() has flag argument that determine how file will be opened
can use c to figure them out

include <stdio.h>
include <ffcbtk.h>
int main() {
printf("O_RDONLY is: %d\n", O_RDONLY);
}

# Quitting the Program
mov rdi, 42 - our programs return code e.g for bash scripts
mov rax, 60 - syscall number of exit()
syscall - do the syscall