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

Every program is made up of **terms**, which are **constants**, **variables**, and **compound terms**. A constant is any word that is lowercase, whether it be a number, a string, or just a word. A variable is a word that begins with a capital letter.

Compound terms are relations. Relations in the logical sense are a subset of the Cartesian product of two sets, where the result set consists of elements that are contained in both. The top level name is the **functor**, and the number of arguments is the **arity**. For example, a compound term might be `male(robb)` where `male` is the functor with an arity of 1. **The order of arguments within the compound term matters!**

**Facts** are a knowledge base for a particular program. Each fact is a compound term showing a relation, followed by a period. **Rules** are a generality on facts where they are facts *given* assumptions. A fact is just a rule that is true without assumptions. **The order of facts/rules is very important to the execution of code!** Prolog reads from top to bottom, so the most important facts/rules should be placed near the beginning.

```pro
father(rickard, ned).
father(rickard, brandon).
father(rickard, lyanna).
father(ned, robb).
father(ned, sansa).
father(ned, arya).
```

## Queries

Once we have our knowledge base, we can **query** for information.

```pro
?- father(ned, sansa).
```

This query will return true because it is in our list of facts. However, what happens if we query something it doesn't know?

```pro
?- father(ned, bran).
```

Prolog operates on the **closed world assumption**, which means that it only knows what it's been told. If it doesn't know something, it will assume falsity. So Prolog will say the Ned is *not* Bran's father, despite it having no idea whether that is the case.

There are also **existential queries**. Instead of asking a boolean question, it asks if a fact exists.

```pro
?- father(ned, X).
```

This query asks for all the relations `father` where `ned` is the first argument. It will return

```pro
X = robb ;
X = sansa ;
X = arya .
```

in the order that the facts are laid out. The semicolon is the logical OR in Prolog, so it's saying that either one of the results is true. It would return false if there are no facts that satisfies the query.

## Rules

Rules in general can be any logical statements from facts that are inferred inductively. The general form is

```pro
H :- B1, B2, B3, ..., BN
```

means that the head `H` is true if `B1` ∧ `B2` ∧ `B3` ... etc.

```pro
parent(X, Y) :- father(X, Y).
```

means that the relation `parent` exists between `X` and `Y` as long as the relation `father` exists between `X` and `Y`. We can construct rules recursively as well:

```pro
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

```pro
[1, 2, 3, 4] % list
[] % nil
[H | T] % head cons tail
```

To find the last element of a list, we can use this rule

```pro
last([H], H).
last([_ | T], V) :- last(T, V).
```

The first rule is the base case, where if there is only one element in a list, that element is the "last element". Then, the second rule is recursively defined. If the first argument 

## Arithmetic

The arithmetic operators are built into Prolog, and are built as relations between terms. Arithmetic equality is *not* the same thing as unification. If you want to check for arithmetic equality, you must use `is`.

The `is` operator evaluates the right hand expression and unifies the expression with the left. There can be variables present, but the variables need to be defined as some ground value. You can't create an algebraic equation using `is`.

Basic arithmetic operators are also built in, but only operate on numerals. If you try to apply, for instance, `+`, to lowercase word constants, there will be an error.

## Backtracking

Prolog computes the length of a list very similar to OCaml. An example OCaml length function might be:

```ocaml
let rec len l =
    match l with
    | [] -> 0
    | h :: t -> len t + 1
```

The Prolog rules are similar:

```pro
len([], 0).
len([H | T], N) :- len(T, M), N is M + 1.
```

The relation `len` with arity 2 defined on the empty list maps to 0, obviously. The relation `len` with arity 2 and the first argument being a list with head and tail is the actual recursive part we are interested in.

You might raise an eyebrow at the use of `is` in the second rule; after all, didn't I just say that the right hand side must have ground terms? We don't know what `M` is!

Actually, we do know. You can think of the order of clauses as the imperative definition of variables. The variable `M` is declared and initialized with the first `len` clause, and it can then be used in the second clause. If we put the second clause first, Prolog would raise an error.

You might notice a pattern here. The function is almost identical to the OCaml function, except for one difference. In the functional paradigm, every function evaluates to an expression which is returned within the function, and those are the values that are used for every operation. In the logical paradigm, there is no implicit return. `len(T, M)` is the same thing as `M = len(T)`!

The length is computed using **backtracking**. It will try every possible rule for `len` and then evaluate every single one. You can think of a depth first search of the tree of rules.

Every point at which there are multiple branches is called a **choice point**. In the `len` example, it can choose to either check the fact for `[]` or the recursive rule.

Backtracking can cause unexpected results. Say we try to query

```pro
?- len(A, 2)
```

We're querying for a list of size 2 which *doesn't exist*. What would happen?

Well, Prolog will try to find a matching branch to an existing rule. It will fail the first rule most of the time, so it will match the second rule. Because `M = N - 1`, when trying to find `M` it will subtract from the second argument and backtrack. It will eventually match to return the result `A = [h1, h2];` when it matches 0 as the length and backtracks to adding arbitrary variables to the list of T.

But then something unexpected happens. After printing that last result, Prolog would spin on and on, never returning again. That's because although we have matched the first rule and returned a list, Prolog will try every rule to see if it matches. And in fact, the second rule does match `[T2, 0]` because a variable will match any value. Prolog will endlessly backtrack because a choice will always be available that can match the new recursion. This is much different from OCaml which will only execute the first match arm that matches. You need to be careful in Prolog in order to prevent this kind of result.

## Tail Recursion

Our `len` function is not tail recursive, since it adds on to the recursive call. In order to optimize our code, including an accumulator will aid because of tail recursion which we have discussed before.

```pro
len2([], Acc, Acc).
len2([H | T], Acc, N) :- M is Acc + 1, len2(T, M, N).
```

What we've done here is shift the `is` clause before the recursive call, which means we don't need to hold on to that choice when recursively calling `len2`, since it's guaranteed to already be checked.

You might ask, why do we even need to have that `is` clause? We might as well directly call `len2(T, Acc + 1, N)`. However, there's an issue here. The `len2` relation only *unifies* terms, which is a powerful operation, but it *does not evaluate them*. For a list `[1, 5, 7, 3]` it would show `N = 0 + 1 + 1 + 1 + 1`, not `N = 4` which would be correct. In order to make this correct, we need to evaluate the addition before making the recursive call into the second clause because that clause does not perform any evaluation.

This means that when we call `len2` we need to do `len2([], 0, X)`. This is a little annoying; we always know that our accumulator will begin at 0, why do we need to include it? There is a way to elide this definition:

```pro
len2([)
```

## Append

Another instructive example is the simple list append.

```ocaml
let rec append p q:
    match p with
    | [] -> q
    | [h :: t] -> h :: (append t q)
```

```pro
append([], Q, Q).
append([H | P], Q, [H | R]) :- append(P, Q, R).
```

## Prefix/Suffix

For a given list, we can place a separator in the list. Every element prior to the separator is the **prefix** and every element after is the **suffix**. The separator can be anywhere, including before or after every element.

We can get all possible prefixes/suffixes for a list for all separators:

```pro
?- prefix(X, [1, 2, 3]).
```

```bash
X = [];
X = [1];
X = [1, 2];
X = [1, 2, 3].
```

Let's think about how to build this out logically. We need to find all lists which are possible prefixes of this list. That means we don't want to capture, for example, `[2, 3]`, because that is not a possible prefix. We have to start from the left and include from there.

```pro
prefix([], []).
```

The empty list should map onto the empty list, naturally

## Generate and Test

We need a general approach to solve problems from a Prolog standpoint. The paradigm is very different from imperative or functional, so we can't just look at it the way that that OCaml or Java does.

Let's make `take`, which will remove exactly one element `x` from a list.

```pro
take([H | T], H, T).
take([H | T], R, [H | S]) :- take(T, R, S).
```

The first rule is the most obvious. If the element is at the head of the list, we can just return the tail of the list. This is our base case.

The second rule maps onto the case in the middle of the list. To imagine this, let's use an example

```pro
?- take([1, 2, 3], 2, T).
```

We try to unify this to the first rule, but it fails because 2 is not at the head of the first list. The second rule will unify because the first element is a list with multiple values, the second is a variable that will unify anything, and the third element is a variable that will unify anything. Upon match, it will query `take([2, 3], 2, S)`. This query will unify with the first rule, which will replace `S` with the value `[ 3 ]`. Once that `S` is unified, the rule will backtrack to the initial query which asked for `[H | S]`. Well, `H` in that initial case was `1` so our final return `T` will be `[1, 3]`.

By extending `take`, we can implement other functions like permutations.

```pro
perm([], []).
perm(L, [H | T]) :- take(L, H, R), perm(R, T).
```

Let's permute `[1, 2, 3]` once again.

```ocaml
?- perm([1, 2, 3], X).
```

It will not unify with the first rule, but it will with the second, obviously. The second clause will permute the list with the head taken away. Now, you might notice that `H` is not an explicit value here as when we used it earlier. Prolog seeing this will try to unify it with *every* value in the list, so `take` will be queried three times, with each value in the list being used in `H`. The resulting `R` from each of those queries is permuted again. The `H` used in `take` will be the head that is cons the backtracked form.

`[1, 2, 3]` will become `take([1, 2, 3], H, R)`. Prolog will run through every choice, starting with `take([ 1,[1, 2, 3] R)` which will unify `R` with `[2, 3]`. Now, the `perm` clause will be `perm([2, 3], T)`. Let's assume that this gives all valid permutations of `[2, 3]` i.e. `[2, 3]` and `[3, 2]`. Well, the resulting match on the original `X` would be `H | [2, 3]` and `H | [3, 2]` or for this particular unification, `[1, 2, 3]` and `[1, 3, 2]`. This conclusively shows the inductive step, and the base case is trivially the empty list.

As you might be able to tell from these examples, there is a general strategy to solving problems in Prolog.

1. **Generate a solution.**

2. **Test if it is valid.**

3. **If not valid, backtrack and try another solution.**

## Quicksort

One way to sort a list in Prolog is to use the generate and test method. You can generate all permutations of a list, and test whether each list is sorted. This is obviously incredibly inefficient with a Big O of \\(O(n!)\\).

```pro
partition([], Y, [], []).
partition([X|Xs], Y, {X|Ls], Rs) :- X =< Y, partition(Xs, Y, Ls, Rs).
partition([X|Xs], Y, Ls, [X|Rs]) :- X > Y, partition(Xs, Y, Ls, Rs).

quicksort([H | T], SL) :-
    partition(T, H, Ls, Rs), quicksort(Ls, SLs), quicksort (Rs, SRs), append(SLs, [H|SRs], SL).
quicksort([], []).
```

This quicksort method is a very declarative manifestation of how to do quicksort in plain English. Partition the list along the pivot, quicksort the left and right, then combine them.

## 8-Queens

One classic problem in math is to find all of the arrangements of queens on a chessboard where none of the queens threaten each other. We will find it for 8 queens on an 8×8 chessboard.

Let's cut down on our sample space, because there are *a lot* of permutations. If a queen is next to another, it obviously threatens it. In fact, because queens can never share the same row or column, that massively cuts our sample space. We can describe our solution as an array of eight values from one through eight, where the index of the value is the row of the queen and the value itself being its column. This way of describing the sample space not only cuts our sample space but makes our computation simpler. However, we have not checked the diagonals. That is what we need Prolog for.

```pro
checkBoard([H | T]) :- L is H - 1, R is H + 1, checkRow(T, L, R), checkBoard(T).
checkRow([H | T], L, R) :- H =\= L, H =\= R, LN is L - 1, RN is R + 1, checkRow(T, LN, RN)
checkBoard([]).
checkRow([], _, _).
```
