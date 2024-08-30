


| RELRO      | STACK CANARY | NX        | PIE        | RPATH   | RUNPATH   | Symbols     | FORTIFY | Fortified | Fortifiable | FILE    |
|------------|--------------|-----------|------------|---------|-----------|-------------|---------|-----------|-------------|---------|
| Full RELRO | Canary found | NX enabled| PIE enabled| No RPATH| RW-RUNPATH| No Symbols  | No      | 0         | 4           | imgstore|


```
$ file imgstore 
imgstore: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter ./ld-linux-x86-64.so.2, BuildID[sha1]=31a60496d15b054df8aa179ff5f4d553f2700265, for GNU/Linux 3.2.0, stripped
```
#### Reflection

Unfortunately I did not have enough time to dedicate to this CTF. This would have been a fun challenge to crack and is one I have been unable to find a writeup for so will return to at some point #todo to figure it out. For when I come back - I will look at leaking PIE base, libc base and I assume use [[ret2libc]]


Difficult - all protections enabled, stripped

Looking through with Ghidra we can find this function:

```
─$ void FUN_00101e2a(void)

{
  long in_FS_OFFSET;
  char local_61;
  uint local_60;
  int local_5c;
  char local_58 [72];
  long canary;
  
  canary = *(long *)(in_FS_OFFSET + 0x28);
  local_5c = open("/dev/urandom",0);
  read(local_5c,&local_60,4);
  close(local_5c);
  local_60 = local_60 & 0xffff;
  do {
    printf("Enter book title: ");
    fgets(local_58,0x32,stdin);
    printf("Book title --> ");
    printf(local_58);
    puts("");
    if (local_60 * 0x13f5c223 == DAT_00106050) {
      DAT_0010608c = 2;
      FUN_00101d77(2);
    }
    puts(
        "Sorry, we already have the same title as yours in our database; give me another book title. "
        );
    printf("Still interested in selling your book? [y/n]: ");
    __isoc99_scanf(&DAT_001038a7,&local_61);
    getchar();
  } while (local_61 == 'y');
  puts("");
  printf("%s[-] Exiting program..%s\n",&DAT_00103200,&DAT_00103009);
  sleep(1);
  if (canary != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}
```

This function is called when we select the 'sell book' option and we can see a format string vuln quite quickly:

```
    fgets(local_58,0x32,stdin);
    printf("Book title --> ");
    printf(local_58);
```



To be continued...