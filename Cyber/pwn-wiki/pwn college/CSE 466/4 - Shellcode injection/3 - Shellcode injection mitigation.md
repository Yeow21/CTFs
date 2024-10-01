

### The no-eXecute bit
Modern architectures support permissions
- PROT_READ - allows process to read memory
- PROT_WRITE - allows process to write
- PROT_EXEC - allows process to execute
Normally all code is in /text segments of loaded ELF files
There is no need to execute code on stack or heap

by default, stack and heap are NOT executable

### A simpler shellcode

the rise of NX has made shellcoding rarer
- it's an ancient art practiced by wise hackers!
- Embedded devices dont have NX always
- Also in General purpose landscape

### Remaining injection points - deprotecting memory


Memory can be made executable.
Syscall `mprotect()` systel call:
	This requires high level control to be able to exec.

ROP can be used
1. Trick the program into mprotect(PROT_EXEC) our shellcode
2. Jump to shellcode

Using ROP


### Remaining: JIT compilers

Allows you to run JS very quickly

- JIT compilers generate code that is executed
- pages must be writable for code generation
- pages must be executable for code generation
- pages must be executable for execution
- pages must be writable for code-regeneration

```
mmap(PROT_READ|PROT_WRITE)
write code
mmap(PROT_READ|PROT_EXEC)
execute
mmap(PROT_READ|PROT_WRITE)
update code
etc
```

THIS IS EXTREMELY SLOW
JIT is meant to be FAST
Slow and safe loses to FAST

writable AND executable pages are common
if your binary uses a lib that has a writable + executable page, that page is in your memory space

#### What if JIT is safe?

JIT spraying:

- Make constants in the code that will be JITed
- the JIT engine will mprotect(PROT_WRITE), compile the code into memory then MPROTECT(PROT_EXEC). Your constant is now present in executable memory
- Now, use a vulnerability to redirect execution to the constant
