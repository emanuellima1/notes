# Notes on C

## Table of Contents

- [Notes on C](#notes-on-c)
  - [Table of Contents](#table-of-contents)
  - [Basics](#basics)

## Basics

The minimal "hello world" C program:

```c
#include <stdio.h>

int main()
{
    printf("Hello World!\n");
    return 0;
}
```

Initializing an array:

```c
double A[5] = {
    [0] = 9.0,
    [1] = 2.9,
    [4] = 3.E+25,
    [3] = .00007,
};
```

Any position that is not listed in the initializer is set to 0. A[2] == 0.0  
