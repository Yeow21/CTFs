CPUs need to be fast
- rapid access to data being processed
- this is done via Register File
- Located within the CPU.

amd64:
- rax
- rcx
- rdx
- rbx
- rsp
- rbp
- rsi
- rdi
- r8
- r9
- r10
- r11
- r12
- r13
- r14
- r15
- Address of next instruction is in a register:
	- eip(x86)
	- rip(amd64)
	- r15(arm)
- typically the size of the word width
### partial register access
**rax** - qword
**eax** - dword
**ax** - half word
ah - most significant nibble of ax
al - least signifcant nibble

right most bytes if a 64 bit register:

| 64  | 32   | 16   | 8H  | 8L   |
| --- | ---- | ---- | --- | ---- |
| rax | eax  | ax   | ah  | al   |
| rcx | exc  | cx   | cd  | cl   |
| rdx | edx  | dx   | dh  | dl   |
| rbx | ebx  | bx   | bh  | bl   |
| rsp | esp  | sp   |     | spl  |
| rbp | ebp  | bp   |     | bpl  |
| rsi | esi  | si   |     | sil  |
| rdi | edi  | di   |     | dil  |
| r8  | r8d  | r8w  |     | r8b  |
| r9  | r9d  | r9w  |     | r9b  |
| r10 | r10d | r10w |     | r10b |
| r11 | r11d | r11w |     | r11b |
| r12 | r12d | r12w |     | r12b |
| r13 | r13d | r13w |     | r13b |
| r14 | r14d | r14w |     | r14b |
| r15 | r15d | r15w |     | r15b |
|     |      |      |     |      |

Setting registers:
**mov** **rax**, 0x539

Move number 1337 into rax (move immediate value to rax register)

**mov** ah 0x5
**mov** al 0x39

**32 bit caveat**
if you write to a 32 bit partial (e.g **eax**) the CPU will zero out the rest of the register!

## Shunting Data around
mov rax 0x539 - Move 0x539 into rax
mov rbx, rax - Move contents of rax into rbx

**mov** actually copies data, old data is not destroyed!

#### Can move partials:
mov rax, 0x539
mov rbx, 0
mov al, bl

this sets rax to 0x539 and rbx to 0x39

## extending data
consider:
**mov** eax, -1
eax is now 0xffffffff - 4294967295 and -1
rax is now 0x00000000ffffffff - only 4294967295
- it would need the most significant bit to be f

To operate on that -1 in 64 bit land:
**mov eax, -1**
**movsx rax, eax**

movsx is a 'sign extending move' preserving two's compliment value This copies the top bit to the rest of the register

## Register Arithmetic

| Instruction   | C / Math equivalent              | Description                                                                 |
| ------------- | -------------------------------- | --------------------------------------------------------------------------- |
| add rax, rbx  | rax = rax + rbx                  | add rax to rbx                                                              |
| sub ebx, ecx  | ebx = ebx - ecx                  | subtract ecx from ebx                                                       |
| imul rsi, rdi | rsi = rsi * rdi                  | multiple rsi to rdi, truncate to 64 bits                                    |
| inc rdx       | rdx = rdx + 1                    | increment rdx                                                               |
| dec rdx       | rdx = rdx - 1                    | decrement rdx                                                               |
| neg rax       | rax = 0 - rax                    | negate rax in terms of numerical value                                      |
| not rax       | rax = ~rax                       | negate each bit of rax                                                      |
| and rax, rbx  | rax = rax & rbx                  | bitwise AND between the bits of rax and rbx                                 |
| or rax, rbx   | rax = rax \| rbx                 | bitwise OR between the bits of rax and rbx                                  |
| xor rcx, rdx  | rcx = rcx ^ rdx                  | bitwise XOR                                                                 |
| shl rax, 10   | rax = rax << 10                  | shift rax bits left by 10 filling with 10 zeroes on the right               |
| shr rax, 10   | rax = rax >> 10                  | shift rax bits right by 10 with sign extension to fill the now missing bits |
| ror rax, 10   | rax = (rax >> 10) \| (rax << 54) | rotate the bits of rax right by 10                                          |
| rol rax, 10   | rax = (rax << 10) \| (rax >> 54) | rotate the bits of rax left by 10                                           |
| xchg rax, rdx |                                  | swap rax and rdx                                                            |

One that isnt on here which should be:
xchg - exchange give it two registers and swaps values

## some are special
rip - address of next instruction
- ip = instruction pointer

rsp - contains the address of a region of memory to store temporary data
- stack pointer