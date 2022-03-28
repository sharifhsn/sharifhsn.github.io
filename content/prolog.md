+++
title = "Prolog"

date = 2022-03-24



[taxonomies]

categories = ["Principles of Programming Languages"]

tags = ["cs314"]

+++

**Prolog** is a special programming language that is not like many others. It exemplifies the paradigm of **logic programming**.

<!-- more -->

## Terms

Functional programming is based on the idea that all operations exist using functions that give the same output for a given input. Imperative programming is based on the idea that we can manipulate variables and order them around like soldiers. Logic programming takes propositional logic from discrete structures and turns it into a language.

Every program is made up of **terms**, which are **constants**, **variables**, and **compound terms**. A constant is any word that is lowercase, whether it be a number, a string, or just a word. A variable is a word that begins with a cpapital letter.

Compound terms are relations. Relations in the logical sense are a subset of the Cartesian product of two sets, where the result set consists of elements that are contained in both. The top level name is the **functor**, and the number of arguments is the **arity**. For example, a compound term might be `male(robb)` where `male` is the functor with an arity of 1. **The order of arguments within the compound term matters!**

**Facts** are a knowledge base for a particular program. Each fact is a compound term showing a relation, followed by a period. **Rules** are a generality on facts where they are facts *given* assumptions. A fact is just a rule that is true without assumptions. **The order of facts/rules is very important to the execution of code!** Prolog reads from top to bottom, so the most important facts/rules should be placed near the beginning.

```prolog
father(rickard, ned).
father(rickard, brandon).
father(rickard, lyanna).
father(ned, robb).
father(ned, sansa).
father(ned, arya).
```

## Queries

Once we have our knowledge base, we can **query** for information.

```prolog
?- father(ned, sansa).
```

This query will return true because it is in our list of facts. However, what happens if we query something it doesn't know?

```prolog
?- father(ned, bran).
```

Prolog operates on the **closed world assumption**, which means that it only knows what it's been told. If it doesn't know something, it will assume falsity. So Prolog will say the Ned is *not* Bran's father, despite it having no idea whether that is the case.

There are also **existential queries**. Instead of asking a boolean question, it asks if a fact exists.

```prolog
?- father(ned, X).
```

This query asks for all the relations `father` where `ned` is the first argument. It will return

```prolog
X = robb ;
X = sansa ;
X = arya .
```

in the order that the facts are laid out. The semicolon is the logical OR in Prolog, so it's saying that either one of the results is true. It would return false if there are no facts that satisfies the query.

## Rules

Rules in general can be any logical statements from facts that are inferred inductively. The general form is

```prolog
H :- B1, B2, B3, ..., BN
```

means that the head `H` is true if `B1` ∧ `B2` ∧ `B3` ... etc.

```prolog
parent(X, Y) :- father(X, Y).
```

means that the relation `parent` exists between `X` and `Y` as long as the relation `father` exists between `X` and `Y`. We can construct rules recursively as well:

```prolog
ancestor(X, Y) :- parent(X, Y)
ancestor(X, Y) :- parent(X, Z), ancestor(Z, Y)
```

This defines ancestor in two ways. The first rule says that `X` is an ancestor of `Y` as long as `X` is a parent of `Y`. The second rule introduces an additional definition where `X` is an ancestor of `Y` as long as `X` is a parent of some `Z` which is an ancestor of `Y`.

We can almost think of this like a list, where parent means adjacent and ancestor means inorder. Of course two elements are inorder if they are adjacent, that's the first rule. However, if we can imagine a long list, two elements are inorder if the adjacent element to `X` is inorder previous to `Y`! The logic applies in any general situation; this is the advantage of the logical paradigm.

## Unification

The core of how Prolog will compute answers is through **unification**, which is similar to pattern matching. It has three unification conditions:

- identical constants will be unified

- variables will always unify

- compound terms will unify if functor and arity match as well as their arguments, recursively (using the top two rules)

## Lists

Data structures seem pretty incongruent with our idea of logical programming so far. Lists are defined similarly to OCaml as recursive data structures.

```prolog
[1, 2, 3, 4] % list
[] % nil
[H | T] % head cons tail
```

To find the last element of a list, we can ues this rule

```prolog
last([H], H).
last([_ | T], V) :- last(T, V).
```

The first rule is the base case, where if there is only one element in a list, that element is the "last element". Then, the second rule is recursively defined. If the first argument 

## Arithmetic

The arithmetic operators are built into Prolog, and are built as relations between terms. Arithmetic equality is *not* the same thing as unification. If you want to check for arithmetic equality, you must use `is`.


