Julia learning notes
=======

# Introduction

This blog is a summary of my learning of Julia. 
A trending programming language dedicated for scientific computation.
More resources for Julia is available [here](https://julialang.org/).
All the code blocks with input/output are done through [JuliaPro](https://juliacomputing.com/),
which is the Anaconda of Julia.
Julia version is 1.0.3.

# Basic syntax

## Hello World
First thing first, print "hello world"
```jl
> println("Hello World")
```
__Note__: double quote mark for string, single quote mark for characters.
__Note__: The variable ans is set to the value of the last expression evaluated in an interactive session. This does not occur when Julia code is run in other ways.



## Variable Naming
```jl
> name = "Roger"
> println("Hello $name")
```
`name` - variable. Simliar to Python, there's no need to declare a var before using it. To insert the var from a print, use `$`.
Allowed chars for Var naming in Julia are diverse. Amazingly, characters in LaTex are also supported in Julia as valid var names. E.g., type `\epsilon` + `Tab` in REPL, you will see
```jl
> ϵ
```
Check out [this part](https://docs.julialang.org/en/v1/manual/variables/#Stylistic-Conventions-1) for more naming conventions. Specifically, the following worth noticing.
> Functions that write to their arguments have names that end in !. These are sometimes called "mutating" or "in-place" functions because they are intended to produce changes in their arguments after the function is called, not just return a value.

## Numbers
For Float64 systems, we can use the E-notation. 
```jl
> 1e10
> 1e-4
```

## Functions, For, and Branch

Let's start from an easy function -- summation of `A`.
```jl
function mysum(A)
    s = 0.0 # s = zero(eltype(A))
    for a in A
        s += a
    end
    s
end
```
Declaration of functions starts from `function` keyword and ends at `end` keyword. All Julia functions return the last line of the funtion. `return` is also fine but only used for returning in the middle of the function.
For loop also ends with `end` and is used together with `in`. `in` can also check whether an element is in a collection.
```jl
> 1 in [1,2,3]
true
```

## Comments
Single line comments use `#`. 
Multiple line comments:
```jl
#=
 xxx
=#
```

## Types
Julia is typed language. But we can also note a `type` instead of `declare` a type. E.g.,
```jl
is_even(x::Int) = x % 2 == 0
```

Julia documentation system:
```
"""
    is_even(x::Int) -> Bool

    Check if a number is even
"""
is_even(x::Int) = x % 2 == 0
```
For more details, check out `Documenter.jl`.

The `if-elseif-else` statements:
```
if cond1
    # ...
elseif cond2
    # ...
else
    # ...
end
```
Use `typeof()` to print the type of the variable.

## Multi-Dim Arrays
```
> rand(10)  # vector of 10 float numbers
> rand(10, 20)  # 2-D array of float numbers
> rand(Int, 10)  # int vector
```

Julia has similar function as List Comprehension in Python.
```
> [i for i in 1:10]
10-element Array{Int64,1}:
  1
  2
  3
  4
  5
  6
  7
  8
  9
 10
> [(i,j) for i in 1:3, j in 1:3]
3×3 Array{Tuple{Int64,Int64},2}:
 (1, 1)  (1, 2)  (1, 3)
 (2, 1)  (2, 2)  (2, 3)
 (3, 1)  (3, 2)  (3, 3)
```
Now we know tuples in Julia. There's a new type of tuple called `NamedTuple`, just introduced from Julia 1.0.
```
> info = (name = "Roger", age = "0")
```
To access these elements, use the operator `.`.
```
> info.name
"Roger"
```

##  Arrays
Creating a 1-dimensional array:
```
> v = [1,2,3,4]  # use "," to create 1D array, col vector
```
1-D array will give a column vector as opposed to numpy which by default creates a row vector.
Use space to create row vector.
```
> v = [1 2 3 4]  # row vector
> v = [1 2; 3 4]  # ";" and "\n" to separate lines
```
`Space` is equivalent to `hcat`, concatenate elements horizontally. `;` and `\n` are equivalent to `vcat`, concatenate elements vertically.

Indexing of arrays,
* 0-D is theoretically infinite-D. But only valid for 1.
```
> 2[1]
2
> 2[2]
ERROR: BoundsError
Stacktrace:
 [1] getindex(::Int64, ::Int64) at ./number.jl:78
 [2] top-level scope at none:0
```
* Use `[i_1, i_2, ..., i_k]` as the indices. `i_k` can be one of the following:
1. scalar
2. Range objects in form of `:`, `a:b`, or `a:b:c`.
3. any integer vectors including empty vector `[]`
4. bool vector

* Also assign values to array is possible.
Special case, if `X` is an array with dim `(length(I_1), length(I_2), ..., length(I_n))`, then `A`'s value at `i_1,i_2,...,i_n` would be revised to `X[I_1[i_1], I_2[i_2], ..., I_n[i_n]]`. If `X` is not an array, then its value would go to all indexed places.
```
> setindex!(A, X, I_1, I_2, ..., I_n)
```

## Comprehension and generator expression
__NOTE__: Julia range object looks like `start:step:stop`. `step` is in the middle.
```
> [i+j for i in 1:3 for j in 1:3]
9-element Array{Int64,1}:
 2
 3
 4
 3
 4
 5
 4
 5
 6
```

For MD array:
```
> [i+j for i in 1:3, j in 1:3]
3×3 Array{Int64,2}:
 2  3  4
 3  4  5
 4  5  6
```
Adding a guard will destory the order.
```
> [i+j for i in 1:3, j in 1:3 if iseven(i+j)]
5-element Array{Int64,1}:
 2
 4
 4
 4
 6
```
But doing the guard in the front will save the problem.
```
> [(iseven(i+j) ? 1 : 2) for i in 1:3, j in 1:3]
3×3 Array{Int64,2}:
 1  2  1
 2  1  2
 1  2  1
```
##  Broadcasting in Multi-dim Arrays
Broadcasting is supported in all MD arrays in Julia that uses the standard interface. E.g.,
```
> sin.(A)  # broadcast sin to all elements in A
```
Any function can support broadcast, e.g.,
```
> foo.(A,B,c)  # Here A and B are MD array
```
applies `foo(a,b,c)` to elements in `a in A` and `b in B` by broadcasting. A and B are not necessarily to have same size.

##  Arithmetic Operators
Others are pretty much the same as Python.
```
> x ^ y  # raise x to the y-th power
```
For every binary operation like `^`, there is a corresponding "dot" operation `.^` that is automatically defined to perform `^` element-by-element on arrays. 
```
> [1,2,3] .^ 3
3-element Array{Int64,1}:
  1
  8
 27
 ```
It performs a broadcast operation: it can combine arrays and scalars, arrays of the same size (performing the operation elementwise), and even arrays of different shapes (e.g. combining row and column vectors to produce a matrix). Moreover, like all vectorized "dot calls," these "dot operators" are fusing.
Furthermore, "dotted" updating operators like a .+= b (or @. a += b) are parsed as a .= a .+ b, where .= is a fused in-place assignment operation.
Spaces must be used around the operator in such cases to avoid ambiguity.

Other operators: `isequal(x,y)`, `isfinite(x)`, `isinf(x)`, and `isnan(x)`.

Chaining comparisons: comparisons can be arbitrarily chained such as 
```
> 1 < 2 <= 2 < 3 == 3 > 2 >= 1 == 1 < 3 != 5
```

Operator Precedence and Associativity. [here](https://docs.julialang.org/en/v1/manual/mathematical-operations/#Operator-Precedence-and-Associativity-1)

Finally, applying functions to vectorized form.
Julia provides a comprehensive collection of mathematical functions and operators. These mathematical operations are defined over as broad a class of numerical values as permit sensible definitions, including integers, floating-point numbers, rationals, and complex numbers, wherever such definitions make sense.
Moreover, these functions (like any Julia function) can be applied in "vectorized" fashion to arrays and other collections with the dot syntax f.(A), e.g. sin.(A) will compute the sine of each element of an array A.

## Numerical Conversions
1. `T(x)` or `convert(T,x)`.
2. `x % T`
3. `round(Int,x)` [here](https://docs.julialang.org/en/v1/manual/mathematical-operations/#Rounding-functions-1) are all the rounding functions.
And some division functions [here](https://docs.julialang.org/en/v1/manual/mathematical-operations/#Division-functions-1).
And some power, logs, and roots functions [here](https://docs.julialang.org/en/v1/manual/mathematical-operations/#Powers,-logs-and-roots-1).

## All objects but no classes
Type System, classes in Julia
```
Abstract Type
Concrete Type
    |-- Mutable Type
    |-- Immutable Type
```

Declarance:
```
abstract type AbstractType end

struct ImmuatbleType <: AbstractType
end

mutable struct MutableType <: AbstractType
end
```
All subtypes of an abstract type will build a tree in which the concrete types are always the leaf nodes. Multiple inheritance is not allowed in Julia to rigorously maintain the tree structure.

Multiple dispatch

[Back to Blog Content](./blog_index.html)
[Back to Home](../index.html)