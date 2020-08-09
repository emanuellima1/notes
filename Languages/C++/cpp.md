# Notes on C++

## Table of Contents

- [Notes on C++](#notes-on-c)
  - [Table of Contents](#table-of-contents)
  - [Basics](#basics)
  - [User-Defined Types](#user-defined-types)
  - [Modularity](#modularity)
  - [Classes](#classes)
  - [Operations](#operations)
  - [Templates](#templates)
  - [Generic Programming](#generic-programming)
  - [Standard Library](#standard-library)
  - [Strings and Regex](#strings-and-regex)
  - [I/O](#io)
  - [Containers](#containers)
  - [Algorithms](#algorithms)
  - [Utilities](#utilities)
  - [Numerics](#numerics)
  - [Concurrency](#concurrency)
  - [References](#references)

## Basics

The minimal "hello world" C++ program:

```cpp
#include <iostream>
int main()
{
    std::cout<<"Hello, World!\n";
    return 0;
}
```

The #include directive imports a library to be used. The main function is the entry point of the program and returns an integer upon successful completion.  
Blocks of code that perform some well defined action should be encapsulated on a function. That function would then be called on main(). A function must be declared before being used. The declaration follows this syntax:

```cpp
Node* next_node();     // Receives no argument, returns pointer to Node
void call_number(int); // Receives an int, returns nothing
double exp(double);    // Receives a double, returns a double.
```

There can be more than one argument. The types of the arguments and of the return are checked at compile time.  
The code that makes up a software must be comprehensible because this is the first step to maintainability. To make code more comprehensible, we must divide its tasks into functions. That way, we will have a basic vocabulary for data (types, both built-in and user defined) and for actions on that data (functions). Encapsulating a specific action into a function, forces us to not repeat ourselves (DRY) and to document that action's dependencies.  
If a function with the same name is declared with different arguments, the compiler will accept all of them and use the function corresponding to the type used. This allows us to change the behavior of an action based on the type of the data we are dealing with. Attention to the fact that the declarations cannot be ambiguous. If so, the compiler will throw an error. Another point is that if a set of functions have the same name, they should have the same semantics, i.e., they should perform the same action. Example:

```cpp
// This is called function overloading
void sum(int, int);
void sum(double, double);

void use()
{
  sum(2, 2);
  sum(2.7, 3.14);
}
```

Every object in the language has a type. An object is a portion of memory that holds a value of the given type. A value is a set of bits that are interpreted according to the type. Some built-in types are bool, char, int and double. Int variables' representation default to decimal (42) but can also be declared as binary (0b1010), hexadecimal (0x04AF) or octal (0432).  
The usual arithmetic, comparison, logical and modification operations are present:

|Arithmetic|Comparison|  Logical   |Modification|
|----------|----------|------------|------------|
|x + y     |  x == y  |   x && y   |   x += y   |
|x - y     |  x != y  |   x \|\| y   |   x -= y   |
|x * y     |  x < y   |     !x     |   ++x      |
|x / y     |  x > y   |            |   --x      |
|x % y     |  x <= y  |            |   x *= y   |
|          |  x >= y  |            |   x /= y   |
|          |          |            |   x %= y   |

Attention to the fact that C++ makes implicit conversion on basic types:

```cpp
void my_function()
{
  double d = 3.14;
  int i = 5;
  d = d + i; // d is 8.14 here  
  i = d * i; // i is 15 here.
  // The resulting multiplication is 15.7,
  // but the value is truncated to fit an int
}
```

There are two forms of variable initialization, "=" or "{}":

```cpp
double d = 3.14;
double d2 {2.7};

// Vector of ints
vector<int> v {1, 2, 3};
```

The "=" is a tradition from C. "{}" is prefered because it avoids implicit conversions:

```cpp
int a = 4.8; // a is 4 here
int b {4.8}; // Error
```

If the data is coming from the user, for example, implicit conversions are a recipe for disaster, if not checked.  
The type of a variable can be deduced by the compiler with the "auto" keyword. My opinion is that auto should only be used when the types are very long (as they usually are with templates). Explicit (short) types help with code readability.

```cpp
auto a = 4.8;
auto b {4.8}; // Both a and b are double
```

-> 1.5


## User-Defined Types

## Modularity

## Classes

## Operations

## Templates

## Generic Programming

## Standard Library

## Strings and Regex

## I/O

## Containers

## Algorithms

## Utilities

## Numerics

## Concurrency

## References

- Bjarne Stroustrup. A Tour of C++. Pearson Education. 2018.