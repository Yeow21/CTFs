
### Sign mixup

`size_t` - unsigned integer
`short, int and long` - signed integers

e.g - negative one stored in two's compliment is equal to 2^32 -1

Signedness matters during comparisons

For example:

```
int main() {
	int size;
	scanf("%i", &size)
	if (size > 16) exit(1);
	read(0, buf, size)
}
```

In above, (`jae` is used for unsigned comparisons)

```
cmp eax, 16; jae if too big
```

Unsigned comparison:
```
0xffffff > 16, jump
```

signed signed comparison (`jge` used for signed comparisons) - 0xffffff = -1 therefore no jump

```
0xffffff > 16, jge too bit
0xffffff = -1 , No jump.
```

##### Note:
`cmp` still sets the same flags. It just depends on which one is used, jge (signed) or jae(unsigned) when determining if to jump or not.

#### WARNING!
using -1 as an input for a signed integer causes GDB to fail. This counts as a very large read (0xffffff) and for some reason (unknown) it will exit with 0

Yan wrote a simple script:

shorted_read.gdb
```
break read
commands
	set $rdx = $rdx & 0xfff
	c
end
```

This breaks at read, and then manually sets the rdx register to hold 0xfff. Setting rdx to 0xffffff is still too big




### Integer overflow

```
int main() {
	unsigned int size;
	scanf("%i", &size);
	char *buf = alloca(size+1);
	int n = read(0, buf, size);
	buf[n] = '\0';
	
}
```

when a large number such as 0xffffff adds one and becomes 0

new function = alloca() - this allocates space on the stack - dynamically sized space. Unsigned. 

So the above unsigned 32 bit int has a maximum value of 0xffffffff - which is equal to 2^32 -1

So in the above example, if you enter 2^32 -1 as the input, alloca(size+1) will allocate 0 bytes then the following read will read 0xffffffff bytes to the stack.


### off by one errors

consider:

```
int a[3] = { 1,2,3};
for ( int i = 0; i <= 3; i++) a[i] = 0
```