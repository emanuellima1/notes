# Notes on C

## Table of Contents

- [Notes on C](#notes-on-c)
  - [Table of Contents](#table-of-contents)
  - [Basics](#basics)
    - [Initialization of structs](#initialization-of-structs)
  - [Preprocessor](#preprocessor)
  - [Some Best Practices](#some-best-practices)
  - [Some Headers](#some-headers)

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

// A[2] is initialized as 0.0 
```

Any position that is not listed in the initializer is set to 0. A[2] == 0.0  

Don’t compare to 0, false, or true. Also, all scalars have a truth value. An integer or floating point 0 will always evaluate to false.  

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

size_t is any integer on the interval [0, SIZE_MAX], as defined on **stdint.h**.  

Best Practice: Never modify more than one object in a statement.  

Ternary operator:  

```c
// Pattern:
// (boolean_expression) ? if_true : if_false;

size_t size_min(size_t a , size_t b) {
    return ( a < b ) ? a : b;
}
```

Attention: in an expression such as `f(a) + g(b)`, there is no pre-established order specifying
whether `f(a)` or `g(b)` is to be computed first. If either the function `f` or `g` works with side effects
(for instance, if `f` modifies `b` behind the scenes), the outcome of the expression will depend on the
chosen order. The same holds for function arguments:  

```c
printf("%g and %g\n", f(a), f(b));
```

**Best Practice**: Functions that are called inside expressions should not have side effects.  

### Initialization of structs

```c
typedef struct vec2 { float x, y; } vec2;

vec2 v0 = {1.0f, 2.0f};
vec2 v1 = {.x = 1.0f, .y = 2.0f};
vec2 v2 = {.y = 2.0f};    // missing struct members are set to zero

// Inside functions, runtime-variable values can be used for initialization:
float get_x(void) {
    return 1.0f;
}

void bla(void) {
    vec2 v0 = { .x = get_x(), .y = 2.0f };
}

// But this doesn't work:
    vec2 v0;
    // this doesn't work
    v0 = {1.0f, 2.0f};
    // instead a type hint is needed:
    v0 = (vec2) {1.0f, 2.0f};

```

## Preprocessor

Defines a macro as 0 and an empty macro:

```c
#define  __MACRO__ 0

#define  __MACRO2__
```

Checks if a macro is defined. If it is, throws an error (the error stops the compilation):

```c
#ifdef  __MACRO__
#error "Error!"
#endif
```

Checks if a macro is not defined. If it isn't, defines it:

```c
#ifndef __MACRO__
#define __MACRO__
#endif
```

## Some Best Practices

- Enable all warnings: -Wall and -Wextra on GCC and Clang
- Wrap your structs in a typedef:

```c
typedef struct bla{
    int a, b, c;
} bla;

// Attention: the POSIX standard reserves the ‘_t’ postfix for its own type names to prevent collisions with user types.
```

- Use void to indicate that a function does not receive arguments:

```c
// GOOD
void my_func(void) {
    ...
}

// BAD
void my_func() {
    ...
}
```

- Don’t be afraid to pass and return structs by value:

```c
typedef struct float2{ float x, y; } float2;

float2 addf2(float2 v0, float2 v1) {
    return (float2) { v0.x + v1.x, v0.y + v1.y };
}

...
float2 v0 = {1.0f, 2.0f};
float2 v1 = {3.0f, 4.0f};
float2 v3 = addf2(v0, v1);
...

// You can also move the initialization of the two inputs right into the function call:
float2 v3 = addf2((float2){ 1.0f, 2.0f }, (float2){ 3.0f, 4.0f });
```

## Some Headers

- **stdint.h**: defines integer types with certain widths.
- **tgmath.h**: defines type generic math functions, for both real and complex numbers.
