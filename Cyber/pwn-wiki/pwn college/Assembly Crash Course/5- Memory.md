
Registers are expensive

If we need, we may need to store excess data in memory

# memory: Process Perspective
Process memory is used for a lot:

memory -> registers
memory -> disk
memory -> network
memory -> video card

Process memory is addressed linearly

from 0x10000
to 0x7fffffffffffffff

this represents 127 terabytes of ram

each memory address references one byte in memory

# A process' memory
We don't use 127 terabytes, we use virtual memory

memory starts out partially filled in by the OS
- from 0x10000 to 0x7fffffffffffffff
	- Program Binary Code
	- Dynamically Allocated Memory
		- managed by libraries
		- this is the heap
	- Library code
	- Process Stack
	- OS Helper regions
- process can request more memory to be mapped in
	- 'i want memory at this region'


Process can ask for more memory from the Operating system

# Memory (Stack)
Temporary data storage
allocated when process starts uo

registers can be **push**ed onto the stack to save values
stack is last in first out

**mov rax, 0xc001ca75**
**push rax**
**push 0xb0bacafe** (warning: even on 64 bit x86 you can only push 32 bit immediate)
**push rax**
(these leave the src register intact)

Values can be **pop**ped back off the stack to any register
**pop rbx** sets rbx to 0xc001ca75
**pop rcx** sets rcx to 0xb0bacafe

*the bytes are still there after popping a value off the stack, we just redefine were the base and rsp is!*

### Addressing the Stack
CPU knows where the stack is because it's address is stored in rsp
typically 0x7f followed by garbage

stack grows backwards toward smaller memory addresses
**push** *decreases* rsp, **pop** *increases* it

# Accessing Memory

Can move data between registers and memory with **mov**

this will load 64 bit value stored at address 0x12345 into rbx
mov rax, 0x12345 - *This says move this to rax and use it as a memory address*
mov rbx, [rax]  - *brackets mean this is a memory address, LOOK IN this address for the value*

this will store 64 bit value in rbx into memory address 0x133337
mov rax, 0x133337
mov [rax], rbx - *takes data at rbx and stores it in the address at rax (0x133337)*

This is the equivalent to push **rcx**:
sub rsp, 8
mov [rsp], rcx

Each addressed memory location contains 1 byte
An 8-byte write at address 0x133337 will write to address 0x133337 through 0x13333f


# Controlling write sizes

Can use partials to store/load fewer bits

load 64 bits from addr 0x12345 and store the lower 32 bits to addr 0x133337

mov rax, 0x12345
mov rbx, [rax]
mov rax, 0x1333337
mov [rax], ebx

Store 8 bits from ah to addr 0x12345
mov rax, 0x12345
mov bh, [rax]


# Memory Endianess

Data on most modern systems is stored backwards in *little endian*

mov eax, 0xc001ca75
mov rcx, 0x10000
mov [rcx], eax
mov bh, [rcx] reads 0x75

0xc001ca75 is read as 0x75 0xca 0x01 0xc0


# Address Calculation

Allows you to do address calculation

Use rax as an offset from some base address (in this case, the stack)
mov rax, 0
mov rbx, [rsp+rax*8] - read a qword right at the stack pointer
inc rax
mov rxc, [rsp+rax*8] - read the qword to the right of the previous one

you can get the calculated address with Load Effective Address (lea)
mov rax, 1
pop rcx
lea rbx, [rsp+rax*8+5] rbx now holds the computed address for double checking
mov rbx, [rbx]

Address calculation has limits
reg+reg*(2 or 4 or 8)+value is as good as it gets

# RIP - Relative Addressing

**lea** is one of the few instructions that can directly access the rip register
**lea** rax, [rip] load the address of the next instruction into rax
**lea** rax, [rip+8] the address of the next instruction plus 8 bytes

You can also use mov to read directly from those locations
mov rax, [rip] load 8 bytes from the location pointed to by the address of the next instruction

or even right there
mov [rip], rax write 8 bytes over the next instruction

# Writing immediate values

Can write immediate values but must specify their size

This writes a 32 bit 0x1337 padded with 0bits to address 0x133337
mov rax, 0x133337
mov DWORD PTR [rax], 0x1337

(assembler might expect dword or dword ptr)

# Other memory regions
mmap
malloc
These cause other regions to be mapped as well
