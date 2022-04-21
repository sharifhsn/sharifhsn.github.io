+++
title = "A Crash Course in Python"

date = 2022-04-14



[taxonomies]

categories = ["Principles of Programming Languages"]

tags = ["cs314"]

+++

Python is a powerful jack-of-all-trades language that is much less limited than OCaml.

<!-- more -->

## Expressions

Arithmetic works in a very simple and clear way in Python, basically like a calculator.

```python
2 # 2
2 + 4 # 6
(2 + 5) * (3 - 5) # -20
```

Unlike in OCaml, operations can be performed between floats and integers with no issues; Python will simply change the types behind the scenes. This is called **type coercion**.

```python
2 + 3.5 # 5.5
```

This causes an interesting side effect: what is the return type of this function?

```python
def add(a, b):
    return a + b
```

In OCaml, the equivalent function would have type `int -> int -> int`, since the operation `+` is only valid for `int`. However, this function is actually polymorphic in Python! In OCaml, you would call it type `'a`. In Python, this type is called **Any**, and operation changes based on what the types are. The reason that this is possible is because *everything* in Python is an object. Essentially, the variables `a` and `b` are just boxes that could contain anything, and Python only checks whether the operation `+` is defined for the two variables at runtime.

## Strings

String manipulation is a very common operation in Python, so there are some very useful ways to handle strings built into the language.

```python
"hello " + "world" # "hello world"
"hello" * 3 # "hellohellohello"
```

You can also convert types to strings very easily.

```python
str(5) # "5"
str(3.5) # "3.5"
```

And vice versa.

```python
int("5") # 5
float("3.5") # 3.5
```

These are special built-in functions to make our lives earlier.

## Variables

Like with OCaml, there is no need to specify types as in Java. Unlike OCaml, Python does not use type inference. Instead, every variable can contain any type in a box as mentioned earlier.

```python
a = 3
b = "hello"
```

Variables in Python work differently under the hood than other languages. A variable is essentially just a name for an element.

```python
c = b
```

`c` here is not just equal to `b`... it is actually `b` itself! And any change you make to `b` will reflect in `c` because of that. Actual copies must be explicit.

## Slices

**Slices** are one of the most powerful tools in Python. In fact, it might be what Python is most known for and most useful for. Slicing allows for powerful manipulation of list-like data.

Ordinary list access uses bracket notation with a single number to access a single element. Slice notation works similarly, but it allows returning multiple elements as a sub-list of the original list.

```python
x = "hello world"
x[1:7] # "ello w"
```

You might notice that I just performed this operation on a string; didn't I just say that slices work on list-like types? In Python, strings are just fancy lists of chars!.

## Tuples and Lists

A tuple is an immutable set of multiple elements, just like in OCaml. Slicing operations work the same way in that they return a subsequence tuple. There's a weird side effect where if you slice in a way that returns a single element, you can get a single-element tuple.

Lists in Python are much more flexible in OCaml, as they can be heterogenous with any type within. They are also mutable, so elements of lists can be reassigned.

Slicing can superpower this assignment by reassigning multiple values at once

## Control

The traditional `if` statements are back, but with a bit of a twist. Python has *significant whitespace*, which means that the amount that you indent by affects the actual execution of the code. This is quite rare.

```python
if x == 15:
    y = 0 # this tab is mandatory!
x = 3 # this line is outside the if block
```

There is a special keyword called `pass` which exists to allow empty blocks. Normally, this construct is forbidden:

```python
if x == 15:
x = 3 # error!
```

But by using `pass`, this is possible:

```python
if x == 15:
    pass # does nothing
x = 3
```

Like in C, boolean evaluation is 0 for false, true for everything else. However, you *cannot* assign a variable in a conditional! So this C construct would not be allowed:

```c
while ((int x = some_func()) == 0) {
    // this is allowed in C
}
```

```python
while (x = some_func()) == 0:
    pass # this is not allowed in Python!
```

## Functions

Functions are defined using the `def` keyword.

```python
def fac(n):
    if n < 0:
        return "negative!"
    elif n == 0:
        return 1
    else:
        return n * fac(n - 1)
```

Functional programming can be used similarly to OCaml where everything is a function.

```python
def compose(f, g):
    def foo(x):
        return f(g(x))
    return foo
```

```ocaml
let compose f g =
    let foo x =
        f g x
    in foo x
```
