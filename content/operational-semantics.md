+++
title = "Operational Semantics"

date = 2022-02-17



[taxonomies]

categories = ["Principles of Programming Languages"]

tags = ["cs314"]

+++

What are the formal semantics of how a programming language works, mathematically? That's a broad topic. **Operational semantics**, which are how programs *execute*, are narrow enough for one article.

<!-- more -->

## Rules

The basis behind the mathematics of operation are based on using **rules** to define a **judgment**. The expression `e` will always evaluate to the value `v`. We can construct a micro-OCaml through the datatypes `exp` and `value`:

```ocaml
eval: exp -> value
```

This way of presenting semantics is called a **definitional interpreter**. `eval` means `exp -> value`, and we use interpretations to define the language's meaning.

## Grammar

It is useful to define **meta-variables** which represent categories of syntax. We will list a few here:

- `x`: any identifier or variable name

- `n`: a numeral value

- `e`: any expression

- `::=`: meta-syntax for definition

- `|`: meta-syntax for variants

```bnf
e ::= x | n | e + e | let x = e in e
```

This is an important line which defines what an expression *exactly* is. Up until now, we have used the term expression quite loosely, so it's good to have an exact definition in operational semantics. An expression is either an identifier, a numeral or a combination of expressions. The last variant may seem a little strange to you, but remember, `let` expressions are simply replacing identifiers in an expression, so they are also expressions. This is a powerful definition because it fully encompasses the **abstract syntax tree (AST)**.


