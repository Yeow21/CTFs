
`ssh bandit0@bandit.labs.overthewire.org -p 2220`

password: `ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If`
ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If
ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If



## level 0-> 1

`ssh bandit1@bandit.labs.overthewire.org -p 2220`

pass: ``ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If``

use `mktemp -d` to create a temp file with a random hard to guess name

## level 1 -> 2

use `cat ./-` to get the password in the dashed file name

password: `263JGJPfgU6LtdEvgfWU1XP5yac29mFx`

## level 2 -> 3

open a file with spaces in filename

`cat spaces\ in\ this\ filename`

password: `MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx`


## level 3 -> 4

use `ls -la ` to show hidden dir

password: `2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ`


## level 4->5

a set of files called -file

use `file -f filename` command


`4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw`


## level 5 -> 6


use ls -la to match human readable, 1033 bytes in size and noexec

was in dir 07. using file again gives this pass:


for next level:
`ssh bandit5@bandit.labs.overthewire.org -p 2220
`HWasnPhtq9AVKe0dmk45nxy20cvUa6EG