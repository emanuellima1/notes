# Notes on Numpy

## Table of Contents

- [Notes on Numpy](#notes-on-numpy)
  - [Table of Contents](#table-of-contents)
  - [Basics](#basics)

## Basics

Basic functions:

```python
import numpy as np

# Creating arrays
np.array([1000, 2300, 4987, 1500])    # Create array from list: array([1000, 2300, 4987, 1500])
np.arange(10)    # array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
np.arange(10, 25, 5)    # array([10, 15, 20])
np.loadtxt(fname='file.txt', dtype=int)    # Creates an array of ints with the contents of file.txt
np.zeros((m, n))    # Create an array of zeros with shape (m, n)
np.ones((m, n),dtype=np.int16)    # Create an array of ones with shape (m, n) and type dtype
np.linspace(0,2,9)    # Create an array of evenly spaced values: array([0., 0.25, 0.5 , 0.75, 1., 1.25, 1.5, 1.75, 2.])
np.full((m, n), a)    # Create an array with shape (m, n) whose elements are always a
np.eye(m)    # Create an identity matrix with dimension m
np.random.random((m, n))    # Create a random matrix with dimension (m, n). Random numbers between 0 and 1
np.empty((m, n))    # Create an empty array with dimension (m, n). The data is garbage.

# Attributes
data_array.dtype    # Returns the type of the elements of data_array
data_array.shape    # Returns the dimensions of the array
data_array.ndim    # Returns the number of dimensions of the array
data_array.size    # Returns the number of elements of the array
```

Operations with floats and integers broadcast to the whole array:

```python
a = np.ones(4, dtype=int)
a/2
# >>> array([0.5, 0.5, 0.5, 0.5])
```

Selecting data:

```python
a = np.random.random((2, 2))
a[1, 1]    # Returns the element on the second line and second column
a[-1]    # Returns the last line

b = np.random.random((10))
b[1:4]    # Returns the elements with index 1, 2 and 3
# The syntax is array[min:max:step]. min is 0 by default, max is not included.
b[:4:2]    # Returns the elements with index 0 and 2

c = np.random.random((10, 10))
c[:, 1:3]    # Returns the columns 1 and 2 of all the lines
c[:4, ::2]    # Returns the first 4 lines (indexes 0, 1, 2 and 3) with columns jumped by 2 (indexes 0, 2, 4, 6, 8)
```

Verifying conditions:

```python
a = np.random.random((4))
a > 1    # Returns a boolean array: array([False, False, False, False])
a[a > 0.5]    # Returns an array containing all numbers that obey the condition
```

Some methods:

```python
a.T    # Returns the transpose of the array
a.tolist()    # Converts the array to a list
a.reshape((m, n), order='C')    # Reorganizes the array into a new array with dimensions (m, n)

# Example:
a = np.arange(10)
a.reshape((5, 2), order='C')    # >>> array([[0, 1], [2, 3], [4, 5], [6, 7], [8, 9]])
a.reshape((5, 2), order='F')    # >>> array([[0, 5], [1, 6], [2, 7], [3, 8], [4, 9]])

# array + array is element-wise sum of arrays
# list + list is concatenation of lists

np.column_stack((a, b, c))    # Take a tuple of 1D arrays and stack them as columns to make a single 2D array.
np.sum(a[:,2])    # Returns the sum of all the elements of the slice
```

Some basic statistics:

```python
np.mean(a)    # Returns the mean of all the elements of the array
np.mean(a[:, 2])    # Returns the mean of all the elements of the slice
np.mean(a, axis=0)    # Returns the mean of all the elements of the array in the axis 0

np.std(a[:,2])    # Returns the standard deviation of the slice
```
