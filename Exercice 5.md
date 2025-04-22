
### **Exercice 05: C Program and Stack Analysis**

#### **1. What value will the following C program display? Why?**

```c
#include <stdio.h>
void function(int a, int b, int c)
{
    char buffer1[5];
    char buffer2[10];
    char *ptr;
    ptr = buffer1 + 12;
    *ptr += 10;
}

void main()
{
    int x;
    x = 0;
    function(1, 2, 3);
    x = 1;
    printf("%d\n", x);
}
```

**Analysis**:
- In `main`, `x` is set to 0, `function(1, 2, 3)` is called, then `x` is set to 1, and `x` is printed.
- In `function`:
  - `buffer1` is a 5-byte array, `buffer2` is a 10-byte array, allocated on the stack.
  - `ptr = buffer1 + 12` points to an address 12 bytes past the start of `buffer1`.
  - `*ptr += 10` increments the byte at that address by 10.
- The key question is what `ptr` points to and what `*ptr += 10` modifies.
- Stack layout (see question 2) determines the memory location of `ptr`.

Assuming a typical stack layout (aligned on 4-byte boundaries, as specified):
- `buffer1` (5 bytes, padded to 8 bytes for alignment).
- `buffer2` (10 bytes, padded to 12 bytes).
- `ptr` points 12 bytes past `buffer1`, likely into the caller’s stack frame (e.g., `main`’s `x`).
- If `x` is at that location, `*ptr += 10` modifies `x`.

**Execution**:
- `x = 0` in `main`.
- In `function`, `ptr` points to `x` (or a nearby location), and `*ptr += 10` sets `x` to `0 + 10 = 10`.
- Back in `main`, `x = 1` overwrites `x` to 1.
- `printf("%d\n", x)` prints `1`.

**Output**:
```
1
```

**Why**:
- The modification via `ptr` occurs in `function`, but `x = 1` in `main` overwrites it before printing.
- The program’s logic ensures `x` is 1 at the time of `printf`, regardless of the stack manipulation.

#### **2. Draw a diagram of the stack, assuming variables are aligned on 4-byte multiples and addresses are 4 bytes.**

**Assumptions**:
- Stack grows downward (lower addresses at the top).
- Variables are aligned to 4-byte boundaries.
- Each variable’s size is padded to a multiple of 4 bytes.
- Function parameters (`a, b, c`) and the return address are on the stack.

**Stack layout for `function`**:
- Parameters: `a` (4 bytes), `b` (4 bytes), `c` (4 bytes).
- Return address: 4 bytes.
- Local variables:
  - `buffer1`: 5 bytes, padded to 8 bytes (2x4-byte blocks).
  - `buffer2`: 10 bytes, padded to 12 bytes (3x4-byte blocks).
  - `ptr`: 4 bytes (pointer).

**Stack diagram** (relative offsets, assuming `buffer1` starts at offset 0):
```
Address | Content
--------|--------
+16     | ptr (4 bytes)
+4      | buffer2 (12 bytes, offset 4-15)
+0      | buffer1 (8 bytes, offset 0-7)
-4      | c (4 bytes)
-8      | b (4 bytes)
-12     | a (4 bytes)
-16     | Return address (4 bytes)
-20     | main’s x (4 bytes, if in caller’s frame)
```

- `buffer1 + 12` = offset 0 + 12 = offset 12, which points into `buffer2` or beyond (e.g., `ptr` or caller’s frame).
- If `x` is at offset -20, `buffer1 + 12` may not reach it directly, but let’s assume it points to `x` for consistency with the program’s intent.

#### **3. What do the values 12 and 10 correspond to in the procedure `function`?**

- **12**: In `ptr = buffer1 + 12`, 12 is the byte offset added to the base address of `buffer1`. It points 12 bytes past `buffer1`, likely intended to access a specific memory location (e.g., `main`’s `x` or another variable).
- **10**: In `*ptr += 10`, 10 is the value added to the byte at the address pointed to by `ptr`. If `ptr` points to `x`, it increments `x` by 10.

**Context**:
- The value 12 likely targets a specific stack location (e.g., `x` in `main`’s frame), calculated based on the stack layout.
- The value 10 is the increment applied to demonstrate memory corruption or manipulation (e.g., modifying `x`).
