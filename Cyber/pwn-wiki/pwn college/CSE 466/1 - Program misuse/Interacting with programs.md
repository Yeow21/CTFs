
## Linux command line

Bandit wargame for learning Linux: https://overthewire.org/wargames/bandit

### Use documentation if you don't know

`man cat` - print the manual page

`help cd` - built in shell documentation

### process

A process is just a running program.

A program is a file on your computer

Files live in a filesystem

Web browser, CLI, Text editor, all start out as files on the filesystem and become processes when they are executed

### The file system

`/` - Root - the anchor of the filesystem
`/usr` - Unix System Resource - contains the system files
`/usr/bin` - Executable files for programs installed on cpu
`/usr/lib` - shared libs for use on cpu
`/usr/share` - Program resources
`/etc` - System config
`/var` - Logs, Caches, etc
`/home` - User-owned data
`/home/ctf` - Data owned by you in the pwn.college infrastructure
`/proc` - runtime process data
`/tmp` - temporary data storage

This is a convention - very configurable though. This is typical

### Directories

Files are stored in dirs

Each process has a current working dir. This can be viewed with `pwd` and can be changed with `cd`

### Specifying paths

**Absolute:** Start with `/` such as `/usr/home/yans` etc
**Relative**: don't start with `/`

. - signifies current dir: `./files/flag.txt`
.. - signifies the dir the current dir resides in: `../files/flag.txt`
**file at the of a path**: reference a file with that name

### Paths to commands

`cat cat` No such file found...

If nothing in front of command - shell will search filesystem for the path variable

#### Environment variables:
Set of K/V pairs passed into every process when launched

**PATH**: A list of dirs to search for programs in
**PWD**: the current working dir
**HOME**: The path to home dir
**HOSTNAME**: The name of your system

`env` print environment vars #todo - Study the man page! [[Cyber/tools/linux/ASLR]]

`export PATH=/usr/bin:/tmp` update environment var using shell syntax

`which` - searches path environment 

### filetypes

`ls -ld` - look at files `-d` allows you to look at dir

`-` regular file
`d` dir
`l` - Symbolic link
`p` - named pipe
`c` - char device file e.g microphone (produces or receives data streams)
`b` - block device file e.g hard drive (stores and loads blocks of data)
`s` - unix socket (local network connection encapsulated in a file)

### Symbolic link

File that references another file

Relative:
`ln -s flags/TOPSECRET link_to_the_flag`

Absolute:
`ln -s /home/yans/flag/TOPSECRET link_to_the_flag`

### Hard link

File references the original file directly by performing magic

`ln flags/TOPSECRET hard_link_to_flag`

### pipes
Mostly not named - ephemeral

One side of the pipe sends data into the other side

named pipes known as FIFO's created using mkfifo command help facilirate data flow

### redirection
output infile to the commands input

`<in_file`: redirect in_file to commands input
`>out_file`: redirect output to out_file, overwriting it
`>>out_file`: redirect to out_file, appending it
`2>error_file`: redirect to cmds error_file overwriting it
`2>>error_file`: redirect to error file, apending it