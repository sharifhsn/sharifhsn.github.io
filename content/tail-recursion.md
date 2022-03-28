+++
title = "Tail Recursion in OCaml"

date = 2022-02-15



[taxonomies]

categories = ["Principles of Programming Languages"]

tags = ["cs314"]

+++

Recursion in many languages can cause significant overhead. It might seem that the excess amount of recursion in OCaml would decrease its performance. But it actually doesn't, and that's thanks to **tail recursion**.

<!-- more -->

## Implementing Reverse

In a functional language with a linked list, it's not a trivial task to reverse a list. There are multiple ways to do it, and they have different performance implications.

The most basic implementation would just be to concatenate backwards.

```ocaml
let rec rev l =
    match l with
    | [] -> []
    | x :: xs -> (rev xs) @ [x]
```

Unfortunately, there's a problem with this. Every time we call `rev` recursively, we must initialize a new stack frame for every recursive call, leaving the value `x` in `[x]` in the calling stack.

```ocaml
rev [1; 2; 3]
 → (rev [2; 3]) @ [1]
 → ((rev [3]) @ [2]) @ [1]
 → (((rev []) @ [3]) @ [2]) @ [1]
 → (([] @ [3]) @ [2]) @ [1]
 → ([3] @ [2]) @ [1]
 → [3; 2] @ [1]
 → [3; 2; 1]
```

As you can see, there are a total of three stack frames for a list with three elements. This is pretty bad. However, we can rewrite this function to use tail recursion.

```ocaml
let rec rev_helper l acc =
    match l with
    | [] -> acc
    | x :: xs -> rev_helper xs (x :: acc)
let rev l = rev_helper l []
```

There doesn't need to be any stack frame here because there are no local variables. The only thing that's being returned is the function call to `rev_helper`, so the stack can simply change to the `rev_helper` call without saving the previous stack frame. This tail recursion is incredibly powerful.

I'll repeat it to be clear. Tail recursion works when the return value of a recursive function is *only* the recursive function call. The reason that the previous `rev` function didn't work is because the return value was `(rev xs) @ [x]`, so `x` must be saved in a stack frame in order to remember it.

However, you might have noticed that we had to include a new variable called `acc`. This accumulator variable is a common pattern for when we want to have tail recursion since the `acc` is located inside the function call as an argument.

The power of tail recursion can not be understated here. In a typical function, having excessive stack frames can easily cause a stack overflow for a large data set. With tail recursion, we can have both the lack of side effects associated with recursion and the performance associated with iteration.

## General Tail Recursion Pattern

```ocaml
let f x =
    let rec aux arg acc =
        if (* base case *) then acc
        else
            let arg' = (* next argument *)
            let acc' = (* updated accumulator *)
            aux arg' acc' in
    aux x (* initial value of accumulator e.g. 0, []*)
```
