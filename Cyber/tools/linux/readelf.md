


Used in pwn to find offsets of functions 

`─$ ldd yawa                                             
  `linux-vdso.so.1 (0x00007ffff7fc9000)
  `libc.so.6 => ./libc.so.6 (0x00007ffff7c00000)
`./ld-linux-x86-64.so.2 => /lib64/ld-linux-x86-64.so.2 (0x00007ffff7fcb000)

`└─$ readelf -s ./libc.so.6 | grep "system"
  `1481: 0000000000050d70    45 FUNC    WEAK   DEFAULT   15 system@@GLIBC_2.2.5Us
  