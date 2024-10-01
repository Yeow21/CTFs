

### Memory access width

`single byte: mov [rax], bl`
`2 byte: mov [rax], bx`
`4 byte: mov [rax], ebx`
`8 byte: mov [rax], rbx`

Sometimes you might need to specify size to avoid ambiguity. This is important when using literals

`single byte: mov BYTE PTR [rax], 5`
`2 byte: mov WORD PTR [rax], 5``
`4 byte: mov DWORD PTR [rax], 5`
`8 byte: mov QWORD PTR [rax], 5`


### Forbidden bytes

Depending on the injection method, certain bytes might not be allowed

strcopy can be a problem

| **Byte (Hex Value)**        | **Problematic Methods**                   |
| --------------------------- | ----------------------------------------- |
| `Null byte \0 (0x00)`       | `strcpy`                                  |
| `Newline \n (0x0a)`         | `scanf`, `gets`, `getline`, `fgets`       |
| `Carriage return \r (0x0d)` | `scanf`                                   |
| `Space (0x20)`              | `scanf`                                   |
| `Tab \t (0x09)`             | `scanf`                                   |
| `DEL (0x7f)`                | `Protocol specific (e.g., telnet, VT100)` |

strcopy will terminate if your shellcode has a 0x00.
scanf terminates on newline etc so shellcode cannot contain 0x0d

### More than one way to pwn noobs

Convey values creatively

For example, below, literals are being added to a 64 bit address will will contain a ton of null bytes.

| **Bad**                 | **Good**                                        |
| ----------------------- | ----------------------------------------------- |
| `mov rax, 0`            | `xor rax, rax`                                  |
| `mov rax, 5`            | `xor rax, rax; mov al, 5`                       |
| `mov rax, 10`           | `mov rax, 9; inc rax`                           |
| `mov rbx, 0x67616c662f` | `mov ebx, 0x67616c66; shl rbx, 8; mov bl, 0x2f` |
note: `shl rbx, 8` - Shift rbx to the left by 8 bits then add 0x2f to the lsb


### meaning within meaning

if the constraints on your shellcode are too hard to get around with clever synonyms, but the page where your shellcode is mapped is writable

remember: code == data!

Bypassing a restriction on int3:
int3 == 0xcc. Let's increment .byte by 1 to make 0xcb == int3
	`inc BYTE PTR [rip]`
	`.byte 0xcb`

When testing this make sure .text is writable:
`gcc -Wl, -N --static nostdlib -o test test.s`

### Try and try again
Sometimes, very complex constraints on shellcode which make it hard to do anything useful

Solution: **Multi stage shellcode**

#### Stage 1:
`read(0, rip, 1000).`
- getting your current instruction pointer might be hard depending on architecture
- on amd64, you can do it with `lea rax, [rip]`
- a read like this will overwrite the rest of your shellcode with unfiltered data
#### Stage 2:
a good stage-1 shellcode is very short and simple, letting you get around complex requirements. Unfortunately, you cannot always inject more shellcode


### Shellcode mangling

Shellcode might be mangled beyond recognition

Examples
- Your shellcode might be sorted
- might be compressed or uncompressed
- might be encrypted or decrypted
Start from what you want your code to look like when executed
parts of shellcode might be uncontrollable, you can jump over something to avoid them

### What good is shellcode when you are unable to speak?

Normally, shellcode will just give you a shell

what if there is no way to output the data

what other ways can you use to communicate the flag?


### Useful tools

[[pwntools]] - lib for writing exploits and shellcode
rappel - lets you explore the effects of instructions
amd64 opcode listing [amd64 opcodes](http://ref.x86asm.net/coder64.htm)
gdb plugins - [pwndbg]


