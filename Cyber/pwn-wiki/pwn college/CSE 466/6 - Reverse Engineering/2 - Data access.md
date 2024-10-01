
### Accessing data on the stack

Accessed via:
- push (store data on top/left of the stack)
- pop ( to retrieve data from the top/left of the stack)
- rsp-relative accesses
	- load: `mov rdx, [rsp+0x10]
	- store `mov [rsp+0x10], rdx`
	- offsets are positive because rsp points to the left edge of the function frame
- rbp-relative access:
	-  load: `mov rdx, [rbp-0x10]
	-  store `mov [rbp-0x10], rdx`
	- offsets are negative because rbp points to the right edge of the frame
- via reference
	- `mov rdx, rsp; mov [rdx], 0x41;`
	- `mov rdx, rbp; mov [rdx], 0x41`


```
 |    rsp    |
 |- - - - -  | 
 | rbp-0x10  |
 | rsp+0x8   |
 |- - - - -  | 
 | rbp-0x8   |
 | rsp+0x10  |
 | - - - - - | 
 |    rbp    |
 | - - - - - |
 |    ret    |
```


### Accessing data in ELF sections

data is .bss. .rodata and .data is stored at known offsets from the program code.

Is accessed via rip - relative instructions

- load `mov rax, [rip + 0x20040]`
- store `mov [rip + 0x20040], rax`
- reference `lea rax, [rip + 0x20040]`

### Accessing data on the heap

Pointers to heap data re stored in memory or on the stack

#### stack data access:
`mov rax, rsp`
`mov rdx, [rax]`

#### Arbitrary data access via stack-stored reference
`mov rax, [rsp]`
`mov rdx, [rax]

### Data structures

Type info is lost in the compilation process.

Must be pieced together.

This is practice, problems must be tacked repeatedly to learn.