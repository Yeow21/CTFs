
program consists of:
- Consists of modules
	- used to make devs lives easier
	- read documentation instead of reversing
- made up of functions
	- well defined goal
		- get
		- set
	- well suited to be reversed in isolation
- contain blocks
	- blocks of code that are executed together
	- every execution of a function is a path through a graph
- of instructions
- that operate on variables and data structures



#### Functions
Contain a prologue and epilogue

Recall: ELF contains 3 sections:
1. .**data**
	1. used for pre init global writable data
2. .**rodata**
	1. read on global data
3. .**bss**
	1. used for uninitialized global writable data such as global arrays without initial values

Local variables are stored on the stack
##### prologue:
Sets up the stack frame

##### epilogue
Tears down stack frame

#### The stack

Grows backwards.

When you **push** to the stack, **rsp** is decreased by 8
when you **pop** from the stack, **rsp** is increased by 8

#### The stack - Initial layout
Starts out storing the env variables and the program arguments:
```
 | STACK   | 
 | - - - - | 
 | ARGC    | # NUMBER OF ARGUMENTS
 | - - - - | 
 | argv[0] | 
 | - - - - | 
 | argv[1] | 
 | - - - - | 
 | NULL    | 
 | - - - - | 
 | envp[0] | 
 | - - - - | 
 | envp[1] | 
 | - - - - | 
 | envp[2] | 
 | - - - - | 
 | NULL    | 
 | - - - - | 
 | argv[0] | DATA
 | - - - - | 
 | argv[1] | DATA
 | - - - - | 
 | envp[0] | DATA
 | - - - - | 
 | envp[1] | DATA
 | - - - - |
 | envp[2] | DATA
 | - - - - | 
```

#### The stack: Calling a function

When a function is called - the address that the called function should return to is implicitly pushed onto the stack

This return address is implicitly popped when the function returns

***Calls don't show pushing the return address to the stack, this is implicit***

### The stack: Function Frame Setup

Every function sets up it's stack frame. It has:

**Stack pointer (rsp)** - Points to the leftmost side of the stack frame
**Base pointer (rbp)** - Points to the rightmost side of the stack frame.

#### Prologue
1. Save off the callers base pointer
2. Set the current stack pointer as the base pointer
3. allocate space on the stack (subtract from the stack pointer)

```
 | STACK   |
 | - - - - | 
 | buffer  |
 | - - - - | 
 |saved rbp| 
 | - - - - |
 | start+3 | 
 | - - - - | 
 | ARGC    | # NUMBER OF ARGUMENTS
 | - - - - | 
 | argv[0] | 
 | - - - - | 
 | argv[1] | 
 | - - - - | 
 | NULL    | 
 | - - - - | 
 | envp[0] | 
 | - - - - | 
 | envp[1] | 
 | - - - - | 
 | envp[2] | 
 | - - - - | 
 | NULL    | 
 | - - - - | 
 | argv[0] | DATA
 | - - - - | 
 | argv[1] | DATA
 | - - - - | 
 | envp[0] | DATA
 | - - - - | 
 | envp[1] | DATA
 | - - - - |
 | envp[2] | DATA
 | - - - - | 
```

### The stack: Function Frame Teardown
#### Epilogue
1. "deallocate" the stack, (`mov rsp, rbp`)
	1. The data is NOT destroyed by default!!! - This leads to vulnerabilities. 
2. Restore old base pointer

now we are ready to ret!