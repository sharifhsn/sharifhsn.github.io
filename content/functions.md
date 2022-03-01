+++
title = "Functions in OCaml"

date = 2022-01-25


[taxonomies]
categories = ["Principles of Programming Languages"]
tags = ["cs314"]
+++

A **function** takes arguments, performs operations using them, and returns a value. In OCaml, we can write functions like so

<!-- more -->

```ocaml
let rec fact n = (* the function is named fact and its arg is n *)
    if n = 0 then 1
    else n * fact (n - 1) ;;
```

This is a *recursive* function, so we must prepend the keyword `rec` to the name of the function. The reason for this is that the function name is not automatically in scope, so if we don't use that we won't necessarily know what `fact` refers to. `rec` adds the function name into scope.

As you can see, the declaration of a function is fairly similar to the declaration of a variable. In fact, we can use functions as variables in OCaml! This is because functions in OCaml are *first-class functions*.

You also might have noticed here that we haven't mentioned the type of anything. This is because the type of the function can be inferred, also known as **type inference**. This process is a normal part of type checking for correctness. The compiler itself will look for a type that the code is correct for. In this example, `n` is compared with an integer, so it must only be an `int`. The return type is `n` multiplied by its own return type, so that must also be `int`. The compiler can see without us telling it that the function is of type `int -> int`.

But what is that `int -> int` that I just wrote? That is the constructor for a function type. The last type indicates the return type, and all other types are the arguments in order. The function type `float -> int -> float`, for example, takes a `float` and `int` as its arguments and returns a `float`. Try looking at different OCaml functions and guessing its type as a fun exercise! :smile:

> Aside: you might question why we couldn't just get a float as a result from the multiplication. The standard operands in OCaml only apply to `int`, and applying them to other types is an error. In order to add `float`, we must use `+.`, not `+`.

Type checking works like a proof by contradiction. We assume that a function has some type, then check if there are any logical contradictions created by this assumption. If there are, then we reject this type.

## Calling a Function

Function calls (also known as *applications*) look very similar to function declarations; they are simply the name of the function followed by its arguments, no parentheses necessary. The type checking involved is that we check if every expression supplied as an argument corresponds to the same type as it's supposed to. We evaluate each expression from right to left to see if they're correct or not (though this doesn't matter unless you have side effects).

Once we find the value for each argument, we replace each argument in the function with its value, which creates an expression that can be evaluated to the final value.

> Aside: Although we have type inference, we can use *type annotations* with OCaml. These increase the readability of code by making the type of the variable obvious: `let (x : int) = 3`
