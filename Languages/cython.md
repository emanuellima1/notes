# Notes on Cython

## Table of Contents

- [Notes on Cython](#notes-on-cython)
  - [Table of Contents](#table-of-contents)
  - [Compilation](#compilation)
    - [Using distutils with cythonize](#using-distutils-with-cythonize)
  - [Typing](#typing)
    - [Typing variables](#typing-variables)
    - [C Functions](#c-functions)
    - [C Functions with Automatic Python Wrappers](#c-functions-with-automatic-python-wrappers)
    - [Exception Handling](#exception-handling)
    - [C structs, unions, enums an typedefs](#c-structs-unions-enums-an-typedefs)
    - [Efficient Loops](#efficient-loops)
  - [Extension Types](#extension-types)
  - [Code Organization](#code-organization)
  - [Wrapping C++](#wrapping-c)
  - [Profiling](#profiling)
  - [Typed Memoryviews](#typed-memoryviews)
  - [Parallelism](#parallelism)
  - [References](#references)

## Compilation

### Using distutils with cythonize

Consider a `fib.pyx` Cython source code. Our goal is to use distutils to create a compiled extension module (`fib.so` on Mac OS X or Linux and `fib.pyd` on Windows). For that, we use a `setup.py` file like so:

```python
from distutils.core import setup
from Cython.Build import cythonize

setup(name='Fibonacci App',
      ext_modules=cythonize('fib.pyx',
                            nthreads=4,
                            force=False,
                            annotate=True,
                            compiler_directives={'binding': True},
                            language_level="3"))
```

The arguments of the function `cythonize` can be seen [here](https://cython.readthedocs.io/en/stable/src/userguide/source_files_and_compilation.html#cythonize-arguments). The most important ones are explained below:

- The first argument is the name of the Cython files. It can also be a glob pattern such as `src/*.pyx`;
- `nthreads`: The number of concurrent builds for parallel compilation (requires the multiprocessing module);
- `force`: Forces the recompilation of the Cython modules, even if the timestamps don’t indicate that a recompilation is necessary. Default is False;
- `annotate`: If True, will produce a HTML file for each of the .pyx or .py files compiled. The HTML file gives an indication of how much Python interaction there is in each of the source code lines, compared to plain C code. It also allows you to see the C/C++ code generated for each line of Cython code. Default is False;
- `compiler_directives`: Allows to set compiler directives. More information [here](https://cython.readthedocs.io/en/stable/src/userguide/source_files_and_compilation.html#compiler-directives);
- `language_level`: The level of the Python language. `3` is for Python 3.

These two function calls succinctly demonstrate the two stages in the pipeline: cythonize calls the cython compiler on the .pyx source file or files, and setup compiles the generated C or C++ code into a Python extension module. A C compiler, such as `gcc`, `clang` or `MSVC` is necessary at compile time.

To build on Linux and MacOS, run:

```bash
python3 setup.py build_ext --inplace
```

The build_ext argument is a command instructing distutils to build the Extension object or objects that the cythonize call created. The optional --inplace flag instructs distutils to place each extension module next to its respective Cython .pyx source file.

On Windows:

```powershell
python setup.py build_ext --inplace --compiler=msvc
```

If you use setuptools instead of distutils, the default action when running `python3 setup.py install` is to create a zipped egg file which will not work with cimport for pxd files when you try to use them from a dependent package. To prevent this, include zip_safe=False in the arguments to setup().

One can also set compiler options in the `setup.py`, before calling `cythonize()`, like so:

```python
from distutils.core import setup
from Cython.Build import cythonize

from Cython.Compiler import Options

Options.embed = True

setup(name='Fibonacci App',
      ext_modules=cythonize('fib.pyx',
                            nthreads=4,
                            force=False,
                            annotate=True,
                            compiler_directives={'binding': True},
                            language_level="3"))
```

The `embed` option embeds the Python interpreter, in order to make a standalone executable. This will provide a C function which initialises the interpreter and executes the body of this module. More options [here](https://cython.readthedocs.io/en/stable/src/userguide/source_files_and_compilation.html#compiler-options).

## Typing

### Typing variables

Untyped dynamic variables are declared and behave exactly like Python variables:

```python
a = 42
```

Statically typed variables are declared like so:

```cython
cdef int a = 42
cdef size_t len
cdef double *p
cdef int arr[10]
```

and behave like C variables.

It is possible to mix both kinds of variables if there is a trivial correspondence between the types, like C and Python ints:

```cython
# C variables
cdef int a, b, c

# Calculations using a, b, and c...

# Inside a Python tuple
tuple_of_ints = (a, b, c)
```

In Python 3, all `int` objects have unlimited precision. When converting integral types from Python to C, Cython generates code that checks for overflow. If the C type cannot represent the Python integer, a runtime OverflowError is raised.  
A Python `float` is stored as a C `double`. Converting a Python `float` to a C `float` may truncate to 0.0 or positive or negative infinity, according to IEEE 754 conversion rules.  
The Python complex type is stored as a C struct of two doubles. Cython has float complex and double complex C-level types, which correspond to the Python complex type.  

We can also use `cdef` to statically declare variables with a Python type. We can do this for the built-in types like `list`, `tuple`, and `dict` and extension types like NumPy arrays:

```cython
cdef list particles, modified_particles
cdef dict names_from_particles
cdef str pname
cdef set unique_particles
```

**The more static type information we provide, the better Cython can optimize the result.**

### C Functions

When used to define a function, the cdef keyword creates a function with C-calling semantics. A cdef function’s arguments and return type are typically statically typed, and they can work with C pointer objects, structs, and other C types that cannot be automatically coerced to Python types. It is helpful to think of a cdef function as a C function that is defined with Cython’s Python-like syntax.  

```cython
cdef long factorial(long n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)
```

A function declared with `cdef` can be called by any other function (`def` or `cdef`) inside the same Cython source file. However, Cython does not allow a `cdef` function to be called from external Python code. Because of this restriction, `cdef` functions are typically used as fast auxiliary functions to help `def` functions do their job.  
If we want to use `factorial` from Python code outside of this extension module, we need a minimal def function that calls `factorial` internally:

```python
def wrap_factorial(n):
    return factorial(n)
```

One limitation of this is that `wrap_factorial` and its underlying `factorial` are restricted to C integral types only, and do not have the benefit of Python’s unlimited-precision integers. This means that `wrap_factorial` gives erroneous results for arguments larger than some small value, depending on how large an `unsigned long` is on your system. We always have to be aware of the limitations of the C types.

### C Functions with Automatic Python Wrappers

A `cpdef` function combines features from `cdef` and `def` functions: we get a C-only version of the function and a Python wrapper for it, both with the same name. When we call the function from Cython, we call the C-only version; when we call the function from Python, the wrapper is called.

```cython
cpdef long factorial(long n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)
```

A `cpdef` function has one limitation, due to the fact that it does double duty as both a Python and a C function: its arguments and return types have to be compatible with both Python and C types.  

Both `cdef` and `cpdef` can be given an `inline` hint that the C compiler can use or ignore, depending on the situation:

```cython
cpdef inline long factorial(long n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)
```

The `inline` modifier, when judiciously used, can yield performance improvements, especially for small inlined functions called in deeply nested loops, for example.

### Exception Handling

A `def` function always returns some sort of PyObject pointer at the C level. This invariant allows Cython to correctly propagate exceptions from def functions without issue. Cython’s other two function types (`cdef` and `cpdef`) may return a non-Python type, which makes some other exception-indicating mechanism necessary. Example:

```cython
cpdef int divide_ints(int i, int j):
    return i / j
```

To correctly propagate the exception that occurs when j is 0, Cython provides an `except` clause:

```cython
cpdef int divide_ints(int i, int j) except? -1:
    return i / j
```

The `except? -1` clause allows the return value -1 to act as a possible sentinel that an exception has occurred. If `divide_ints` ever returns -1, Cython checks if the global exception state has been set, and if so, starts unwinding the stack.  
In this example we use a question mark in the `except` clause because -1 might be a valid result from `divide_ints`, in which case no exception state will be set. If there is a return value that always indicates an error has occurred without ambiguity, then the question mark can be omitted.  

### C structs, unions, enums an typedefs

The following C constructs:

```c
struct mycpx {
    int a;
    float b;
};

union uu {
    int a;
    short b, c;
};

enum COLORS {ORANGE, GREEN, PURPLE};
```

Can be declared on Cython like this:

```cython
cdef struct mycpx:
    float real
    float imag

cdef union uu:
    int a
    short b, c

cdef enum COLORS:
    ORANGE, GREEN, PURPLE
```

We can combine `struct` and `union` declarations with `ctypedef`, which creates a new type alias for the `struct` or `union`:

```cython
ctypedef struct mycpx:
    float real
    float imag

ctypedef union uu:
    int a
    short b, c
```

To declare and initialize:

```cython
cdef mycpx a = mycpx(3.1415, -1.0)

# Or
cdef mycpx b = mycpx(real=2.718, imag=1.618034)

# Or
cdef mycpx zz
zz.real = 3.1415
zz.imag = -1.0

# Or, structs can be assigned from a Python dictionary (with CPython overhead):
cdef mycpx zz = {'real': 3.1415, 'imag': -1.0}
```

### Efficient Loops

Considering this Python for loop over a range:

```python
n = 100
# ...
for i in range(n):
    # ...
```

Its cythonized version that would produce the best performing C code is:

```cython
cdef unsigned int i, n = 100
for i in range(n):
    # ...
```

## Extension Types

A Python class such as:

```python
class Particle():
    def __init__(self, m, p, v):
        self.mass = m
        self.position = p
        self.velocity = v
    def get_momentum(self):
        return self.mass * self.velocity
```

Would be cythonized as:

```cython
cdef class Particle():
    cdef double mass, position, velocity

    def __init__(self, m, p, v):
        self.mass = m
        self.position = p
        self.velocity = v
    def get_momentum(self):
        return self.mass * self.velocity
```

To make an attribute readonly (for a Python caller): 

```cython
cdef class Particle():
    cdef readonly double mass
    cdef double position, velocity
# ...
```

`mass` will be readable and not writable by the Python caller, but `position` and `velocity` will be completely private.

```cython
cdef class Particle():
    cdef readonly double mass
    cdef public double position
    cdef double velocity
# ...
```

Here, `position` will be both readable and writable by the Python caller.  
If C-level allocations and deallocations must occur, then use the `__cinit__` and `__dealloc__` methods:

```cython
cdef class Matrix:
    cdef:
        unsigned int nrows, ncols
        double *_matrix

    def __cinit__(self, nr, nc):
        self.nrows = nr
        self.ncols = nc
        self._matrix = <double*>malloc(nr * nc * sizeof(double))
        if self._matrix == NULL:
            raise MemoryError()

    def __dealloc__(self):
        if self._matrix != NULL:
        free(self._matrix)
```

You can cast a Python object to a static object:

```cython
# p is a Python object that may be a Particle

cdef Particle static_p = p

# Or, with the possibility of segfault if p is not a particle:

<Particle>p 

# Or, safelly, but with overhead:

<Particle?>p
```

`None` can be passed as argument for functions that receive static type. This will lead to segfaults. To protect against it:

```cython
def dispatch(Particle p not None):
    print p.get_momentum()
    print p.velocity
```

## Code Organization

## Wrapping C++

## Profiling

## Typed Memoryviews

## Parallelism

## References

- Kurt W. Smith. Cython. 1st Edition. O’Reilly.
- [Official Cython’s Documentation](https://cython.readthedocs.io/en/stable/index.html)
