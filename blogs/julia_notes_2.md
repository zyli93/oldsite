Julia Notes (2)
=============

The following notes are taken from this [great book](https://benlauwens.github.io/ThinkJulia.jl/latest/book.html)

Summary: we started to follow the [ThinkJulia.jl](https://benlauwens.github.io/ThinkJulia.jl/latest/book.html) book by Ben Lauwens.
Now we basically now everything about Chapter 15 to Chapter 18. 
The next chapter to be learned is the former ones.

# REPL
Press "?" for help.

# Type System
Julia does not support Python/Cpp style OOP where object carries member method and inheritance. It instead uses the Multiple Dispatch property to disable the usage of member method in object.

1. Composite type
```
struct <type name>
    ...
end
```
The assignment of composite type is by default uses the pointer style. Access by ".".
```
> anm = Animal(100, "f")
Animal(100, "f")
> anm.sex
"f"
```
Another example:
```
struct Point
    x
    y
end
```
A struct is like a factory for creating objects. To create a point, you call `Point` as if it were a function having as arguments the values of the fields. When Point is used as a function, it is called a constructor.
```
> p = Point(3.0, 4.0)
Point(3.0, 4.0)
```
`struct` are immutable. Structs are however by default immutable, after construction the fields can not change value.
```
> p.y = 1.0
ERROR: setfield! immutable struct of type Point cannot be changed
```

2. Mutable Structs
```jl
mutable struct MPoint
    x
    y
end
```
If a mutable struct object is passed to a function as an argument, the function can modify the fields of the object. For example, movepoint! takes a mutable Point object and two numbers, dx and dy, and adds the numbers to respectively the x and the y attribute of the Point:
```
function movepoint!(p, dx, dy)
    p.x += dx
    p.y += dy
    nothing  # ?
end
```
(__TODO__: find out what is `nothing`?)
*You cannot reassign a mutable attribute of an immutable object*

* Copying
`deepcopy` function deep copies `p1` to `p2`.
```jl
> p1 = MPoint(3.0, 4.0)
MPoint(3.0, 4.0)
> p2 = deepcopy(p1)
MPoint(3.0, 4.0)
> p1 ≡ p2
false
> p1 == p2
false
```
(≡: `\equiv`, identical to "===". For mutable objects, the default behavior of the == operator is the same as the === operator; it checks object identity, not object equivalence.)

Use `isa` to check whether an object is an instance of a type. Use `fieldnames` to check field names. Use `isdefined` to check whether a particular attribute/field is defined.
```
> p isa Point
true
> fieldnames(Point)
(:x, :y)
> isdefined(p, :z)
false
> isdefined(p, :x)
true
```



2. Assigned type
By "::", can speed up the execution.

3. Abstract type
Abstract can help control the interface.
```
abstract struct AbstractAnimal
end
```
Composite type can be a sub-type of an abstract type, but cannot be a sub-type of an composite type.

4. Immutable type
Mutable elements in immutable type is still mutable.


## Multiple Dispatch

### Type declarations: "::":
```
> function returnfloat()
      x::Float64 = 100
      x
  end

> function sinc(x)::Float64
      if x == 0
          return 1
      end
      sin(x)/(x)
  end

> using Printf
> function printtime(time::MyTime)  # This is a method.
      @printf("xxx", time.hour, time.minute, time.second)
  end
```

### Methods
A method is a function definition with a specific signature.
By the way, optional arguments are implemented as syntax for multiple method definitions. For example, this definition:
```jl
> function f(a=1,b=2)
      a + 2b
  end
```

### Constructors
* Default constructor
```
MyTime(hour, minute, second)
MyTime(hour::Int64, minute::Int64, second::Int64)
```
* Outer constructors, (aka copy constructor)
```
function MyTime(time::MyTime)
    MyTime(time.hour, time.minute, time.second)
end
```
* Inner constructors to enfore invariants
```
struct MyTime
    hour :: Int64
    minute :: Int64
    second :: Int64
    function MyTime(hour::Int64=0, minute::Int64=0, second::Int64=0)
        @assert(0 ≤ minute < 60, "Minute is not between 0 and 60.")
        @assert(0 ≤ second < 60, "Second is not between 0 and 60.")
        new(hour, minute, second)
    end
end
```
This way, `MyTime` has 4 inner constructor methods. An inner constructor method is always defined inside the block of a type declaration and it has access to a special function called new that creates objects of the newly declared type. The default constructor is not available if any inner constructor is defined. You have to write explicitly all the inner constructors you need.

A second method without arguments of the local function `new` exists: 
```
mutable struct MyTime
    hour :: Int
    minute :: Int
    second :: Int
    function MyTime(hour::Int64=0, minute::Int64=0, second::Int64=0)
        @assert(0 ≤ minute < 60, "Minute is between 0 and 60.")
        @assert(0 ≤ second < 60, "Second is between 0 and 60.")
        time = new()
        time.hour = hour
        time.minute = minute
        time.second = second
        time
    end
end
```
This allows to construct recursive data structures, i.e. a struct where one of the fields is the struct itself. In this case the struct __has to be mutable__ because its fields are modified after instantiation.


__`show`__
`show` is a special function that returns a string representation of an object. For example, here is a show method for `MyTime` objects:
```
using Printf

function Base.show(io::IO, time::MyTime)
    @printf(io, "%02d:%02d:%02d", time.hour, time.minute, time.second)
end
```
The prefix `Base` is needed because we want to add a new method to the `Base.show` function. When you print an object, Julia invokes the `show` function:
```
> time = MyTime(9, 45)
09:45:00
```
Notes: When I write a new composite type, I almost always start by writing an outer constructor, which makes it easier to instantiate objects, and `show`, which is useful for debugging.


### Operator Overloading
```
import Base.+

function +(t1::MyTime, t2::MyTime)
    seconds = timetoint(t1) + timetoint(t2)
    inttotime(seconds)
end
```

### Multiple Dispatch
The choice of which method to execute when a function is applied is called dispatch. Julia allows the dispatch process to choose which of a function’s methods to call based on the number of arguments given, and on the types of all of the function’s arguments. Using all of a function’s arguments to choose which method should be invoked is known as multiple dispatch. Examples are:
```
function +(time::MyTime, seconds::Int64)
    increment(time, seconds)
end

function +(seconds::Int64, time::MyTime)
  time + seconds
end
```

### Debugging
Julia allows to introspect the signatures of the methods of a function. To know what methods are available for a given function, you can use the function methods:
```
> methods(printime)
# 2 methods for generic function "printtime":
[1] printtime(time::MyTime) in Main at REPL[3]:2
[2] printtime(time) in Main at REPL[4]:2
```


## Subtyping

### Global variables
```jl
const suit_names = ["♣", "♦", "♥", "♠"]
const rank_names = ["A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"]
```
We introduced _multiple dispatch mechanism_ and _polymorpich methods_.
`@test`: unit test macro. `@assert` assert macro.
```jl
struct Deck
    cards :: Array{Card, 1}  # create empty array, {T N}
end

function Deck()
    deck = Deck(Card[])
    for suit in 1:4
        for rank in 1:13
            push!(deck.cards, Card(suit, rank))  # push funtion, modifier
        end
    end
    deck
end
```

### Add, Remove, Shuffle, and Sort
veneer function: A method like this that uses another method without doing much work. See [here](https://benlauwens.github.io/ThinkJulia.jl/latest/book.html#_add_remove_shuffle_and_sort)

### Abstract Types and Subtyping
Basically rewrite what we just wrote by using subtyping.
```jl
abstract type CardSet end

struct Deck <: CardSet
    cards :: Array{Card, 1}
end

function Deck()
    deck = Deck(Card[])
    for suit in 1:4
        for rank in 1:13
            push!(deck.cards, Card(suit, rank))
        end
    end
    deck
end
```
Use the operator `isa` to check whether an object is of a given type:
```
> deck = Deck()
> deck isa CardSet
true
```

`@which` macro
```
> @which sort!(hand)
sort!(hand::Hand) in Main at REPL[5]:1
```
So the `sort!` method for `hand` is the one having as argument an object of type `Hand`.The function `supertype` can be used to find the direct supertype of a type.
```
> supertype(Deck)
CardSet
```

Development plan for designing types:

* Start by writing functions that read and write global variables (when necessary).
* Once you get the program working, look for associations between global variables and the functions that use them.
* Encapsulate related variables as fields of a struct.
* Transform the associated functions into methods with as argument objects of the new type.


[Back to Blog Content](./blog_index.html)
[Back to Home](../index.html)