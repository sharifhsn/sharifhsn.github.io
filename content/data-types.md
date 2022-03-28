+++
title = "Data Types in OCaml"

date = 2022-02-17



[taxonomies]

categories = ["Principles of Programming Languages"]

tags = ["cs314"]

+++

When we make our programs more and more complex, we need more complex data types as well. We have only used OCaml's built-in data types, how can we construct our own data types?

<!-- more -->

## `type`

The `type` keyword is similar to the `typedef` keyword in C, except more limited in scope. A `type` can only be multiple variants of arbitrary values.

```ocaml
(* coin is enum with variants Heads and Tails*)
type coin = Heads | Tails
```

Each variant can also contain data of other data types.

```ocaml
type shape =
 | Rect of float * float
 | Circle of float
let r = Rect (3.0, 4.0) (* r has type shape *)
```

`shape` here has two variants. It can either be a tuple of two `float`s when it is a `Rect`, or it can be a single `float` when it is a `Circle`.

These data types are also known as *algebraic data types* or *tagged unions*.

## Option

ADTs can be useful when we want to ensure the complete handling of all cases. For example, if an object is nullable, it is useful to make sure that we must handle the null case instead of passing that off to the developer who might carelessly not handle it. This is where the **option** type comes from.

```ocaml
type 'a option =
 | Some of 'a
 | None
```

The `'a` keyword means that that the type `option` is polymorphic, and the variant `Some` will contain whatever type that `option` is defined for. When handling an `option`, you *must* destructure it into its `Some` and `None` variants and handle both cases, otherwise OCaml will warn you for a non-exhaustive pattern match.

## List

We can actually define our own list data type as a **recursive data type**, which is a data type which contains itself.

```ocaml
type 'a list =
 | Nil
 | Cons of 'a * 'a list
```

Here, `list` has two variants, `Nil` and a `Cons` tuple of an element and a `list`. If we think of the traditional list data type, this is actually just a more verbose version. `[]` is sugar for `Nil` and `::` is sugar for `Cons` tuple. 

```ocaml
let rec len l =
    match l with
    | Nil -> 0
    | Cons (_, t) -> 1 + (len t)
(* same as *)
let rec len l =
    match l with
    | [] -> 0
    | _ :: t -> 1 + (len t)
```

## Exceptions

**Exceptions** are a special data type used for errors in OCaml. Exceptions are similar to type constructors in that they can take arguments or have none.

```ocaml
exception Sign of int
let f n =
    if n > 0 then
        raise (Sign n)
    else
        raise (Failure "foo")
```

We can `raise` an exception with arguments whenever we want, which will exit the function with the exception name and its arguments. `Failure` is a generic exception type that is used with strings.

There is also special `try` syntax used to catch exceptions.

```ocaml
let g n =
    try
        f n
    with Sign n ->
            Printf.printf "Caught %d\n" n
          | Failure s ->
            Printf.printf "Caught %s\n" s
```

The function `g` will try running `f n`, but if that raises an exception, it will be caught in `with`. It can be pattern matched for different exception types.
