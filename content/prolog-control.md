+++
title = "Control in Prolog"

date = 2022-04-05



[taxonomies]

categories = ["Principles of Programming Languages"]

tags = ["cs352"]

+++

Conditional statements are very powerful and are in used in almost every language. How does Prolog implement the same idea?

<!-- more -->

## Logic + Control

The basic idea behind a control statement is that there is *logic*, such as facts, rules, and queries, which are composed of clauses/**goals**, as well as *control*, which is how Prolog chooses the logic among several options. The control is implemented by the order of the facts/goals, which is **extremely important**. Prolog will *always* choose the first applicable rule in a program, and it will *always* choose the leftmost clause/goal in a query.

Brevity is key. If there are two rules that do the same thing, or one rule that is implied by another rule, you should probably take it out unless executing something more than once is your explicit intention.

## Abstract Interpreter

**A substitution \\(σ\\) is a finite set of pairs of terms \\(\\{X_1/t_1, ..., X_n/t_n\\}\\) where each \\(t_i\\) is a term and each \\(X_i\\) is a variable such that \\(X_i ≠ t_i\\) and \\(X_i ≠ X_j\\) if \\(i ≠ j\\).**

An empty substitution is denoted by the letter \\(ε\\).

Some important rules:

- A variable cannot substitute itself e.g. \\(Z/Z\\) is illegal.

- Only a variable can be substituted e.g. \\(m/n\\) is illegal because \\(m\\) is an atom.

The meaning of applying a term to a substitution is that every occurrence of \\(X_i\\) in the compound term is replaced with the corresponding substituent in \\(σ\\) simultaneously. This application is known as **instantiation**, and that new compound term \\(Eσ\\) is an **instance**.

Bringing back the earlier rules about unification, you can see how unification is derived from substitution. Variables can unify with anything, so they are the term that is substituted in Prolog. There is an exception to this, which is known as the **occurs check**.

We can use a substitution \\(σ\\) as a **unifier** for two terms if the application of \\(σ\\) to those terms makes them *syntactically equal*. This is distinct from *semantic* equality. When we talk about unifiers, we are only talking about the actual letters, not the meaning of the term. For \\(S = f(X,Y)\\) and \\(T=f(g(Z),Z)\\), if we have a \\(σ = \\{X/g(Z),Y/Z\\}\\) then \\(S\\) and \\(T\\) will be unified. Since the unification only cares about syntax, more than one unifier may exist for two terms. We could have just as well substituted the other way and it would still be a unifier.

## Unifier Computation

```
unify(X, Y, θ) =
  X = Xθ
  Y = Yθ
  case X is a variable that does not occur in Y:
    return (θ{X/Y} ∪ {X/Y}) // this replaces X with Y in the unifier, and then adds a new substitution just in case X will appear later
  case Y is a variable that does not occur in X:
    return (θ{Y/X} ∪ {Y/X}) // this replaces Y with X in the unifier. this is a rare case when the variables substitute each other
  case X and Y are identical constants
    return Θ
  case X and Y are compound terms like f(X1, ..., Xn) and f(Y1, ..., Yn)
    return (fold_left (fun Θ (X,Y) -> unify(X, Y, Θ)) θ [(X1, Y1), ..., (Xn, Yn)]
    // this applies the unification process to the compound terms
    // the function is just folding left with θ as acc and the set of compound terms as the list
```

This is the basic pseudocode for the unify function. It's recursive, and side-effect free. However, it's not complete for writing an interpreter. For that, we also need backtracking.

## Backtracking

The resolvent will maintain a list to satisfy our query. When we try to resolve a compound term, any goals within will be queued onto the resolvent to be resolved. This is *non-deterministic*, there is no order that will necessarily be followed other than that sub-goals within a single term will be inorder.

## Missionaries and Cannibals

Let's imagine a problem where we have three missionaries and three cannibals that need to cross a river in one boat. If the cannibals outnumber the missionaries, they will get eaten. How can we get every person across the river?

The concept of a *safe state* will be important here. A state is safe when no missionaries are eaten. By labelling certain states as unsafe, we can cut those paths out of our search algorithm. We should also define *transitions* between states, where a predicate moves a state from A to B. 

Our states are essentially a pair representing position.

```pro
start(3-3-0-0-l).
finish(0-0-3-3-_).
```

The elements in order are missionaries on the original side, cannibals on the original side, missionaries on the target side, cannibals on the other side, and the location of the boat.

The safety of the state is based on whether there are more cannibals than missionaries.

```pro
safe(0-_-M2-C2-_) :- M2 >= C2.
safe(M1-C1-0-_-_) :- M1 >= C1.
safe(M1-C1-M2-C2-_) :- M1 >= C1, M2 >= C2.
```

Every state here represents a point at which there are at least as many missionaries as cannibals on each side of the river.

The change in state can be caused by a `carry/2` predicate which details how many missionaries and cannibals are being moved, respectively. In our case, the boat can only move up to 2 people so we need to represent every possible case of that.

```pro
carry(2, 0).
carry(1, 1).
carry(0, 2).
carry(1, 0).
carry(0, 1).
```

However, we need a way to represent moving on both sides of the river, so we will have two transitions.

```prolog
step(M1-C1-M2-C2-l, M3-C3-M4-C4-r) :-
    carry(X, Y),
    M1 >= X, M3 is M1 - X, M4 is M2 + X,
    C1 >= Y, C3 is C1 - Y, C4 is C2 + Y.

step(M1-C1-M2-C2-r, M3-C3-M4-C4-l) :-
    carry(X, Y),
    M2 >= X, M4 is M2 - X, M3 is M1 + X,
    C2 >= Y, C4 is C2 - Y, C3 is C1 + Y.
```

This `step` predicate checks every valid `carry`, then will execute that transition from the current state.

```pro
travel(A, A, _, []).
travel(A, C, Visited, [B | Steps]) :-
```
