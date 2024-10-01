
## summary

In summary, pwninit helps streamline the initial setup and preparation process for binary exploitation tasks, allowing the user to focus more on the actual exploitation rather than the preliminary configuration.



https://github.com/io12/pwninit/blob/master/README.md

## Usage


#### Short version
Run `pwninit`

#### Long version
Run `pwninit` in a directory with the relevant files and it will detect which ones are the binary, libc, and linker. If the detection is wrong, you can specify the locations with --bin, --libc, and --ld.


```
┌──(kali㉿kali)-[~/Documents/LITCTF 24/pwn/infinite echo]
└─$ pwninit                      
bin: ./main2
libc: ./libc.so.6
ld: ./ld-2.31.so

unstripping libc
https://launchpad.net/ubuntu/+archive/primary/+files//libc6-dbg_2.31-0ubuntu9.16_amd64.deb
warning: failed unstripping libc: libc deb error: failed to find file in data.tar
copying ./main2 to ./main2_patched
running patchelf on ./main2_patched
writing solve.py stub
```

