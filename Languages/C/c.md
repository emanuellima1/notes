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

Don’t compare to 0, false, or true. Also, all scalars have a truth value.

```c
// GOOD
bool b = true;

if ( b ) {
  // do something
}

// BAD
bool b = true;

if (( b != false ) == true ) {
  // do something
}
```

The scalar types:  

| Name          | Where       | printf              |
| ------------- | ----------- | ------------------- |
| size_t        | <stddef.h>  | "%zu" "%zx          |
| double        | Built in    | "%e" "%f" "%g" "%a" |
| signed        | Built in    | "%d"                |
| unsigned      | Built in    | "%u" "%x"           |
| bool          | <stdbool.h> | "%d"                |
| ptrdiff_t     | <stddef.h>  | "%td"               |
| char const*   | Built in    | "%s"                |
| char          | Built in    | "%c"                |
| void*         | Built in    | "%p"                |
| unsigned char | Built in    | "%hhu" "%02hhx"     |

pag25
