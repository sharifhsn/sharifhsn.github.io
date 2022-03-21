+++

title = "Lets, Tuples, and Records in OCaml"

date = 2022-02-03



[taxonomies]

categories = ["Principles of Programming Languages"]

tags = ["cs314"]

+++

Lists are the most basic data structure in OCaml, but there are others that we need to be aware of.

<!-- more -->

## Let Expressions

We have seen the keyword `let` used to define expressions and store values. However, the same keyword can be used to create expressions which bind variables in other expressions. The `let` *statements* we used before do not evaluate to any value, while **let expressions** do evaluate.

```ocaml
let x = 5 in x * 3
```

The `in` keyword gives us a clue on what's going on. You can think of it almost like a function where we replace variables in the inner expression with the values in the outer expression. The expression above evaluates to 15, because it is `x * 3` *in* which `x` is 5.

We can type check this expression where `x` has the same type of the binding expression.

If you omit `in`, you can think of that as a `let` expression which is bound in the global scope instead of the scope of the body expression.

```ocaml
let x = 37;;
```

In this statement `x` is defined as 37 in the global scope and it can be used elsewhere.

I've used the word *scope* a lot here, so let's define that a little more concretely. In the above `let` expression, the variable `x` is not visible in any other part of the program. Let's imagine that we have both of these lines together, the `let` expression and the `let` statement. They're both named `x`, so what would the value be?

We can imagine evaluating expressions right to left, and upon encountering a variable, act like it is a pointer to an expression in the outer scope. In the innermost scope, the expression is `x * 3`. This expression has no meaning because `x` is a variable, so we back up one scope and check if `x` is defined. And in fact, it is! So we will replace the `x` in that expression with 5. Note that even though `x` is defined as 37 in the global scope, the inner scope **shadows** the global scope.

Shadowing refers to when a variable name is rebound in an inner scope to have a different meaning. Some languages, such as Java, do not allow you to do this because of possible confusion. However, it is sometimes useful to use the same name for different things, so languages like C and OCaml permit shadowing.

You can also use `let` expressions inside a function, and this is often good style because it clarifies constants:

```ocaml
let area d =
    let pi = 3.14 in
    let r = d /. 2.0 in
    pi *. r *. r
```

Much better than C `#define`, right?

OCaml does not permit you to mutate variables. However, you can simulate this by shadowing a variable with a new value:

```ocaml
let x = 0;;
x = x + 1;; (* not allowed! *)
let x = x + 1; (* allowed, but discouraged *)
```

This is kind of an ugly hack so you should avoid it in real code, though it is technically possible under OCaml's rules.

We can nest `let` expressions, but this is generally bad practice like shadowing. Realistically, it's usually better to just write linear expressions.

`let` expressions don't just have to use a plain variable. We can also use patterns to bind expressions, and if the binding expressions fails to match the pattern then we have an exception. This is useful when we want to extract a particular value from an expression.

```ocaml
let [x] = [1] in 1 :: x (*  evaluates to [1; 1*)
```

## Tuples

Tuples represent collections, like lists, but they contain a fixed amount of values. The tradeoff is that they can be *heterogenous*, which means they can have multiple types. The type of a tuple is the type of each of its component, separated by asterisks.

```ocaml
(1, 2) (* int * int *)
(1, "string", 3.5) (* int * string * float *)
```

Because each tuple has a distinct type, a list of tuples can only have one type of tuples in it.

Tuples lend themselves particularly well to pattern matching. Instead of having multiple function arguments like is typical in OCaml, we can have one argument that is a tuple and then pattern match it in order to destructure it. This is also a convenient way to return multiple variables from a function, which is otherwise not allowed.

## Records

Each element of a tuple is referenced by its position. Sometimes, we want to reference elements by name, like in a dictionary. For this use, we use **records**. Records are a distinct type that must be pre-defined before being used.

```ocaml
type date = { month: string; day: int; year: int }
```

Now, we can construct records by using the same brace notation but giving each name a value.

```ocaml
let today = { day=3; year=2022; month="f"^"eb" };;
```

You might notice


