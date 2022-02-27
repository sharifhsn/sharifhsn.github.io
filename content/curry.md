+++
title = "Currying Arguments in a Function"
date = 2022-02-21

[taxonomies]
categories = ["Principles of Programming Languages"]
tags = ["cs314"]
+++
**Currying** is the concept of having multiple arguments in a function. OCaml defaults to currying its functions. `int -> int -> int` is a function that takes two `int`s and returns an `int`.

<!-- more -->

The `->` is *right-associated* and the function application is *left-associated*. The last element of the function definition is always the return type, but calling a function always counts arguments from the left.

```ocaml
let f a b = a / b;;
let f = fun a -> (fun b -> a / b);;
```

These two lines are equivalent because the second line is just the uncurried function separated into two parts. The `fun` lambda only has one argument, and the `->` keyword is right-associated so `b` is considered part of the arguments.

Currying allows you to pass only a portion of the expected arguments to the function, the same way that Python uses keyword arguments.

Another way to enable multiple arguments is by using a tuple that contains both arguments, which are destructured in the function definition. However, the advantage of currying is that you can separate the call of the function from the arguments.

```ocaml
let add a b = a + b;;
let addthree = add 3;;
addthree 4;; (* evaluates to 7 *)
```

This code allows `addthree` to exist as an implementation of `add` with a specific argument already given.

However, it's not all roses with currying. Function need to retain state regardless of stack state, so the local variable `3` that is temporarily in the function `addthree` may not always be there. Anonymous functions may not have the same call stack.

In C-like languages, local variables are contained within their own stack frames. When a function calls another function, the new stack frame that is created contains its own local variables. If a variable is declared, then initialized through a function call, that variable contains junk until the function returns.

If we return a function that references a local variable in another function, reading it off the stack can get confusing. The first variable you look for is still uninitialized, so you need to evaluate where that variable is coming from to understand.

We solve this problem using **static scoping**. Nonlocal names refer to their nearest binding in the program text. This is also known as lexical scoping. If two variables have the same name in an inner scope and an outer scope, then we read the one in the inner scope first.
