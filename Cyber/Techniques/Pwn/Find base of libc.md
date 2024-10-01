
### Examples:
[[yawa]]


### resources
cryptocat videos


When dealing with a binary exploitation scenario, one common technique is to use a libc leak to gain the address of a function in libc, calculate the base address of libc, and subsequently call any desired function within libc, such as `system`. Here's how to do this:



### Steps to Find the Base Address of libc

#### Leak a Known Function's Address:

Identify and exploit a vulnerability in the binary (e.g., format string vulnerability, buffer overflow) to leak the address of a known function from libc, such as puts, printf, or fgets.
Identify the Offset of the Known Function:

Use tools like [[readelf]] or [[objdump]] to find the offset of the known function within libc. This offset is crucial for calculating the base address of libc.

#### Calculate the Base Address of libc:

Subtract the offset of the known function from its leaked address to determine the base address of libc. This calculation is straightforward:

`base_address = leaked_function_address - offset_of_function_in_libc`

#### Understanding the Output of `ldd`

The `ldd` command is used to print the shared object dependencies of a binary. Here's an example output:

```sh
─$ ldd yawa
  linux-vdso.so.1 (0x00007ffff7fc9000)
  libc.so.6 => ./libc.so.6 (0x00007ffff7c00000)
  ./ld-linux-x86-64.so.2 => /lib64/ld-linux-x86-64.so.2 (0x00007ffff7fcb000)

`└─$ readelf -s ./libc.so.6 | grep "system"
  `1481: 0000000000050d70    45 FUNC    WEAK   DEFAULT   15 system@@GLIBC_2.2.5

Offset from base of libc to system: `0x50d70`

#### finding `/bin/sh`

`─$ strings -a -t x ./libc.so.6 | grep "/bin/sh" 
`-> 1d8678 /bin/sh