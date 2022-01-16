# Notes on C

## Table of Contents

- [Notes on C](#notes-on-c)
  - [Table of Contents](#table-of-contents)
  - [Basics](#basics)
    - [Enums](#enums)
    - [Pointers](#pointers)
    - [Arrays](#arrays)
    - [Structs](#structs)
    - [Unions](#unions)
  - [Qualifiers](#qualifiers)
  - [Preprocessor](#preprocessor)
  - [Some Best Practices](#some-best-practices)
  - [Some Headers](#some-headers)
  - [Bibliography](#bibliography)

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

**Best Practice**: Never modify more than one object in a statement.

**Best Practice**: Never declare various pointers or arrays on the same line to avoid errors.

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

Every type in C is either an object type or a function type.

C is call-by-value. When you provide an argument to a function, the value of that argument is copied into a distinct variable for use within the function.

Scopes can be nested, with inner and outer scopes. For example, you can have a block scope inside another block scope, and every block scope is defined within a file scope. The inner scope has access to the outer scope, but not vice versa. If you declare the same identifier in both the inner scope and an outer scope, the identifier declared in the outer scope is hidden by the identifier within the inner scope, which takes precedence. In this case, naming the identifier will refer to the object in the inner scope; the object from the outer scope is hidden and cannot be referenced by its name.

Scope and lifetime are different. Scope applies to identifiers, whereas lifetime applies to objects. The scope of an identifier is the code region where the object denoted by the identifier can be accessed by its name. The lifetime of an object is the time period for which the object exists.

**Automatic** lifetimes are declared within a block or as a function parameter. The lifetime of these objects begins when the block in which they’re declared begins execution, and ends when execution of the block ends. If the block is entered recursively, a new object is created each time, each with its own storage.

Objects declared in file scope have **static** storage duration. The lifetime of these objects is the entire execution of the program, and their stored value is initialized prior to program startup. One can use `static` to declare a variable within a block scope to have a static lifetime. These objects persist after the function has exited.

**Best Practice**: Never declare functions with an empty parameter list in C. Always use `void` in the parameter like so:

```c
int my_function(void);
```

### Enums

Allows you to define a type that assigns names (enumerators) to integer values in cases with an enumerable set of constant val-
ues.

```c
enum day { sun, mon, tue, wed, thu, fri, sat };
enum cardinal_points { north = 0, east = 90, south = 180, west = 270 };
enum months { jan = 1, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec };
```

### Pointers

```c
int* ip;
char* cp;
void* vp;

int i = 17;
int* ip = &i;
```

The `&` operator takes the address of an object or function.
The `*` operator converts a pointer to a type into a value of that type. It denotes indirection and operates only on pointers.

### Arrays

A contiguously allocated sequence of objects that all have the same element type.

```c
int ia[11];
float* afp[17];

int matrix[3][5];
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

### Structs

A struct contains sequentially allocated member objects. One can reference members of a struct by using the struct member operator (`.`). If you have a pointer to a struct, you can reference its members with the struct pointer operator (`->`).

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

### Unions

Union types are similar to structures, except that the memory used by the member objects overlaps. Unions can contain an object of one type at one time, and an object of a different type at a different time, but never both objects at the same time, and are primarily used to save memory.

## Qualifiers

Types can be qualified by using one or more of the following qualifiers: const, volatile, and restrict.

- `const`: not modifiable. Will be placed in read-only memory by the compiler, and any attempt to write to them will result in a runtime error.
- `volatile`: values stored in these objects may change without the knowledge of the compiler. For example, every time the value from a real-time clock is read, it may change, even if the value has not been written to by the C program. Using a volatile-qualified type lets the compiler know that the value may change, and ensures that every access to the real-time clock
occurs (­otherwise, an access to the real-time clock may be optimized away or replaced by a previously read and cached value).
- `restrict`: Used to promote optimization. Objects ­indirectly accessed through a pointer frequently cannot be fully optimized because of potential aliasing, which occurs when more than one pointer refers to the same object. Aliasing can inhibit optimizations, because the compiler can’t tell if portions of an object can change values when another apparently unrelated object is modified, for example.

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

## Bibliography

- SEACORD, Robert. Effective C: an introduction to professional C programming. No Starch Press. 2020.
