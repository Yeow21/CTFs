For launching programs from Python, we recommend using [pwntools](https://docs.pwntools.com/en/stable/intro.html), but [subprocess](https://docs.python.org/3/library/subprocess.html) should work as well. If you are not using one of these two, you will suffer heavily when you get to input redirection (for that, check out the stdin and stdout arguments to `pwn.process` or `subprocess.Popen`).

For reading and writing directly to file descriptors in bash, check out the `read` and `echo` builtins.

You will find the `env` command useful, and the `exec` bash builtin.

Quick refreshers on `fork()` versus `exec*()` [here](https://www.geeksforgeeks.org/difference-fork-exec/) and [here](https://linuxhint.com/linux-exec-system-call/) and [here](https://iximiuz.com/en/posts/how-to-on-processes/). 

Remember to wait() on your children! If you use subprocess.Popen, pwn.process, or good old fork(), your parent will keep executing and, unless it waits for the child in some way, will just terminate! This is almost never what you want.

Some documentation on networking in C.

Useful resource for [pipes in C](https://jameshfisher.com/2017/02/17/how-do-i-call-a-program-in-c-with-pipes/).

Useful resource for [FIFOs in C](https://www.geeksforgeeks.org/named-pipe-fifo-example-c-program/).

A treatise on I/O redirection in Linux shells, which has applications in this assignment: https://web.archive.org/web/20220629044814/http://bencane.com:80/2012/04/16/unix-shell-the-art-of-io-redirection/
A guide on Linux symbolic links. https://www.nixtutor.com/freebsd/understanding-symbolic-links/

A video tutorial on FIFOs in C.

A great visual guide to I/O redirection in Linux.

An incredible pwntools cheatsheet by a pwn.college student!

A deep dive into the history and technology behind command line terminals.