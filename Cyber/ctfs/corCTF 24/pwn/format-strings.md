

## introduction
pwn/format-string

>ryaagard
>
31 solves / 199 points
>
This is not a good challenge and you will learn nothing. There is no need to search for obscure format specifiers, the intended way is to use a very widely used specifier
>
`nc be.ax 32323`

### reflection

I didn't get this challenge and couldn't get past the limit of only allowing a 3 wide input for the format string vuln. A writeup which I haven't read properly yet suggested % * x would use the ptr in the address before as the width giving you space to leak the libc address then using the system offset to pass that address to the do_call() function. #todo If i get a chance!

### Preliminary checks:

```
└─$ file chal 
chal: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=3c364891d9e85f75b8d800e45ed24149ec52ff7a, for GNU/Linux 3.2.0, not stripped
```

| RELRO      | STACK CANARY | NX         | PIE         | Symbols    | FILE |
| ---------- | ------------ | ---------- | ----------- | ---------- | ---- |
| Full RELRO | Canary found | NX enabled | PIE enabled | 74 Symbols | chal |

Tricky - all protections enabled. This will be a challenge for me.

Source code:
```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

void do_printf()
{
    char buf[4];
    if (scanf("%3s", buf) <= 0)
        exit(1);
    printf("Here: ");
    printf(buf);
}

void do_call()
{
    void (*ptr)(const char *);
    if (scanf("%p", &ptr) <= 0)
        exit(1);
    ptr("/bin/sh");
}

int main()
{
    int choice;
    setvbuf(stdin, 0, 2, 0);
    setvbuf(stdout, 0, 2, 0);
 
	while (1)
    {
        puts("1. printf");
        puts("2. call");
        if (scanf("%d", &choice) <= 0)
            break;

        switch (choice)
        {
            case 1:
                do_printf();
                break;
            case 2:
                do_call();
                break;
            default:
                puts("Invalid choice!");
                exit(1);
        }
    }
    return 0;
}
```

Looking through the source code, it looks like we need to pass the address of `system` to the `do_call()` function.

let's write a quick fuzzer to see what we can leak from the stack using the format string vuln in `do_printf()`

