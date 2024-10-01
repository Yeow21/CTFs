
#### Lifecycle

1. A process is created
2. Cat is loaded
3. Cat is initialized
4. **Cat is launched**
5. **Cat reads its arguments and environment**
6. **Cat does it's thing**
7. **Cat terminates**


### How does execution begin?

#### Cat is launched

A normal ELF automatically calls __libc_start_main() in libc which in turn calls the programs main() function

#### Entry point:

This is where loader will redirect execution once libs are loaded etc

### Cat reads arguments and environment

`int main(int argc, void **argv, void **envp);`

Processes entire input from the outside world comprises of 
- The loaded objects
- command-line arguments in argv
- environment in envp

Of course, programs need to keep interacting with the outside world

### Environment variables
ways of passing information to a program

They change behaviour of normal utility

### Cat does it's thing
### Using lib functions

The binary's import symbols have to be resolved using the libraries' export symbols.

In the past, this was an on-demand process and carried great peril

In modern times, this is all done when the binary is loaded and is much safer

[[Interacting with binaries]]


### Interacting with the env

**syscalls**
`man syscalls`

`strace` - trace process system calls

can call syscalls with a number
`long syscall(number)`
To trigger it, pass the number and that can trigger a systemcall. e.g read is systemcall 0

`n=syscall(0, args)` - read
`n=syscall(0, args)` - write

[Look up - ryan a chapman - linux syscall table](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/)


### System calls

There are over 300 system calls in linux:

`int open(const char *pathname, int flags)` Returns a  file descriptor of the open file
`ssize_t read(int fd, void *buf, size_t count)` reads data from the file descriptor
`ssize_t write(int fd, void *buf, size_t count)` writes data to the file descriptor
`pid_t fork()` forks off an identical child process
`int execve(const char *filename, char **argv, char **envp)` replaces your process

##### Typical signal combinations
- fork, execve, wait (think: a shall)
- open, read, write(cat)


### Signals
Syscalls are a way for a process to call into the OS, what about the other way around

*enter* Signals. Relevant System calls:

`sighandler_t signal(int signum, sighandler_t handler)` register a signal handler
`int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact)` more modern way of registering a sig handler
`int kill(pid_t pid, int sig)` Send a signal to a process

signals pause process execution and invoke the handler
handlers are functions that take one arg: The signal number
Without a handler for a signal, the default action is used (kill)
SIGKILL (signal 9) and SIGSTOP (signal 19) cannot be handled


### Shared memory

Another way of interacting with the outside world is by sharing memory with other processes

Requires system calls to establish but once established, comms happen without system calls

Easy way: Use a shared memory mapped file in /dv/shm


### Cat terminates

Happens one of two ways
1. Recv unhandled signal
2. calling the exit() system call: `int exit(int status)`
All processes must be "reaped":
- After termination they will remain in a zombie state until they are waited on by their parent
- when this happens, their code will be returned to the parent and the process will be freed
- if their parent dies without waiting on them, they are reparented to PID 1 and will stay there until they are cleaned up