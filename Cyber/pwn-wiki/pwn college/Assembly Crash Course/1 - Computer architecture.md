Fundamentals

Source code - > interpreter or JIT -> CPU
C, C++ or Rust -> compiler -> CPU

Logic gates:
![[Pasted image 20240611200622.png]]

Many logic gates computing to make device run.

4 types of gate:
& gate - if either is true, output is true
XOR gate
OR gate
NOT gate

Input true or not true can be a voltage, optical (light coming in), 

CPU consists of:
- registers
	- Small bits of storage
- Control unit
	- contains instructions and dispatchers
- Arithmatic logic unit (ALU)
	- self explanitory
- Cache
	- fast
		- variable size
			- 8 or 16 bytes
	- Storage
		- L1
		- L2
		- L3
		- etc
		- CPU pulls from cache to put data in registers
- multicore processors
	- each one has their own ALU, Cache, Registers, ALU etc

Von Neumann architecture

Shrinking but increasing speed layers. As cache and registers get closer to CPU.

Computer architecture - caching layers stored on top of each other.
