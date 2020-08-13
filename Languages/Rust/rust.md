# Notes on Rust

## Table of Contents

- [Notes on Rust](#notes-on-rust)
  - [Table of Contents](#table-of-contents)
  - [Basics](#basics)
  - [Importing and Namespaces](#importing-and-namespaces)
  - [Panic, Options and Result](#panic-options-and-result)
  - [Lifetime](#lifetime)
  - [Memory](#memory)
    - [Stack and Heap](#stack-and-heap)
    - [Ownership](#ownership)
  - [References](#references)

## Basics

Rust's philosophy:

- Strictly enforcing safe borrowing of data
- Functions, methods and closures to operate on data
- Tuples, structs and enums to aggregate data
- Pattern matching to select and destructure data
- Traits to define behavior on data

```rust
// Hello world
fn main() {
    println!(“Hello!”);
}
```

Functions to write strings:

- `format!`: write formatted text to String
- `print!`: same as format! but the text is printed to the console (io::stdout).
- `println!`: same as print! but a newline is appended.
- `eprint!`: same as format! but the text is printed to the standard error (io::stderr).
- `eprintln!`: same as eprint!but a newline is appended.

Ways to print to stdout:

```rust
fn main() {
    println!(“Today is {}”, 20);

    // By positional argument
    println!("{0}, this is {1}. {1}, this is {0}", "Alice", "Bob");

    // By naming the arguments
    println!("{subject} {verb} {object}", object="the lazy dog",subject="the quick brown fox", verb="jumps over");

    // By right-aligning a text with a specified width
    println!("{number:>width$}", number=1, width=6);

    // By padding numbers with extra zeroes
    println!("{number:>0width$}", number=1, width=6);

    // By rounding floats
    let pi = 3.141592;
    println!("Pi is roughly {:.3}", pi);
}
```

Tests:

```rust
fn main() {
    // Checks if x is equal to y. If not, panic.
    assert_eq!(x, y);

    // Checks if x is not equal to y. If not, panic.
    assert_neq!(x, y);

    // Checks if expression is true. If not, panic.
    assert!(expression);
}
```

Commentaries:

```rust
fn main() {
    // Regular commentary.

    /*
        Multi-line
        Commentary
    */

    /// Commentary that will become documentation with cargo doc --open
    /// It accepts markdown
    /// And can contain tests:
    ///
    /// ```
    /// use my_lib::*;
    ///
    /// assert_eq!(3, soma(1, 2));
    /// ```
    pub fn soma(x: i32, y: i32) -> i32
    {
        x + y
    }
}
```

The build system and dependency manager is called cargo:

```bash
# Creates a new project
cargo new [name]

# Compiles
cargo build

# Compiles with optimizations
cargo build --release

# Checks for syntax errors
cargo check

# Compiles and executes
cargo run

# Updates the dependencies. Doesn't bump major version
cargo update

# Creates documentation
cargo doc --open

# Linting
cargo fmt
cargo fix
cargo clippy
```

Declaration of variables:

```rust
fn main() {
    // Immutable variable
    let a = 1;

    // Mutable variable
    let mut b = 2;

    // Constant with type annotation
    const CTE: u32 = 100;

    // It is possible to do shadowing. The final value o x is 6
    let x = 5;
    let x = x + 1;
}
```

For integers, we have i8, i16, i32, i64, i128 and u8, u16, u32, u64, u128 for unsigned. For floats, f32 and f64. i32 and f64 are the default types. The usual operators are present: + - * / %  
The underscore _ means to throw away something:

```rust
// this does nothing because 42 is a constant
let _ = 42;

// this calls `get_thing` but throws away its result
let _ = get_thing();

// Starting with an underscore means the compiler  won't warn about them being unused:
let _x = 42;
```


Tuples:

```rust
fn main() {
    // Doesn't grow in size
    let tup: (i32, f64, u8) = (500, 6.4, 1);

    // Destructuring
    let (x, y, z) = tup;
    println!("The value of y is: {}", y);

    // Destructuring only a part of it
    let (_, _, one) = tup;

    // By index
    let five_hundred = tup.0;
}
```

Arrays:

```rust
fn main() {
    // Doesn't grow in size. All elements of the same type.
    let a = [1, 2, 3, 4, 5];
    let first = a[0];

    // Creates an array of 5 elements, each of which is 3
     let a = [3; 5];
}
```

Structs:

```rust
// Declaration
struct Vec2 {
    x: f64,
    y: f64,
}

// Initialization
// The order does not matter, only the names do
let v1 = Vec2 { x: 1.0, y: 3.0 };
let v2 = Vec2 { y: 2.0, x: 4.0 };

// Initializing the rest of the fields from another struct
let v3 = Vec2 { x: 14.0, ..v2 };
let v4 = Vec2 { ..v3 };

// Destructuring
// `x` is now 1.0, `y` is now `3.0`
let Vec2 { x, y } = v1;

// This throws away `v.y`
let Vec2 { x, .. } = v;
```

Functions:

```rust
fn my_function(arg1: type1, arg2: type2) -> return_type
{
​   //body
}
```

You can declare methods on your own types:

```rust
struct Number {
    odd: bool,
    value: i32,
}

impl Number {
    fn is_strictly_positive(self) -> bool {
        self.value > 0
    }
}
```

Traits are something multiple types can have in common. You can implement:

- One of your traits on anyone's type
- Anyone's trait on one of your types
- But not a foreign trait on a foreign type

```rust
struct Number {
    odd: bool,
    value: i32,
}

trait Signed {
    fn is_strictly_negative(self) -> bool;
}

// Our trait on our type
impl Signed for Number {
    fn is_strictly_negative(self) -> bool {
        self.value < 0
    }
}

// Our trait on a foreign type
impl Signed for i32 {
    fn is_strictly_negative(self) -> bool {
        self < 0
    }
}

// A foreign trait on our type:
// The `Neg` trait is used to overload `-`, the unary minus operator.
// An impl block is always for a type, so, inside that block, Self means that type.
impl std::ops::Neg for Number {
    type Output = Self;

    fn neg(self) -> Self {
        Self {
            value: -self.value,
            odd: self.odd,
        }
    }
}
```

Some traits are markers - they don't say that a type implements some methods, they say that certain things can be done with a type:

```rust
fn main() {
    // i32 implements trait Copy (in short, i32 is Copy):
    let a: i32 = 15;
    let b = a; // `a` is copied
    let c = a; // `a` is copied again
    print_i32(a); // `a` is copied again

    // The Number struct is not Copy, so this doesn't work:
    let n = Number { odd: true, value: 51 };
    let m = n; // `n` is moved into `m`
    let o = n; // error: use of moved value: `n`

    // It works if print_number takes an immutable reference instead:
    print_number(&n); // `n` is borrowed for the time of the call
    print_number(&n); // `n` is borrowed again

    // It also works if a function takes a mutable reference - but only if our variable binding is also mut:
    let mut m = Number { odd: true, value: 51 };
    print_number(&m);
    invert(&mut m); // `m is borrowed mutably - everything is explicit
    print_number(&m);
}

fn print_i32(x: i32) {
    println!("x = {}", x);
}

fn print_number(n: &Number) {
    println!("{} number {}", if n.odd { "odd" } else { "even" }, n.value);
}

fn invert(n: &mut Number) {
    n.value = -n.value;
}
```

Trait methods can also take self by reference or mutable reference:

```rust
impl std::clone::Clone for Number {
    fn clone(&self) -> Self {
        Self { ..*self }
    }
}

// Marker traits like Copy have no methods:
// `Copy` requires that `Clone` is implemented too.
// Number values will no longer be moved, but copied.
impl std::marker::Copy for Number {}

```

When invoking trait methods, the receiver is borrowed implicitly:

```rust
fn main() {
    let n = Number { odd: true, value: 51 };
    let mut m = n.clone();
    m.value += 100;

    print_number(&n);
    print_number(&m);
}
```

Some traits are so common, they can be implemented automatically by using the derive attribute:

```rust
#[derive(Clone, Copy)]
struct Number {
    odd: bool,
    value: i32,
}
```

Functions can be generic:

```rust
fn foobar<T>(arg: T) {
    // do something with `arg`
}

// Multiple type parameters:
fn foobar<L, R>(left: L, right: R) {
    // do something with `left` and `right`
}
```

Type parameters can have constraints:

```rust
fn print<T: Display>(value: T) {
    println!("value = {}", value);
}

fn print<T: Debug>(value: T) {
    println!("value = {:?}", value);
}

// Longer syntax:
fn print<T>(value: T)
where
    T: Display,
{
    println!("value = {}", value);
}

// If you want multiple constraints:
use std::fmt::Debug;

fn compare<T>(left: T, right: T)
where
    T: Debug + PartialEq,
{
    println!("{:?} {} {:?}", left, if left == right { "==" } else { "!=" }, right);
}

fn main() {
    compare("tea", "coffee"); // prints: "tea" != "coffee"
}
```

Generic functions can be navigated using ::

```rust
fn main() {
    use std::any::type_name;

    // Turbofish syntax
    println!("{}", type_name::<i32>()); // prints "i32"
    println!("{}", type_name::<(f64, char)>()); // prints "(f64, char)"
}
```

Structs can be generic:

```rust
struct Pair<T> {
    a: T,
    b: T,
}

fn print_type_name<T>(_val: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let p1 = Pair { a: 3, b: 9 };
    let p2 = Pair { a: true, b: false };
    print_type_name(&p1); // prints "Pair<i32>"
    print_type_name(&p2); // prints "Pair<bool>"

    // Vec is generic
    let mut v1 = Vec::new();
    v1.push(1);
    let mut v2 = Vec::new();
    v2.push(false);
}
```

Enums are types that can only have some specific values

```rust
fn main() {
    enum WebEvent {
    // An `enum` may either be `unit-like`,
    PageLoad,
    PageUnload,

    // like tuple structs,
    KeyPress(char),
    Paste(String),

    // or c-like structures.
    Click { x: i64, y: i64 },
    }
}
```

Vectors:

```rust
fn main() {
    // Vec is generic
    let mut v1 = Vec::new();
    v1.push(1);
    let mut v2 = Vec::new();
    v2.push(false);

    // Literals:
    let v1 = vec![1, 2, 3];
    let v2 = vec![true, false, true];
}
```

Conditional:

```rust
if number % 4 == 0 {
    println!("number is divisible by 4");
} else if number % 3 == 0 {
    println!("number is divisible by 3");
} else if number % 2 == 0 {
    println!("number is divisible by 2");
} else {
    println!("number is not divisible by 4, 3, or 2");
}

let number = if condition {
    5
} else {
    6
};
```

Match:

```rust
// Similar to switch in C, but more powerful
match feeling_lucky {
    true => 6,
    false => 4,
}

/*******************************************************/
struct Number {
    odd: bool,
    value: i32,
}

fn main() {
    let one = Number { odd: true, value: 1 };
    let two = Number { odd: false, value: 2 };
    print_number(one);
    print_number(two);
}

fn print_number(n: Number) {
    match n {
        Number { odd: true, value } => println!("Odd number: {}", value),
        Number { odd: false, value } => println!("Even number: {}", value),
    }
}

/*******************************************************/
// A match has to be exhaustive: at least one arm needs to match.
fn print_number(n: Number) {
    match n {
        Number { value: 1, .. } => println!("One"),
        Number { value: 2, .. } => println!("Two"),
        Number { value, .. } => println!("{}", value),
        // If the last arm didn't exist, we would get a compile-time error
    }
}

/*******************************************************/
// _ can be used as a "catch-all" pattern:
fn print_number(n: Number) {
    match n.value {
        1 => println!("One"),
        2 => println!("Two"),
        _ => println!("{}", n.value),
    }
}
```

Loop:

```rust
// Infinite loop
loop {
    // body
}

// result is 20
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;
    }
};

while number != 0 {
    println!("{}!", number);
    number -= 1;
}

let a = [10, 20, 30, 40, 50];
for element in a.iter() {
    println!("the value is: {}", element);
}

for number in (1..4).rev() {
    println!("{}!", number);
}
```

A pair of brackets declares a block, which has its own scope:

```rust
// This prints "in", then "out"
fn main() {
    let x = "out";
    {
        // this is a different `x`
        let x = "in";
        println!(x);
    }
    println!(x);
}
```

Blocks are also expressions, which mean they evaluate to a value.

```rust
// This:
let x = 42;

// Is equivalent to this:
let x = { 42 };
```

Inside a block, there can be multiple statements:

```rust
let x = {
    let y = 1; // First statement
    let z = 2; // Second statement
    y + z // This is the tail - what the whole block will evaluate to
};
```

That's why "omitting the semicolon at the end of a function" is the same as
returning.

## Importing and Namespaces

`use` directives can be used to "bring in scope" names from other namespace:

```rust
// std is a crate (~ a library), cmp is a module (~ a source file), and
// min is a function:

use std::cmp::min;
let least = min(7, 1); // This is 1
```

Within use directives, curly brackets have another meaning: they're globs". If we want to import both min and max , we can do any of these:

```rust
// this works:
use std::cmp::min;
use std::cmp::max;

// this also works:
use std::cmp::{min, max};

// this also works!
use std::{cmp::min, cmp::max};
```

A wildcard ( * ) lets you import every symbol from a namespace:

```rust
// This brings `min` and `max` in scope, and many other things
use std::cmp::*;
```

## Panic, Options and Result

```rust
// Is a macro that violently stops execution with an error message, and the file name / line number of the error
panic!("Error message");

// Option is a type that contains something, or nothing. If .unwrap() is called on it, and it contains nothing, it panics
enum Option<T> {
    None,
    Some(T),
}

let o1: Option<i32> = Some(128);
o1.unwrap(); // this is fine

let o2: Option<i32> = None;
o2.unwrap(); // this panics!

// Result is an enum that can either contain something, or an error. It also panics when unwrapped and containing an error
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

## Lifetime

Variables have lifetimes:

```rust
fn main() {
    // `x` doesn't exist yet
    {
        // `x` starts existing
        let x = 42;

        // `x_ref` starts existing - it borrows `x`
        let x_ref = &x;

        println!("x = {}", x);

        // `x_ref` stops existing
        // `x` stops existing
    }
    // `x` no longer exists
}
```

The lifetime of a reference cannot exceed the lifetime of the variable binding it borrows:

```rust
fn main() {
    let x_ref = {
        let x = 42;
        &x
    };
    println!("x_ref = {}", x_ref);
    // error: `x` does not live long enough
}
```

## Memory

### Stack and Heap

Both the stack and the heap are parts of memory that are available to your code to use at runtime, but they are structured in different ways. The stack is last in, first out. Adding data is called pushing onto the stack, and removing data is called popping off the stack.  
All data stored on the stack must have a known, fixed size. Data with an unknown size at compile time or a size that might change must be stored on the heap instead. The heap is less organized: when you put data on the heap, you request a certain amount of space. The operating system finds an empty spot in the heap that is big enough, marks it as being in use, and returns a pointer, which is the address of that location. This process is
called allocating on the heap and is sometimes abbreviated as just allocating. Pushing values onto the stack is not considered allocating. Because the pointer is a known, fixed size, you can store the pointer on the stack, but when you want the actual data, you must follow the pointer.  
Pushing to the stack is faster than allocating on the heap because the operating system never has to search for a place to store new data; that location is always at the top of the stack. Comparatively, allocating space on the heap requires more work, because the operating system must first find a big enough space to hold the data and then perform bookkeeping to prepare for the next allocation.  
When your code calls a function, the values passed into the function (including, potentially, pointers to data on the heap) and the function’s local variables get pushed onto the stack. When the function is over, those values get popped off the stack.  
The stack is the memory set aside as scratch space for a thread of execution. When a function is called, a block is reserved on the top of the stack for local variables and some bookkeeping data. When that function returns, the block becomes unused and can be used the next time a
function is called. The stack is always reserved in a LIFO (last in first out) order; the most recently reserved block is always the next block to be freed. This makes it really simple to keep track of the stack;
freeing a block from the stack is nothing more than adjusting one pointer.  
The heap is memory set aside for dynamic allocation. Unlike the stack, there's no enforced pattern to the allocation and deallocation of blocks from the heap; you can allocate a block at any time and free it at any time. This makes it much more complex to keep track of which parts of the heap are allocated or free at any given time; there are many custom heap allocators available to tune heap performance for different usage patterns.  
Each thread gets a stack, while there's typically only one heap for the application (although it isn't uncommon to have multiple heaps for different types of allocation).  
The OS allocates the stack for each system-level thread when the thread is created. Typically the OS is called by the language runtime to allocate the heap for the application.  
The stack is attached to a thread, so when the thread exits the stack is reclaimed. The heap is typically allocated at application startup by the runtime, and is reclaimed when the application (technically process) exits.  
The size of the stack is set when a thread is created. The size of the heap is set on application startup, but can grow as space is needed (the allocator requests more memory from the operating system).  
The stack is faster because the access pattern makes it trivial to allocate and deallocate memory from it (a pointer/integer is simply incremented or decremented), while the heap has much more complex bookkeeping involved in an allocation or deallocation. Also, each byte in the stack tends to be reused very frequently which means it tends to be mapped to the processor's cache, making it very fast. Another performance hit for the heap is that the heap, being mostly a global resource, typically has to be multi-threading safe, i.e. each allocation and deallocation needs to be - typically - synchronized with "all" other heap accesses in the program.  

### Ownership

Each value in Rust has a variable that’s called its owner. There can only be one owner at a time. When the owner goes out of scope, the value will be
dropped.  
Copies of heap objects are shallow, meaning that a new variable is just another pointer (on the stack) to the same heap address. The first pointer becomes stale.  

```rust
let s1 = String::from("Hello!");
let s2 = s1;
```

Stack variables are deep copied. They have a fixed size known in compile time.  As a general rule, any group of simple scalar values can be Copy, and nothing that requires allocation or is some form of resource is Copy. Ex.: integers, floats, booleans, characters and tuples of these types.  
Passing a variable to a function will move or copy, just as assignment does:

```rust
fn main() {
    // s comes into scope
    let s = String::from("hello");

    // s's value moves into the function and so is no
    // longer valid here
    takes_ownership(s);
​
    // x comes into scope
    let x = 5;
​
    // x would move into the function, but i32 is Copy,
    // so it’s okay to still use x afterward
    makes_copy(x);
}
// Here, x goes out of scope, then s. But because s's
// value was moved, nothing special happens.

fn takes_ownership(some_string: String) {
​    // some_string comes into scope
    println!("{}", some_string);
}​
// Here, some_string goes out of scope and `drop` is
// called. The backing memory is freed.

fn makes_copy(some_integer: i32) {
    // some_integer comes into scope
    println!("{}", some_integer);
}​
// Here, some_integer goes out of scope. Nothing
// special happens.
```

The ownership of a variable follows the same pattern every time: assigning a value to another variable moves it. When a variable that includes data on the heap goes out of scope, the value will be cleaned up by drop unless
the data has been moved to be owned by another variable.  
At any given time, you can have either one mutable reference or any number of immutable references. References must always be valid.  

## References

- [Rust Book](https://doc.rust-lang.org/book/)
- [A Half Hour to Learn Rust](https://fasterthanli.me/articles/a-half-hour-to-learn-rust)