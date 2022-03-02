+++
title = "Lists in OCaml"
date = 2022-01-27

[taxonomies]
categories = ["Principles of Programming Languages"]
tags = ["cs314"]
+++
Lists are the most basic data structure in OCaml. The most analoguous structure is a vector in C++ or Rust. Lists are *homogenous* and of *arbitrary length*.

<!-- more -->

The most basic list is **nil**, the empty list: `[]`. We can prepend elements to a list through the **cons** operator: `::`. Every display of a list is actually just syntactic sugar for every element being cons with the empty list:

```ocaml
[1, 2, 3] = 1 :: 2 :: 3 :: []
```

Importantly, lists are immutable! When they are created, they cannot be changed. If you want to add an element to a list, you must construct a new list based on the old one.

> This might seem inefficient, to create a new list every time we want to change it. But the semantics of a language do not necessarily correspond to its compiled execution. In particular, the invariant that lists are immutable can lead to optimizations where a compiler doesn't need to keep track of mutated information.

It is convention to call the right hand side the *tail* and the left hand side the *head*. The cons operator has an element on the left side and an list on the right side.

Let's think of lists in terms of expressions. `[]` is a value like 0 or 1. The cons operator evaluates the left expression to an element and the right expression to a list of type element. Like with function arguments, the order of this evaluation is irrelevant as long as we don't introduce side effects.

Lists can also have the **polymorphic** type `'a`. This is similar to a generic type in languages like Java, C++, and Rust. An `'a` list can have any type. This is most useful when it comes to function types. If the function type accepts or returns a `'a` type, then it is a generic function that can be used on a list of any type. Nil is a `'a` list.

You can also have nested lists like this:

```ocaml
let m = [[1]; [2; 3]] ;; (* int list list *)
```

Unlike tuples, these lists do not have to be the same size. They do need to have the same type, however, so this is a list of int lists, or an `int list list`.

The cons operator *does not* work to concatenate two lists. It can only add one element to an existing list. This is a very common bug (at least for myself). OCaml has a special sugar operand `@` to concatenate lists, although it is technically an ordinary function.
