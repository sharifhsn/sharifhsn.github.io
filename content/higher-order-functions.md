+++
title = "Higher Order Functions in OCaml"

date = 2022-02-08

[taxonomies]

categories = ["Principles of Programming Languages"]

tags = ["cs314"]

+++

We can do a lot of cool things with functions besides calling them in OCaml.

<!-- more -->

## Anonymous Functions

Values are a subset of expressions, as previously stated. All expressions can evaluate to values, but values are final.

**Anonymous functions** are also values. Sometimes, it's more convenient not to create and name a whole new function for our purpose. Anonymous functions are ad hoc functions that exist as values in expressions. They are expressed using the keyword `fun`.

```ocaml
let y = fun x -> x + 3
```

This might not seem to have much benefit compared to a full function definition, but it is very useful within `let` expressions. Since anonymous functions are values, not just expressions, they can be manipulated far more powerfully than even general expressions.

```ocaml
let y = (fun x -> x + 1) 2 in
(fun z -> z - 2) y
```

This code might seem a little hard to parse, but it's easier to think about if we rewrite it to use traditional function definitions.

```ocaml
let f x =
    x + 1
let g z =
    z - 2
let y = f 2 in
g y
```

Now we can tell that `y` is 3 in the function `g`, which then evaluates to 1. However, the former code snippet is a much terser way to write this expression  if we don't need the functions `f` and `g` anymore.

One good way to think about it is that anonymous functions are to regular functions as literals are to variables. If we only need to use the value `"really_long_string"` once, we don't need to store it in a variable. On the other hand, it can be useful to store that literal in a variable `s` that is much shorter to write. Similarly, if we only need to use the function `x -> x + 1` once, we don't need to store it in a function variable.

In fact, this isn't even an analogy. Functions are first-class in OCaml, so regular functions are just variables that store anonymous functions:

```ocaml
let f x = body
(* this is sugar for this *)
let f = fun x -> body
```

And in the same vein, we can name functions within `let` expressions in an anonymous ways.

```ocaml
let move l x =
    let left x = x - 1 in
    let right x = x + 1 in
    if l then left x
    else      right x
;;
(* same as *)
let move' l x =
    if l then (fun y -> y - 1) x
    else      (fun y -> y + 1) x
```

Note also that the local variable in the anonymous function doesn't actually matter to the expression it's used in; this is a consequence of the shadowing rules of OCaml.

There are several functions in the standard library of OCaml that use higher order functions.

## Map

`map` is a function in the `List` module of OCaml. Like the name implies, this function maps a function onto every element of a list and returns that list. It has type `('a -> 'b) -> 'a list -> 'b list`.

```ocaml
let rec map f l =
    match l with
    | [] -> []
    | h :: t -> (f h) :: (map f t)
```

This is a simple, yet powerful and useful function. That's why it is included in the `List` module, although it's trivial to write yourself.

## Fold

`fold` is another function in the `List` module in OCaml, that iterates over a list. The essential idea is that you have an accumulator variable that you want to get based on the values in a list. 

```ocaml
let rec fold f acc l =
    match l with
    | [] -> acc
    | h :: t -> fold f (f acc h) t
```

For example, this is a way to implement a sum function using `fold`.

```ocaml
let rec sum acc l =
    match l with
    | [] -> acc
    | h :: t -> sum (acc + h) t
sum 0 [2; 5; 100; 53];;
(* same as *)
fold (fun acc x -> acc + x) 0 [2; 5; 100; 53];;
```

Its type is `('a -> 'b -> 'a) -> 'a -> 'b list -> 'a`. We can deconstruct that and understand each part of the function. The initial function `f` applies the type `'b` to `'a` and returns `'a`. Then for the next two arguments, we keep the same `'a` accumulator and iterate over the `'b list`.

The use of an accumulator makes `fold` very versatile, since you can put anything in there.

We can combine `map` and `fold` to create the *map/reduce* framework which can be massively parallelized. We first map a function over our list, then we reduce the list into a single accumulator value.

There is also an alternative version of `fold` called `fold_right` that works in reverse, which can be better for certain problems. However, it comes with steep performance cost: every recursive call builds a new stack frame. The original `fold` is able to optimize this call away by using tail recursion.
