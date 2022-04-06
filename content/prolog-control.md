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

**A substitution \\(σ\\) is a finite set of pars of terms \\(\\{X_1/t_1, ..., X_n/t_n\\}\\) where each \\(t_i\\) is a term and each \\(X_i\\) is a variable such that \\(X_i ≠ t_i\\) and \\(X_i ≠ X_j\\) if \\(i ≠ j\\).**

An empty substitution is denoted by the letter \\(ε\\).

Some important rules:

- A variable cannot substitute itself e.g. \\(Z/Z\\) is illegal.

- Only a variable can be substituted e.g. \\(m/n\\) is illegal because \\(m\\) is an atom.

The meaning of applying a term to a substitution is that every occurrence of \\(X_i\\) in the compound term is replaced with the corresponding substituent in \\(σ\\). This application is known as **instantiation**, and that new compound term \\(Eσ\\) is an **instance**.

Bringing back the earlier rules about unification, you can see how unification is derived from substitution. Variables can unify with anything, so they are the term that is substituted in Prolog.


