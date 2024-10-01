
### level 1:

Find license key:

```
atscs
Initial input:

        61 74 73 63 73 

The mangling is done! The resulting bytes will be used for the final comparison.

Final result of mangling input:

        61 74 73 63 73 

Expected result:

        61 74 73 63 73 

Checking the received license key!

You win! Here is your flag:
pwn.college{0f09ogFW-usldVWepFVyXQuaB9M.0VM1IDLzITM4UzW}
```

### level 1.1

This was a simple case of opening the binary with a disassembler and looking at the value that was compared to your input. in my case, it was `vmmk`

### level 2

Inputs 2&3 are swapped

```
fupjx
Initial input:

	66 75 70 6a 78 

This challenge is now mangling your input using the `swap` mangler for indexes `2` and `3`.

This mangled your input, resulting in:

	66 75 6a 70 78 

The mangling is done! The resulting bytes will be used for the final comparison.

Final result of mangling input:

	66 75 6a 70 78 

Expected result:

	66 75 6a 70 78 

Checking the received license key!

You win! Here is your flag:
pwn.college{AMlGkkNyZrDqR4c49NM5nHdY9ly.01M1IDLzITM4UzW}

```


### level 2.1

Same as above but got to work out what function is being used

reversing shows it swaps the first and `[3]` letter of the array

### level 3

This one clearly reverses the order of your input and reveals what it should be in ascii

`dscxe`
### level 3.1


### level 6:

```
def swap_indexes(data, i, j):
    """Swap elements at indexes i and j in the list."""
    data[i], data[j] = data[j], data[i]
    return data

def xor_mangle(input_bytes):
    print(input_bytes)
    key1 = 0x3e
    key2 = 0xf5
    
    output_bytes = bytearray(input_bytes)
    
    for i in range (0, len(input_bytes)):
        if (i % 2 == 0):
            input_bytes[i] = input_bytes[i] ^ key1
        elif (i % 2 == 1):
            input_bytes[i] = input_bytes[i] ^ key2
            
    print(input_bytes)
    return input_bytes
        
    
    
def reverse_mangling(input_bytes):
    # XOR keys as used in the original mangling process
    KEY1 = 0x3e
    KEY2 = 0xf5

    # Initialize the output list (the reversed mangled bytes)
    output_bytes = bytearray(input_bytes)  # Using bytearray for mutable sequence

    # Iterate through each byte in the input
    for index in range(17):  # The loop runs for index from 0 to 16 inclusive
        # Retrieve the byte value from the input
        if index >= len(input_bytes):
            break
        temp0_1 = input_bytes[index]  # This is the byte at the current index
        temp1_1 = input_bytes[index]  # temp1_1 is used as the same value in the reverse process

        # Calculate the sign extension and result
        rdx_1 = temp0_1 >> 7  # Logical right shift by 7 (instead of 31 in C code because we're dealing with 8-bit bytes)
        rax_23 = ((temp1_1 + rdx_1) & 1) - rdx_1

        # Determine the XOR key based on the result of rax_23
        if rax_23 == 0:
            output_bytes[index] ^= KEY1
        elif rax_23 == 1:
            output_bytes[index] ^= KEY2

    return output_bytes

def reverse_list(data):
    """Reverse the list."""
    return data[::-1]


def print_bytes_as_ascii(byte_list):
    # Convert bytes to ASCII characters
    ascii_chars = ''.join(chr(byte) if 32 <= byte <= 126 else '.' for byte in byte_list)
    return ascii_chars
    
    
# Final result of mangling
final_result = [0x51, 0x90, 0x52, 0x86, 0x58, 0x98, 0x4d, 0x81, 
            0x4b, 0x84, 0x4f, 0x9a, 0x57, 0x8c, 0x51, 0x91, 0x56]

# Apply the reverse steps of mangling in reverse order
# Reverse the list (undo the reverse mangling)
intermediate = reverse_list(final_result)

# Apply XOR with the key (undo the XOR mangling)

intermediate = xor_mangle(intermediate)

# Swap indexes (undo the swap mangling)
# Indexes to swap are 5 and 6
swap_indexes(intermediate, 5, 6)

print("Original input (before mangling):")
print([hex(x) for x in intermediate])

print(print_bytes_as_ascii(intermediate))


```