### **Exercice 04: C Program Analysis**

#### **What value will the following C program display? Why?**

```c
#include <stdio.h>
void main()
{
    char buffer[10];
    char *ptr;
    buffer[0] = 'A';
    buffer[1] = 'B';
    buffer[2] = 'C';
    buffer[3] = 'D';
    ptr = buffer + 2;
    *ptr = 'Z';
    printf("%c %c %c %c \n", buffer[0], buffer[1], buffer[2], buffer[3]);
}
```

**Analysis**:
- `buffer` is an array of 10 chars: `buffer[0] = 'A', buffer[1] = 'B', buffer[2] = 'C', buffer[3] = 'D'`.
- `ptr = buffer + 2` sets `ptr` to point to `buffer[2]` (address of the third element, holding 'C').
- `*ptr = 'Z'` overwrites `buffer[2]` with 'Z'.
- After modification, `buffer` is: `['A', 'B', 'Z', 'D', ...]`.
- According to the fromatting "%c %c %c %c \n" he program prints: `buffer[0] buffer[1] buffer[2] buffer[3] \n`.

**Output:**
```
A B Z D
```

**Why**:
- The assignments set `buffer[0..3]` to `'A', 'B', 'C', 'D'`.
- `ptr` modifies `buffer[2]` to `'Z'`.
- `printf` outputs the first four elements of `buffer` as characters.
