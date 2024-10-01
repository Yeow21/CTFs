We will be building the server in assembly.

1)
Use the exit() function to close the program

assembly:
`mov rdi, 0
`mov rax, 60
`syscall

2)
create a socket() - This is an endpoint for communication that returns a file descriptor if successful. File that represents an A point and B point which are linked. An Endpoint for communication.

`int socket()
``	int domain
``	int type
``	int host
    #create socket (AF_INET, TCP, 0)
        mov rdi, 2
        mov rsi, 1
        xor rdx, rdx
        mov rax, 41
        syscall
        mov rdi, rax
        
use POSIX standards to find the constant associated with domains etc

#### x86 64 calling convention
https://www.ired.team/miscellaneous-reversing-forensics/windows-kernel-internals/linux-x64-calling-convention-stack-frame
- Arguments 1-6 are passed via registers RDI, RSI, RDX, RCX, R8, R9 respectively;


3)
	bind socket - assigns address specified by addr to the socket referred to by file descriptor sockfd

`int bind()`
	`int sockfd - result of socket system call`
	`struct sockaddr *addr`
	`socllen_t addrlen`
	