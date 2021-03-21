# Notes on Julia

## Table of Contents

- [Notes on Julia](#notes-on-julia)
  - [Table of Contents](#table-of-contents)
  - [Basics](#basics)
  - [Data Types and Structures](#data-types-and-structures)
    - [Scalar Types](#scalar-types)
    - [Arrays](#arrays)
    - [Matrixes](#matrixes)
    - [Tuples](#tuples)
    - [Dictionaries](#dictionaries)
    - [Sets](#sets)
    - [Memory and Copy](#memory-and-copy)
    - [Random Numbers](#random-numbers)
  - [Basic Syntax](#basic-syntax)
  - [Custom Types](#custom-types)
  - [I/O](#io)
  - [Metaprogramming](#metaprogramming)
  - [Exceptions](#exceptions)
  - [Performance](#performance)
  - [Plotting](#plotting)
  - [DataFrames](#dataframes)
  - [References](#references)

## Basics

The minimal "hello world" program:

```julia
# Single line comment

#=
Multi-line comment
=#

println("Hello, World!")
```

Indentation doesn't matter. Indexing starts at 1, like Matlab and Octave.  
In the REPL, by pressing "]" you can enter the "package mode", where you can write commands that manage the packages you have or want. Some commands:  

- `status`: Retrieves a list with name and versions of locally installed packages
- `update`: Updates your local index of packages and all your local packages to the latest version
- `add myPkg`: Automatically downloads and installs a package
- `rm myPkg`: Removes a package and all its dependent packages that has been installed automatically only for it
- `add pkgName#master`: Checkouts the master branch of a package (and free pkgName returns to the released version)
- `add pkgName#branchName`: Checkout a specific branch
- `add git@github.com:userName/pkgName.jl.git`: Checkout a non registered pkg

To use a package on a Julia script, write `using [package]` at the beginning of the script. To use a package without populating the namespace, write `import [package]`. But then, you will have to use the functions as `[package].function()`. You can also include local Julia scripts as such: `include("my_script.jl")`.  

I think that `using [package]` is bad practice. The best way to import a package is this:

```julia
# Importing the JSON package through an alias 
# BEFORE JULIA 1.6
import JSON; const J = JSON

# Importing the JSON package through an alias 
# AFTER JULIA 1.6
import JSON as J

# Using:
J.print(Dict("Hello, " => "World!"))
```

## Data Types and Structures

Some built-in data structures of the Julia language:

### Scalar Types

The usual scalar types are present: Int64, Float64, Char, String and Bool. We use single quote for chars and double quote for strings. Some string operations are also present, like:

- `split`: Separates string into other strings based on a char. Default char is whitespace.
- `join([string1, string2], "")`: Concatenates strings with a certain string.
- `replace(s, "toSearch" => "toReplace")`: Replaces occurrences on the string s.
- `strip(s)`: Remove leading and trailing whitespaces.

Other ways to concatenate strings:

- Concatenation operator: `*`;
- Function `string(string1,string2,string3)`;
- Interpolate string variables in a bigger one using the dollar symbol: `a = "$str1 is a string and $(myobject.int1) is an integer"`.

### Arrays

Arrays are N-dimensional mutable containers. Ways to create one:

- `a = []` or `a = Int64[]` or `a = Array{T,1}()` or `a = Vector{T}()`: Empty array. Array{} is the constructor, T is the type and Vector{} is an alias for 1 dimensional arrays.
- `a = zeros(5)` or `a = zeros(Int64,5)` or `a = ones(5)`: Array of zeros (or ones).
- `a = [1,2,3]`: Explicit construction.
- `a = [10, "foo", false]`: Can be of mixed types, but will be much slower.

If you need to store different types on a data structure, better to use an Union: `a = Union{Int64,String,Bool}[10, "Foo", false]`.  
Some operations on arrays:

- `a[1]`: Access element.
- `a[from:step:to]`: Slice
- `collect(myiterator)`: Transforms an iterator in an array.
- `y = vcat(2015, 2025:2028, 2100)`: Initialize an array expanding the elements. 2025:2028 means [2025, 2026, 2027, 2028].
- `push!(a,b)`: Append b to the end of a
- `append!(a,b)`: Append the elements of b to the end of a. If b is scalar, append b to the end of a.
- `a = [1,2,3]; b = [4,5]; c = vcat(1,a,b)`: Concatenation of arrays.
- `pop!(a)`: Remove element from the end of a.
- `popfirst!(a)`: Remove first element of a.
- `deleteat!(a, pos)`: Remove element at position pos from array a.
- `pushfirst!(a,b)`: Add b at the beginning of array a.
- `sort!(a) or sort(a)`: Sorting, depending on whether we want to modify or not the original array.
- `a[end:-1:1]`: Reverses array a.
- `in(1, a)`: Checks for existence.
- `length(a)`: Length of array.
- `maximum(a) or  max(a...)`: Maximum value. max returns the maximum value between the given arguments.
- `minimum(a) or  min(a...)`: Minimum value. min returns the minimum value between the given arguments.
- `isempty(a)`: Checks if an array is empty.

### Matrixes

A matrix is an array of arrays that have the same length. Ways to create one:

- `a = [[1,2,3] [4,5,6]]`: [[elements of the first column] [elements of the second column] ...].
- `a = hcat(col1, col2)`. By the columns.
- `a = [1 4; 2 5; 3 6]`: [elements of the first row; elements of the second row; ...].
- `a = vcat(row1, row2)`: By the rows.
- `a = zeros(2,3)` or `a = ones(2,3)`: A 2x3 matrix filled with zeros or ones.

Attention to the difference:

- `a = [[1,2,3],[4,5,6]]`: creates a 1-dimensional array with 2-elements.
- `a = [[1,2,3] [4,5,6]]`: creates a 2-dimensional array (a matrix with 2 columns) with three elements (scalars).

Access the elements with `a[row,col]`.  
You can also make a boolean mask and apply to the matrix:

```julia
a = [[1,2,3] [4,5,6]]
mask = [[true,true,false] [false,true,false]]
println(a[mask])
# Will print [1, 2, 5]. Always flattened.
```

Other useful operations:

- `size(a)`: Returns a tuple with the sizes of the n dimensions.
- `ndims(a)`: Returns the number of dimensions of the array.
- `a'`: Transpose operator.
- `reshape(a, nElementsDim1, nElementsDim2)`: Change dimensions.
- `dropdims(a, dims=(dimToDrop1,dimToDrop2))`: Change dimensions.

These last three operations performe only a shallow copy (a view) on the matrix, so if the underlying matrix changes, the view also changes. Use `collect(reshape/dropdims/transpose)` to force a deep copy.

### Tuples

Tuples are an immutable collection of elements. Initialize with `a = (1,2,3)` or `a = 1,2,3`. Tuples can be unpacked like so: `var1, var2 = (x,y)`. And you can convert a tuple into a vector like this: `v = collect(a)`.

### Dictionaries

Dictionaries mappings from keys to values. Ways to create one:

- `mydict = Dict()`
- `mydict = Dict('a'=>1, 'b'=>2, 'c'=>3)`

Useful operations:

- `mydict[key] = value`: Add pairs to the dictionary
- `mydict[key]`: Look up value. If it doesn't exist, raises error.
- `get(mydict,'a',0)`: Look up value with a default value for non-existing key.
- `keys(mydict)`: Get all keys. Results in an iterator. Use collect() to transform into array.
- `values(mydict)`: Iterator of all the values.
- `haskey(mydict, 'a')`: Checks if a key exists.
- `in(('a' => 1), mydict)`: Checks if a given key/value pair exists.

You can iterate over both keys and values:

```julia
for (k,v) in mydict
   println("$k is $v")
end
```

### Sets

A set is a collection of unordered and unique values. Ways to create one:

- `a = Set()`: Empty set.
- `a = Set([1,2,2,3,4])`: Initialize with values.
- `intersect(set1,set2)`, `union(set1,set2)`, `setdiff(set1,set2)`: Intersection, union, and difference.

### Memory and Copy

Shallow copy (copy of the memory address only) is the default in Julia. Some observations:

- When a variable receives other variable: Basic types (Float64, Int64, String) are deep copied. Containers are shallow copied.
- `copy(x)`: Simple types are deep copied, containers of simple types are deep copied, containers of containers, the content is shadow copied (the content of the content is only referenced, not copied).
- `deepcopy(x)`: Everything is deep copied recursively.

Observations on types:

- `convertedObj = convert(T,x)`: Casts variable x to type T.
- `myInt = parse(Int,"2017")`: Convert String into Int.
- `myString = string(123)`: Convert Int to String.

### Random Numbers

- `rand()`: Random float in [0,1].
- `rand(a:b)`: Random integer in [a,b].
- `rand(a:0.01:b)`: Random float in [a,b] with "precision" to the second digit.
- `rand(2,3)`: Random 2x3 matrix.

## Basic Syntax

The typical control flow is present:

```julia
for i = 1:5
    println(i)
end

for j in [1, 2, 3]
    println(j)
end

for i = 1:2, j = 3:4
    println((i, j))
end

i = 0
while i < 5
    println(i)
    global i += 1
end

if x < y
    println("x is less than y")
elseif x > y
    println("x is greater than y")
else
    println("x is equal to y")
end
```

There are list comprehensions:

```julia
[myfunction(i) for i in [1,2,3]]

[x + 2y for x in [10,20,30], y in [1,2,3]]

mydict = Dict()
[mydict[i]=value for (i, value) in enumerate(mylist)]
# enumerate returns an iterator to tuples with the index and the value of elements in an array

[students[name] = sex for (name,sex) in zip(names,sexes)]
# zip returns an iterator of tuples pairing two or multiple lists, e.g. [("Marc","M"),("Anne","F")]

map((n,s) -> students[n] = s, names, sexes)
# map applies a function to a list of arguments
```

The ternary operator is present:

```julia
a ? b : c
# If a is true, then b, else c
```

The usual logic operators exist:

- And: `&&`
- Or:  `||`
- Not: `!`

Functions can be declared like so:

```julia
function f(x)
    x+2
end
```

Julia has multiple-dispatch. If you declare the same function with different arguments, the compiler will choose the correct function to call based on the arguments you passed.  
You can also do type parametrization on functions:

```julia
function f(x::T)
    x+2
end

myfunction(x::T, y::T2, z::T2) where {T <: Number, T2} = 5x + 5y + 5z
```

Functions are objects that can be assigned to new variables, returned, or nested:

```julia
f(x) = 2x   # define a function f inline
a = f(2)    # call f and assign the return value to a
a = f       # bind f to a new variable name (it's not a deep copy)
a(5)        # call again the (same) function
```

The arguments of a function are normally passed by reference.  
Anonymous functions can be declared like so:

```julia
(x, y) -> x^2 + 2y - 1
# you can assign an anonymous function to a variable.
```

You can "broadcast" a function to work over all the elements of an array:

```julia
myArray = broadcast(i -> replace(i, "x" => "y"), myArray)
```

## Custom Types

You can define structures like this:

```julia
# Structs are immutable by default. Hence the mutable keyword.
# Immutable structs are much faster.
mutable struct MyStruct
  property1::Int64
  property2::String
end

# Parametrized:
mutable struct MyStruct2{T<:Number}
 property1::Int64
 property2::String
 property3::T
end

# Instantiating and accessing attribute:
myObject = MyStruct(20,"something")
a = myObject.property1 # 20
```

Attention to this:

- `a::B`: Means "a must be of type B".
- `A<:B`: Means "A must be a subtype of B".

An example of object orientation in Julia:

```julia
struct Person
  myname::String
  age::Int64
end

struct Shoes
    shoesType::String
    colour::String
end

struct Student
    s::Person
    school::String
    shoes::Shoes
end

function printMyActivity(self::Student)
    println("I study at $(self.school) school")
end

struct Employee
    s::Person
    monthlyIncomes::Float64
    company::String
    shoes::Shoes
end

function printMyActivity(self::Employee)
    println("I work at $(self.company) company")
end

gymShoes = Shoes("gym","white")
proShoes = Shoes("classical","brown")

Marc = Student(Person("Marc",15),"Divine School",gymShoes)
MrBrown = Employee(Person("Brown",45),1200.0,"ABC Corporation Inc.", proShoes)

printMyActivity(Marc)
printMyActivity(MrBrown)
```

Observations:

- Functions are not associated to a type. Do not call a function over a method (`myobj.func(x,y)`) but rather you pass the object as a parameter (`func(myobj, x, y)`)
- Julia doesn't use inheritance, but rather composition (a field of the subtype is of the higher type, allowing access to its fields).

Some useful functions:

- `supertype(MyType)`: Returns the parent types of a type.
- `subtypes(MyType)`: Lists all children of a type.
- `fieldnames(MyType)`: Queries all the fields of a structure.
- `isa(obj,MyType)`: Checks if obj is of type MyType.
- `typeof(obj)`: Returns the type of obj.

## I/O

Opening a file is similar to Python. The file closes automatically in the end:

```julia
# Write to file
open("file.txt", "w") do f  # "w" for writing, "r" for read and "a" for append.
    write(f, "test\n")      # \n for newline
end

# Read whole file:
open("file.txt", "r") do f
  filecontent = read(f,String)
  print(filecontent)
end

# Read line by line:
open("file.txt", "r") do f
    for ln in eachline(f)
        println(ln)
    end
end

# Read, keeping track of line numbers:
open("file.txt", "r") do f
    for (i,ln) in enumerate(eachline(f))
        println("$i $ln")
    end
end
```

## Metaprogramming

## Exceptions

Exceptions are similar to Python:

```julia
try
    # Some dangerous code...
catch
    # What to do if an error happens, most likely send an error message using:
    error("My detailed message")
end

# Check for specific exception:
function volume(region, year) 
    try
        return data["volume",region,year]
    catch e
        if isa(e, KeyError)
            return missing
        end
        rethrow(e)
    end
end
```

## Performance

Statically typing the program, or facilitating the type inference of the JIT compiler makes the code run faster. Some notes:

- Avoid global variables and run your performance-critical code within functions rather than in the global scope;
- Annotate the inner type of a container, so it can be stored in memory contiguously;
- Annotate the fields of composite types (use eventually parametric types);
- Loop matrices first by column and then by row.

Notes on profiling :

- To time a part of the code type `@time myFunc(args)` (be sure you ran that function at least once, or you will measure compile time rather than run-time).
- `@benchmark myFunc(args)` (from package BenchmarkTools) also works.
- Profile a function: `Profile.@profile myfunct()` (best after the function has been already ran once for JIT-compilation).
- Print the profiling results: `Profile.print()` (number of samples in corresponding line and all downstream code; file name:line number; function name;)
- Explore a chart of the call graph with profiled data: `ProfileView.view()` (from package ProfileView).
- Clear profile data: `Profile.clear()`.

## Plotting

The Plots package provides an unified API to several supported [backends](http://docs.juliaplots.org/latest/backends/). Install the packages "Plots" and at least one backend, like "PlotlyJS" or "PyPlot.jl". Example:

```julia
using Plots
plotlyjs()
plot(sin, -2pi, pi, label="sine function")
```

## DataFrames

Examples:

```julia
# Read data from a CSV
using DataFrames, CSV
myData = CSV.read(file; delim=';', missingstring="NA", delim=";", decimal=',', copycols=true)

# Read data from the web:
using DataFrames, HTTP, CSV
resp = HTTP.request("GET", "https://data.cityofnewyork.us/api/views/kku6-nxdu/rows.csv?accessType=DOWNLOAD")
df = CSV.read(IOBuffer(String(resp.body)))

# Read data from spreadsheet:
using DataFrames, OdsIO
df = ods_read("spreadsheet.ods";sheetName="Sheet2",retType="DataFrame",range=((tl_row,tl_col),(br_row,br_col)))

# Empty df:
df = DataFrame(A = Int64[], B = Float64[])
```

Insights about the data:

- `first(df, 6)`
- `show(df, allrows=true, allcols=true)`
- `last(df, 6)`
- `describe(df)`
- `unique(df.fieldName)` or `[unique(c) for c in eachcol(df)]`
- `names(df)`: Returns array of column names
- `[eltype(col) for col = eachcol(df)]`: Returns an array of column types
- `size(df)`: (r,c); `size(df)[1]`: (r); `size(df)[2]`: (c).
- `ENV["LINES"] = 60`: Change the default number of lines before the content is - truncated (default 30).
- `for c in eachcol(df)`: Iterates over each column.
- `for r in eachrow(df)`: iterates over each row.

To query the data from a DataFrame you can use the Query package. Examples:

```julia
using Query

dfOut = @from i in df begin
           @where i.col1 > 1
           @select {aNewColName=i.col1, i.col3}
           @collect DataFrame
        end
 dfOut = @from i in df begin
            @where i.value != 1 && i.cat1 in ["green","pink"]
            @select i
            @collect DataFrame
        end
```

## References

- [Julia language: a concise tutorial](https://syl1.gitbook.io/julia-language-a-concise-tutorial/).
- Antonello Lobianco. Julia Quick Syntax Reference. 1st Edition. Apress.
