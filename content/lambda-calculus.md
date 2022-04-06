+++
title = "Lambda Calculus"

date = 2022-03-03



[taxonomies]

categories = ["Principles of Programming Languages"]

tags = ["cs314"]

+++

When we discuss the principles of programming languages, **lambda calculus** is the bedrock of our discussion. It is the most abstract, formal way to describe functions and application.

<!-- more -->

## Turing Completeness

A programming language is said to be **Turing complete** if it can compute any function also computable on a Turing machine. It must either be able to emulate a Turing machine, or it must be able to emulate a Turing complete language.

Although this is a powerful definition, in truth it is a simple one to satisfy. Features like loops and currying are not related to Turing completeness.

## Syntax

Lambda calculus is a very simple language which only uses functions and applications, but is still Turing complete.

An expression in lambda calculus is defined as

```bnf
e ::= x    # variable
    | λx.e # abstraction (function definition)
    | e e  # application (function call)
```

Everything is a function in lambda calculus. We can make a simple interpreter in OCaml as with Imp:

```ocaml
type exp =
    | Var of string
    | Lam of string * exp
    | App of exp * exp
```

in the same order that was defined earlier. `Lam` here is similar to `Fun` for Imp. The lambda calculus expression

```bnf
(λx.λy.x y) λx.x x
```

can be deconstructed to its AST as

```ocaml
App(
    Lam("x",
        Lam("y",
            App(Var "x", Var "y")
        )
    ),
    Lam("x",
        App(Var "x", Var "y")
    )
)
```

Let's parse this out forwards. First we read that there is a parenthesis, which means that there is an expression enclosed within. There is a \\(λ\\), which means that the expression is a lambda. The next letter is `x`, which is the name of the `Lam`. The period following separates the function definition to the containing expression, which continues to another \\(λ\\). That means that the containing expression is also a `Lam`, which is called "y". The containing expression is now an `App` which is composed of two `Var`s. This is the end of the nesting because we hit the end parenthesis. The same logic applies for the second expression but with less nesting. Since we have two expressions at the top-level, the full expression is an `App`.

The scope of \\(λ\\) extends as far to the right as possible, excepting parentheses. This is why we needed the parentheses for the first term in the above statement, otherwise the first \\(λ\\) would extend throughout the entire statement. The application, however, is left-associative, like OCaml. 

## Beta Reduction

A function call of type `(λx.e1) e2` replaces all instances of `x` in `e1` with `e2`. That means that we can substitute this statement with `e1{e2/x}`. This is called **beta reduction**. All we have done is apply the function and replace the formal parameters through substitutions. Beta reductions should always be idempotent for the statement. When no more beta reductions can be performed on a term, then it is said to be in *beta normal form*, for example `λx.e`.

Another example will be instructive here. Take the term `(λx.λz.x z) y`. This is a function application, since it has two terms. It follows the form that we stated earlier, so we can substitute all instances of `y` on the outside \\(λ\\). This would give us the final term `λz.(y z)` eliminating the outside `λx` and replacing the `x` in the inner term with `y`.

## Alpha Conversion

Lambda calculus is **statically scoped** which means that variable definitions are only scoped locally. That means that within a function, you can rename *bound* variables with the same meaning. This is called **alpha conversion**.

An important distinction here is between a free and bound variable. Free variables are not contained within a lambda, while bound variables are. For example, in `(λy.λz.y z x)`, `y` and `z` are bound to `λz` and `λy`, respectively, while `x` is free. `λy` contains the `App` of `λz` and `z`.
