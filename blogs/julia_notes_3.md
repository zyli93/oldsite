Julia Note (3)
==============

Now we finished Chapter 1 to Chapter 7. The only question remains is the scope
question of the keyword `global`. To me, this problem is shitty and need to be
clearly understood before using Julia in my research.

# Varible, Expressions, and Statements

## Operator predecence
The order is _PEMDAS_: parentheses, exponential, multiplication, 
division, addtion and subtraction.

## String Operations

`*` and `^` are operators for strings. `*` is for string concatenation.
`^` duplicate the string by n times.
```jl
> "spam"^3
"spamspamspam"
```

# Functions
A string can also take in a variable type, e.g., `parse()` and `trunc()`.
```
> parse(Int64, "32")
32
> parse(Float64, "3.14159")
3.14259
> trunc(Int64, -2.3)  # convert float to integer, but doesn't round off
-2
> float(32)
32.0
> string(32)
"32"
```

## _Fruitful_ and _Void_ functions
If assign void function to a variable, you will see `nothing`.
To print the value `nothing`, you have to use the function `show`.
`show` is like `print` but can handle the value `nothing`.

## Docstring
Very similar as the Python.

## Words
* _module_: a file that contains a collection of related functions and other
    definition.
* _package_: an external library
* `using` _statement_: a statement that reads a module file and creates a module
    object.

# Conditionals and Recursion
`\div + TAB` gives `÷`, which is the floor division in Julia. `%` is the modulus
operator.
For conditionals, it goes like `if`, `elseif`, and finally `else`.
A semi-colon `;` allows to put multiple statements on the same line.

## Fruitful functions
By default, it will return the last statement. Sometimes, it is useful to have
multiple `return` statements.
Use `@show` macro to show the intermediate results inside a function.
Use `isa` to check the variable type and use `error("err msg")` to throw and
error.


# Iteration
```jl
while n > 0
    some statement
end
```
Julia also have `break` and `continue`

# String
A char is in `'x'`. A string is in `""`. It's a sequence.
All indexding in Julia is 1-based -- the first element of any integer-indexed
object is fount at index 1 and the last element at index `end`.
Build-in function `length` that returns the number of characters in a string.
Some of the UTF-8 characters encoded by 4 Bytes, which means that not every byte
index into a UTF-8 string is necessarily a valid index for a character.

* String slices
```
> str = "Julius Caesar"
> str[1:6]
"Julias"
```
* String are immutable.
The strings are immutable, which means you can't change an existing string. But
you can create a new string.
* String interpolation
Julia allows string interpolation using `$`.
```jl
> greet = "Hello"
> whom = "world"
> "$greet, $(whom)!"
"Hello, world!"
```
The shortest complete expression after the $ is taken as the expression 
whose value is to be interpolated into the string. 
Thus, you can interpolate any expression into a string using parentheses:
```
> "1 + 2 = $(1+2)"
"1 + 2 = 3"
```

* Searching a char in string
```jl
function find(word, letter)
    index = firstindex(word)  # firstindex() function
    while index <= sizeof(word)  # sizeof() function
        if word[index] == letter
            return index
        end
        index = nextind(word, index)  # nextind() function
    end
    -1
end
```

* string library
`uppercase()`, `findfirst()`, `findnext()`.



## The scope of the variables
Key words that will create new scope blocks:
`function`, `while`, `for`, `try`, `catch`, `let`, `type`. 
Note that `begin` cannot create new scope.
In `function` scope, there is no need to use `global` keyword.
```jl
function foo(n)
    counter = 1
    for i in 1:4
        counter += 1  # this works
    end
end
```
The following doesn't work
```jl
counter = 1
for i in 1:4
    counter += 1  # cannot actually add
end

# TO DID THE ADD
counter = 1
for i in 1:4
    global counter += 1
end
```

Ways of introducing variables to current scope:
1. declaring `local x` or `const x` to introduce local variable
2. declaring `global x` introduce `x` to current scope
3. params of functions

In non-outer-most scope, the only way to declare a global variable is by
explicitly declaration. Otherwise, assignments will introduce new local
variables.

__Functions can be called before defined__

How to declare a global variable:
```jl
function foo()
    global x=1, y="bar", z=3
end
```

`let` is another way of introducing new variables. It's usage:
```
let var1 = value1, var2, var3 = values
    code
end
```
Check out the following statements:
```
# This section has not been verified yet.
fs = cell(2)
i = 1
while i <= 2
    fs[i] = () -> i
    i += 1
end

> fs[1]()
3
> fs[2]()
3
```
However, in another example, we have
```
# This part has not been verified yet.
fs = cell(2)
i = 1
while i <= 2
    let i = i
        fs[i] = () -> i  # here i is not the original i
    end
    i += 1
end

> fs[1]()
1
> fs[2]()
2
```

For Loop and Comprehensions. Both for loop and comprehension will not
carry the old value.

Use `const` on global variables. If a global variable doesn't change the value,
declaring it as the `const` will be more efficient.

Here is an unsolved problem:
```
i = 0
while i <= 2
    println(i)
    global i += 1
end
println(i)
```
why the first `println` doesn't need to do `global`.
