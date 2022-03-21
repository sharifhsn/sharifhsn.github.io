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

## Pattern Matching

**Pattern matching** is an extremely important concept in OCaml for functional programming. It allows for powerful destructuring of enums, collections, and other complex objects.

```ocaml
let head l =
    match l with
    | (h :: _) -> h
```

This function `head` here matches the list `l` where there is an element `h` that is cons with an arbitrary expression. The `_` indicates that some expression must be present, but we don't care about what expression it is, and in fact we are throwing it away. `h` is also an arbitrary expression, but we are using it as the return type.

This match statement is not *exhaustive*. An exhaustive match means that there is an evaluation for every possible case of `l`. Here, there is no case for when `l` is nil. This does not work in OCaml; every match must be exhaustive. Here is an example of an exhaustive match:

```ocaml
let rec sum l =
    match l with
    | [] -> 0
    | h :: t -> h + sum t
```

This `sum` function works for every case of `l`. Either it is nil, or it is a head cons a tail. You will also notice here that the two cases resolve to the same type. This is another requirement for match statements: all patterns must evaluate to the same type, which is not necessarily the type of the initial expression. Here, the initial expression has the type `int list` and the return expression has the type `int`. This function only works for `int list`. This restriction is not present for our `head` function because `h` is polymorphic type `'a`.

Pattern matching is generally much better than its alternatives. OCaml will warn you if your function is non-exhaustive, and it will throw an exception for an unhandled case. Also, duplicated cases are easy to avoid becaues OCaml will give a similar warning for unused cases.

## Recursion with Lists

In order to manipulate lists in any significant way, we need to use recursion. Lists are immutable, so functions over them must be recursive.

```ocaml
let rec length l =
    match l with
    | [] -> 0
    | (_ :: t) -> 1 + length t
```

We can think of this as recursive because we have our base case of nil and our iterative step of applying the function to successive tails. This function does not use `h`, but we might, as in the `sum` function above. Recursion might seem like it incurs overhead here, as in C-like languages recursion typically costs stack frames. However, OCaml knows that even though you are writing your code recursively, all you're doing is iterating over an immutable list, so it will elide those issues. Such recursive functions are called **tail-recursive** and are essential to the use of functional languages.

However, these functions work through the lists forward, like a linked list. How can we do something like reversing a list?

```ocaml
let rec rev_aux l acc =
    match l with
    | [] -> acc
    | x :: xs -> rev_aux xs (x :: acc)
let rev l = rev_aux l []
```

The key here is the variable `acc`, which is known as an **accumulator**. This is a variable that is included with each recursive function call that accumulates operations on it. This is a way of simulating side effects in a controlled way which is often useful. `acc` will accumulate heads while the tail grows smaller, and the actual `rev` function will get the accumulated nil that it passed to `rev_aux`.
