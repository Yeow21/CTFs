

### using /bin/cat as the example for this lecture


### Lifecycle

1. **A process is created**
2. **Cat is loaded**
3. **Cat is initialized**
4. Cat is launched
5. Cat reads its arguments and environment
6. Cat does it's thing
7. Cat terminates


### Portrait of a process
Every Linux process has:
- State
	- running? waiting for resources? zombie?
- Priority
	- including other scheduling info
- Parent, siblings and children
	- Process has a parent process that created it and children processes it created
- shared resources
	- files piles and sockets
- virtual memory space
	- can do whatever it wants in that memory state
- security context
	- effective uid and gid
	- saved uid and gid
	- capabilities

### where do processes come from

- Mitosis!
	- system call that creates a nearly exact copy of the calling process, a parent and child
		- This creates a *parent* and *child* System calls are:
			- Fork
			- Clone
				- Child processes uses the `execve` syscall to replace itself with another process
- Example:
	- You type `/bin/cat` in bash
	- Bash *forks* itself into the old parent process and the child process
	- The child process `execves /bin/cat` becoming `/bin/cat`

### Can we load?
Before it is loaded - the kernel checks for exec permissions. If not executable, `execve` will fail


### What to load
To figure out what to load, it will read the beginning of the file
1. if file starts with `#!`, kernel extracts the interpreter from the rest of the line and executes this interpreter.
2. If file matches a format in `/proc/sys/fs/binfmt_misc`
	1. this is kernel subsystem that figures out how to execute files
	2. The kernel executes the interpreter specified for that format with the original file as an argument
3. If file is dynamically linked ELF, the kernel reads the interpreter/loader defined in the ELF, loads the interpreter and the original file and lets the interpreter take control
4. If the file is statically linked, the kernel will load it
5. Other legacy file formats are checked for
**These can be recursive!**


### Dynamically linked ELFs: the interpreter

Process loading is done by the ELF interpreter specified in the binary
`readelf -a /bin/cat | grep interpret`
This will give the program interpreter/loader

Can be overridden:
`/lib64/ld-linux-x86-64.so.2 /bin/cat /flag`

Can be changed permanently
`patchelf --set-interpreter <interpreter>`

### Dynamically linked ELFs: The loading process
1. Program and interpreter are loaded by the kernel
2. Interpreter locates libraries
	1. LD_PRELOAD environment variable
		1. Set lib to run and override functions of future loaded libraries
			1. 
	2. LD_LIBRARY_PATH env variable
		1. set path and every library that cat depends on will be loaded from here
	3. DT_RUNPATH or DT_RPATH
		1. `patchelf --set-rpath /some/runpath`
			1. will look for libs here
	4. system-wide configuration (/etc/ld.so.conf)
	5. /lib and /usr/lib
3. The interpreter loads the libraries
	1. these libraries can depend on other libs causing more to be loaded
	2. relocations updated
		1. updates pointers to show where other libs have been loaded


### Where does it get loaded into?

- Each process has virtual memory state. It contains
	- The binary
	- The libraries
	- The heap (for dynamically allocated memory)
	- The stack (for functional local variables)
	- Any memory specifically mapped by the program
	- some helper regions
	- the kernel code in the upper half of memory
- *Virtual* memory is dedicated to your process
- *Physical* memory is shared among the whole system
- You can see this whole space by looking at `/proc/self/maps`


### The standard C library
**What is libc?**
Library full of helper functions that's used by almost every process.
- printf
- scanf
- malloc
- puts
- socket
- atoi
- free
- xdr_keystatus

### The loading process: Statically linked binaries

Essentially, libc included with the compiled process (i think?)

Much bigger binaries
No external libraries mapped in when using `./cat-static /proc/self/maps`

#### Why not ship everything statically linked?
- File size
	- much bigger
- rust - mostly supports static
	- can load and use c libs but rust are all static
- less security opportunities
	- you can't know where libs are loaded by knowing where cat is loaded in dynamically linked


### Initialization

Every ELF binary can specify constructors which are functions that run before the program is launched. 

e.g libc can initialize mem regions for dynamic allocations (malloc/free) when the program launches

#library-injection
You can specify your own!

```
__attribute__((constructor)) void haha()
{
	puts("Hello world!");
}
```
Demo: LD_PRELOAD and constructors

Usage:
`gcc -shared -o start_main.so start_main.c`
`LD+PRELOAD=./start_main/.so ./cat cat.c`


