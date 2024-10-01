
# Computers make decisions

`if (authenticated) 
{`
	`leetness =1337;`
`}`
`else`
`{`
	`leetness = 0;`
`}`

# What to execute?

assembly instructions are just data

variable width architecture. Some instructions are 1 byte, the add instruction below is 3 bytes

| program in binary | pop rax | pop rbx | add rax, rbx | push rax |
| ----------------- | ------- | ------- | ------------ | -------- |
| program in hex    | 58      | 5b      | 48 01 d8     | 50       |

These assembly instructions are literally just hex data

**jmp** skips x bytes and then resumes execution
	mov cx, 1337
	jmp STAY_LEET
	mov cx, 0
	STAY_LEET:
	psh rcx
The above program is:
The label is for the assembler to use
jmp - adds to ip
**so eb** = jump
**04** = bytes
**fe** = jump back to itself (negative 2)



| program binary code | 0x400800          | 0x400804             | 0x400806   | 0x40080a |
| ------------------- | ----------------- | -------------------- | ---------- | -------- |
|                     | mov rcx, 0x1337   | jmp STAY_LEET        | mov rcx, 0 | push rcx |
|                     | 66 b9 ***37 13*** | eb 04 (skip 4 bytes) | 66 b9 00   | 51       |

# Control flow: Conditional jumps


| assembly instruction | description                             |
| -------------------- | --------------------------------------- |
| je<br>jne            | jump if equal/not equal                 |
| jg<br>jl             | jump greater or less than               |
| jle<br>jge           | jump less / greater or equal            |
| ja<br>jb             | jump if above/below (unsigned)          |
| jae<br>jbe           | jump if above/below or equal (unsigned) |
| js<br>jns            | jump if signed/unsigned                 |
| jo<br>jno            | jump if overflow/not overflow           |
| jz<br>jnz            | jump if 0/not 0                         |
|                      |                                         |

For conditional jump - you compare things first THEN take action

**Conditional jumps check conditions stored in the flags register**
- flags are updated by
	- most arithmetic instructions
	- comparison instruction cmp
	- comparisons instruction test 
- Main conditional flags
	- Carry flag: was the 65th bit1?
	- zero flag: was the result 0?
	- Overflow Flag: did the result wrap between positive to negative
	- Signed flag: was the result signed bit set? (was it negative)
- Common patterns
	- cmp rax, rbx; ja STAY LEET - *unsigned rax > rbx 0xffffffff >= 0*
	- cmp rax, rbx; jle STAY_LEET - *signed rax <= rbx. 0xffffffff = -1 < 0*
	- test rax, rax; jnz STAY_LEET - *rax != 0*
	- comp rax, rbx; je STAY_LEET - *rax == rbx*

# Looping

loop that counts to 10:

mov rax, 0
LOOP_HEADER:
inc rax
cmp rax, 10
jb LOOP_HEADER

# Control flow: Function calls

Assembly code is split into functions with **call** and **ret**

**call** pushes **rip** and jumps away
**ret** pops **rip** and jumps to it

Using a function that takes an authenticated value and returns leetness

| Assembly                                                                                                                                                                                                                        | c                                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| mov rdi, 0<br>call FUNC_CHECK_LEET<br>mov rdi, 1<br>call FUNC_CHECK_LEET<br>call EXIT<br><br>FUNC_CHECK_LEET:<br>	test rdi, rdi<br>	jnz LEET<br>	mov ax, 0<br>	ret<br>	LEET:<br>	mov ax, 1337<br>	ret<br>	<br>FUNC_EXIT<br>	??? | `int check_leet(int authed) {`<br>`if (authed) return 1337;`<br>`else return 0;`<br>`}`<br><br>`int main() {`<br>`check_leet(0);`<br>`check_leet(1);`<br>`exit();}` |

# Calling conventions

Callee and caller functions agree on argument passing
linux x86 push arguments then call return value in eax
linux amd64:rdi, rsi, rdx, rcx, r8, r9 return value in rax
linux arm: r0, r1, r2, r3, return value in r0

registers are shared between functions so calling conventions should agree on what registers are protected

linux amd64
rbx, rbp, r12, r13, r14, r15 are callee saved (the function you call keeps their values safe on the stack).
other registers are up for grabs
but rsp must be maintained