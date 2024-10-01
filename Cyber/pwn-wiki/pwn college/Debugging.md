
GDB is a very powerful dynamic analysis tool which you can use in order to understand the state of a program throughout
its execution. You will become familiar with some of gdb's capabilities in this module.

We write code in order to express an idea which can be reproduced and refined. We can think of our analysis as a program
which injests the target to be analyzed as data. As the saying goes, code is data and data is code.

While using gdb interactively as we've done with the past levels is incredibly powerful, another powerful tool is gdb
scripting. By scripting gdb, you can very quickly create a custom-tailored program analysis tool. If you know how to
interact with gdb, you already know how to write a gdb script--the syntax is exactly the same. You can write your
commands to some file, for example `x.gdb`, and then launch gdb using the flag `-x <PATH_TO_SCRIPT>`. This file will
execute all of the gdb commands after gdb launches. Alternatively, you can execute individual commands with `-ex
'<COMMAND>'`. You can pass multiple commands with multiple `-ex` arguments. Finally, you can have some commands be
always executed for any gdb session by putting them in `~/.gdbinit`. You probably want to put `set disassembly-flavor intel` in there.

Within gdb scripting, a very powerful construct is breakpoint commands. Consider the following gdb script:

  start
  break *main+42
  commands
    x/gx $rbp-0x32
    continue
  end
  continue

In this case, whenever we hit the instruction at `main+42`, we will output a particular local variable and then continue
execution.

Now consider a similar, but slightly more advanced script using some commands you haven't yet seen:

  start
  break *main+42
  commands
    silent
    set $local_variable = *(unsigned long long*)($rbp-0x32)
    printf "Current value: %llx\n", $local_variable
    continue
  end
  continue

In this case, the `silent` indicates that we want gdb to not report that we have hit a breakpoint, to make the output a
bit cleaner. Then we use the `set` command to define a variable within our gdb session, whose value is our local
variable. Finally, we output the current value using a formatted string.


### Modifying program behaviour

GDB is a very powerful dynamic analysis tool which you can use in order to understand the state of a program throughout
its execution. You will become familiar with some of gdb's capabilities in this module.

As it turns out, gdb has FULL control over the target process. Not only can you analyze the program's state, but you can
also modify it. While gdb probably isn't the best tool for doing long term maintenance on a program, sometimes it can be
useful to quickly modify the behavior of your target process in order to more easily analyze it.

You can modify the state of your target program with the `set` command. For example, you can use `set $rdi = 0` to zero
out $rdi. You can use `set *((uint64_t *) $rsp) = 0x1234` to set the first value on the stack to 0x1234. You can use
`set *((uint16_t *) 0x31337000) = 0x1337` to set 2 bytes at 0x31337000 to 0x1337.

Suppose your target is some networked application which reads from some socket on fd 42. Maybe it would be easier for
the purposes of your analysis if the target instead read from stdin. You could achieve something like that with the
following gdb script:

  start
  catch syscall read
  commands
    silent
    if ($rdi == 42)
      set $rdi = 0
    end
    continue
  end
  continue

This example gdb script demonstrates how you can automatically break on system calls, and how you can use conditions
within your commands to conditionally perform gdb commands.

In the previous level, your gdb scripting solution likely still required you to copy and paste your solutions. This
time, try to write a script that doesn't require you to ever talk to the program, and instead automatically solves each
challenge by correctly modifying registers / memory.


Automatically break on system calls then set rdi to 0 (this will change to read from file descriptor 42 which might be for example a web socket to 0 which is stdin)

```
  start
  catch syscall read
  commands
    silent
    if ($rdi == 42)
      set $rdi = 0
    end
    continue
  end
  continue
```


### level 6:

hacker@debugging-refresher~level7:~$ cat gdb-script.gdb 
start
run
break *main+686
commands
        printf "setting rax...\n"
        set $rdx = $rax
        continue
end
continue

### level 7

```
Temporary breakpoint 1, 0x00005630fd221a86 in main ()
(gdb) call (void)win()
You win! Here is your flag:
pwn.college{ktR9KUMhIMHNXY_3kPeQq1JasgC.0FM1IDLzITM4UzW}
```


### level 8:

Unable to use call (void)win() as they broke win!

```
   0x000055659ee9d969 <+24>:    mov    eax,DWORD PTR [rax]
   0x000055659ee9d96b <+26>:    lea    edx,[rax+0x1]
   0x000055659ee9d96e <+29>:    mov    rax,QWORD PTR [rbp-0x8]
   0x000055659ee9d972 <+33>:    mov    DWORD PTR [rax],edx
```

win+24 and win+33 are the broken ones, use set $rip = <next address> to ensure we skip broken instructions.