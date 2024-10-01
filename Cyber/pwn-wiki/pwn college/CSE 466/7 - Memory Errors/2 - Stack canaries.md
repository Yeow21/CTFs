Principle:

During function prologue - write a value at the end of the stack frame
during the function epilogue - make sure the value is still intact


### Are they effective?

Effective in general:

#### Situational bypass methods

1) leak the canary
2) Brute force the canary
	1) using forking processes
		1) every process on android phone forks from one process.
		2) This canary is only set on the first instance of the process.
			1) On android, every single canary is the same as the parent process. If you leak one, you know them all
3) Jump the canary 



### Brute forcing:

`ipython

For example, in program which forks itself for each input, you can continuously input data without changing the parent process:

`r.write(b'A' * 128)`

What we can do is add padding until the canary is triggered:

`r.write(b'A' * 8)`

`r.write(b'A' * 16)`

`r.write(b'A' * 24)`
`STACK SMASHING DETECTED`

Therefore, the canary must be 24 bytes after our input
`r.write(b'A' * 16 + b'b')
`STACK SMASHING DETECTED`

We know canary starts with null byte:
`r.write(b'A' * 16 + '\0x00)`
`Process forked successfully!`

We know we just overwrote the first byte of the canary!

Now, we have to find the second byte:
`r.write(b'A' * 16 + '\0x00\0x01)`
`STACK SMASHING DETECTED`

(...)

`r.write(b'A' * 16 + '\0x00\0x52)`
`Process forked successfully!`

Now we know the 2nd byte!


### Jumping the canary:

for example:

`for (int i = 0, i < 128, i++) read(0, buf+i,  1)`

If we can overflow i, we can jump to a read point after the canary


### Network forking service: