[[UAF]]
### Overview

This exploit revolves around reusing dynamically allocated memory once it has already been freed. Once reallocated, this memory has a new pointer with the original one pointing to a position within the freed memory. 

[ctf101 explains it here with a good](https://ctf101.org/binary-exploitation/heap-exploitation/)

In the example linked above - it shows a basic UAF bug with memory being allocated, and then freed, with another reallocation with a 2nd pointer being returned after. calling puts with the first pointer then shows the data entered from the 2nd memory allocation. If during the 2nd memory allocation, malicious code is written then when the first pointer is used again, it could cause that unspecified behaviour.

### defense:

Make sure that pointer is set to NULL once freed:

```
if (ptr) {
    free(ptr);
    ptr = NULL;
}
```