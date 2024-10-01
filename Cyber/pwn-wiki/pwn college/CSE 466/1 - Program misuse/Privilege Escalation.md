
### Linux permission model

Every process has a userID and GID

Every file and dir is owned by a user and group

Child processes inherit from parent processes

`UID 0` - this is root or Linux administrator


### Privelege Elevation


Run an *suid binary*

**SUID**: execute with the eUID of the file owner
**SGID**: Execute with the eGID of the file owner rather than parent process
**Sticky**: Used for shared directories to limit file removal to file owners

When a file is run and setuid bit is set, the process executes with the eUID of the file owner rather than parent process. Same with eGID

### *e*UID

effective vs real vs safe uid

eUID used for access control check
real ID - true identity of the process. e.g sending signals to a process, this will be checked
saved id - used to temporarily drop privileges. Usually leads to sec vulnerabilities.

`sudo chmod u+s <file>` - To the user (u) add (+) the setuid bit (s)

```
int main() { printf("UID: %d\n", geteuid()); }
```

`gcc -w -o getuid getuid.c`

`sudo chown root.root getuid`
`sudo chmod u+s getuid`
`ls -l getuid`


```
┌──(kali㉿kali)-[~/Documents]
└─$ sudo chown root:root getuid
                                                                            
┌──(kali㉿kali)-[~/Documents]
└─$ ./getuid 
UID: 1000                                                                                                                  
┌──(kali㉿kali)-[~/Documents]
└─$ sudo chmod u+s getuid                                                                                                                         
┌──(kali㉿kali)-[~/Documents]
└─$ ./getuid             
UID: 0                                                                                  
```


### Responsibility
eUID 0 is powerful. Root can:

**Open** any file
- things in the /proc filesystem
- any device backed file
**Execute** any program
**Assume** any other UID or GID
**Debug** any program
**Read** the memory of any file

### Privilege Escalation Exploit

*Exploit in which the attacker elevates their privileges to root level*

- Typical flow:
	- Gain a foothold on the system (Vulnerable network service, intended shell access, code in app context etc)
	- Identify a vulnerable privileged service
	- exploit the privileged service to gain it's privileges
- E.g if an SUID binary has a security problem, an attacker can use privilege escalation attack


### practice problems

	1.Connect to pwn.college
	2 Select a program path to unnecessarily SUID
		3 Tons of programs can be chosen
	4 Use program to read / flag

scripting:
https://g0ldf15h.github.io/posts/pwn_college/program_misuse/#23-genisoimage
https://beefwhale.github.io/posts/pwn.college-program-misuse/




#### opening a program with python:

```
with open('/flag', 'r') as f:
    content = f.read()
    print(content)
```

#### With perl:
```    
#!/usr/bin/perl
use strict;
use warnings;

my $file = '/flag';

open my $fh, '<', $file or die "Cannot open $file: $!";

while (my $line = <$fh>) {
    print $line;
}

close $fh;
```

#### With Ruby:

```
File.open('/flag', 'r') do |file|
  content = file.read
  puts content
end
```

#### With bash
`bash -p /flag`
https://stackoverflow.com/questions/63689353/suid-binary-privilege-escalation
This tells bash to not set up an environment and run with what it's got instead

```
file="/flag"

if [[ -f "$file" ]]; then
    while IFS= read -r line; do
        echo "$line"
    done < "$file"
fi
```