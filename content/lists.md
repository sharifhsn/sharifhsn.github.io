# Lists

Lists are the most basic data structure in OCaml. The most analoguous structure is a vector in C++ or Rust. Lists are *homogenous* and of *arbitrary length*.

The most basic list is **nil**, the empty list: `[]`. We can prepend elements to a list through the **cons** operator: `::`. Every display of a list is actually just syntactic sugar for every element being cons with the empty list:

```ocaml
[1, 2, 3] = 1 :: 2 :: 3 :: []
```

Importantly, lists are immutable! When they are created, they cannot be changed. If you want to add an element to a list, you must construct a new list based on the old one.

> This might seem inefficient, to create a new list every time we want to change it. But the semantics of a language do not necessarily correspond to its compiled execution. In particular, the invariant that lists are immutable can lead to optimizations where a compiler doesn't need to keep track of mutated information.
