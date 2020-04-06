# Notes on Julia

## Table of Contents

- [Notes on Julia](#notes-on-julia)
  - [Table of Contents](#table-of-contents)
  - [Basics](#basics)
  - [Data Structures](#data-structures)
    - [Arrays](#arrays)
    - [Matrixes](#matrixes)
    - [Tuples](#tuples)
    - [Dictionaries](#dictionaries)
    - [Sets](#sets)
    - [Memory and Copy](#memory-and-copy)
    - [Random Numbers](#random-numbers)
  - [Basic Syntax](#basic-syntax)
  - [Structures](#structures)
  - [I/O](#io)
  - [Exceptions](#exceptions)
  - [Metaprogramming](#metaprogramming)
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
The usual scalar types are present: Int64, Float64, Char, String and Bool. We use single quote for chars and double quote for strings. Some string operations are also present, like:

- `split`: Separates string into other strings based on a char. Default char is whitespace.
- `join([string1, string2], "")`: Concatenates strings with a certain string.
- `replace(s, "toSearch" => "toReplace")`: Replaces occurrences on the string s.
- `strip(s)`: Remove leading and trailing whitespaces.

Other ways to concatenate strings:

- Concatenation operator: `*`;
- Function `string(string1,string2,string3)`;
- Interpolate string variables in a bigger one using the dollar symbol: `a = "$str1 is a string and $(myobject.int1) is an integer"`.

## Data Structures

Some built-in data structures of the Julia language:

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

## Structures

## I/O

## Exceptions

## Metaprogramming

## Performance

## Plotting

## DataFrames

## References

- [Julia language: a concise tutorial](https://syl1.gitbook.io/julia-language-a-concise-tutorial/).
