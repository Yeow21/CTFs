Overwhelms senses with a lot of digits.

difficult to compute without writing out
decimals round dont align well to binary round numbers
we can use base 8 (octal) or base 16(hex)

hex numbers are prefixed with 0x

### Expressing numbers
x86 - can use specialised hardware to crunch 512 bits at a time
64 bit machine can reason 64 bits at a time
**Minimum:** 0b0
**1337**: 0x539
 **n bit number** = 2^n numbers
**maximum**: 0xffffffffffff

If adding 1 to maximum number
overflow maximum value
carry bit get stored in CPU and rest of computation becomes 0

#### expressing negatives 
2's compliment
- left most bit describes if it's positive or negative
	- This gives positive and negative 0, bad news!
- Twos compliment:
	- negative numbers are represented by the largest positive numbers they would correlate to
	- 0b11111111 == 255 == -1
	- 0b11111110 == 254 == -2
	- This way, computations don't have to be sign aware
	- This halves the smallest expressible negative number
	- 0b10000000 == -128
### expressing text
Encoding:
- ascii (evolved into UTF 8)
	- specifies chats using 7 bits
	- Upper case letters:
		- 0x40 + letter index in hex
	- Lower case
		- 0x60 + letter index in hex
	- digits
		- 0x30 + digit
	- Control chars
		- 0x09 - tab
		- 0x20 - space
		- 0x0a - new line
		- 0x07 - bell

## Grouping Bits into Bytes
Standard sized grouping - 8 bits is 1 byte
This is tied to text encoding 

## Grouping Bytes into words
**nibble** - half of a byte
**Byte** - 8 bits
**Half word / word** - 2 bytes
**Double word** 4 bytes
**Quad word** 8 bytes

## Word anatomy
- left most bit = most significant bit
- right most bit = least significant bit
