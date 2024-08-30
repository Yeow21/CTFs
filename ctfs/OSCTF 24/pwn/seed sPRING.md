RANDOMisation successful: Your mission is to jump on the sPRiNG and see how high you can soar. With 30 levels of increasing complexity, you'll need to master the laws of PHYSICS and TIME to reach new heights. Can you crack the code and defy gravity? Let the experiment begin!

Author: @Inv1s1bl3
#32bit #pie-enabled
`nc 34.125.199.248 2534`

![[fake_flag.txt]]

![[seed_spring]]



### tools used

[[ghidra]]
```
└─$ file seed_spring 
seed_spring: ELF 32-bit LSB pie executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=24cf2e205931e9f950290de2fac6cff03e58e3df, not stripped
```

| RELRO      | STACK CANARY    | NX         | PIE         | FILE        |
|------------|-----------------|------------|-------------|-------------|
| Full RELRO | No canary found | NX enabled | PIE enabled | seed_spring |
### Reflection

I ran out of time on this challenge so unfortunately was unable to follow up the writeup.
#todo - I expect it is abusing the format string bug to leak the values needed for the comparison?


interesting - PIE enabled and 32 bit. I've not tried a 32bit challenge before

Decompiled in Ghidra:
```
{
 uint user_input;
  uint random_number;
  uint current_time;
  int level;
  undefined *local_10;
  
  local_10 = &stack0x00000004;
  puts("");
  puts("");
  puts("");
  puts("");
  puts("Welcome! The game is easy: you jump on a sPRiNG.");
  puts("How high will you fly?");
  puts("");
  fflush(_stdout);
  current_time = time((time_t *)0x0);
  srand(current_time);
  level = 1;
  while( true ) {
    if (0x1e < level) {
      puts("Congratulation! You\'ve won! Here is your flag:\n");
      get_flag();
      fflush(_stdout);
      return 0;
    }
    printf("LEVEL (%d/30)\n",level);
    puts("");
    random_number = rand();
    random_number = random_number & 0xf;
    printf("Guess the height: ");
    fflush(_stdout);
    __isoc99_scanf(&DAT_00010c9a,&user_input);
    fflush(_stdin);
    if (random_number != user_input) break;
    level = level + 1;
  }
  puts("WRONG! Sorry, better luck next time!");
  fflush(_stdout);
                    /* WARNING: Subroutine does not return */
  exit(-1);
}



```
