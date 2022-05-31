+++
title = "Floating Point Architecture"

date = 2022-05-31



[taxonomies]

categories = ["Computer Architecture"]

tags = ["floating-point"]

+++

Floating point numbers are complicated enough in the abstract. How do systems programmers and hardware designers cope with the complexity of floating point?

<!-- more -->

## Floating Point Instructions

In the general computer architecture, CPUs function by executing a stream of instructions. The set of instructions is defined by the hardware model, such as x86 or ARM. Instructions are typically tiny operations that need to be done extremely often. Some examples of instructions are `mov` to move values in and out of memory, `sub` to subtract numbers in registers, etc. In order to speed up floating point operations, hardware can include floating point instructions instead of letting software handle it.

Most of the time, instruction sets only allow floating point operations in the same precision. However, there is a useful optimization that is not often implemented in hardware. A hypothetical instruction to multiply two single precision numbers that results in a double precision number would be very useful. In particular, it would enable for the use of *iterative improvement* algorithms that would otherwise have compounding catastrophic cancellations if the precision was not increased temporarily.

## Subexpression Evaluation

The order of operations is very important for preserving invariants in floating point. For example, floating point numbers do not necessarily follow the associative property, so compilers that optimize out parentheses will exhibit undefined behavior.

This can also cause issues when numbers with different precisions are operated on. There are two solutions to this problem. One is the solution that languages like OCaml take where all values in an arithmetic expression must have the same type. This means that integers will never be implicitly cast to floating point when they are operated with them, because they are not allowed to be operated with them anyway. However, this can be an overly strict restriction on the types of programs that can be written.

Another solution is the one that C takes, which is to establish rules for subexpression evaluation while allowing mixed type expressions. K&R C requires that every operation be done in double precision, but this can cause incompatibility between expressions and a stored value that are in different precisions.

A better way to do subexpression evaluation is to set a tentative precision for each expression, then increase the precision as needed in the wider expression. However, this means that the precision of a sub-expression can change with the expression it is embedded in, which is potentially unexpected behavior for a programmer.

## IEEE Conformation

Languages like C conform to the IEEE specification, but the ways in which they do it can cause issues in implementation. If the exact rounding operation is implemented in hardware, then all the language has to do is call the appropriate instruction. 

However, there are some aspects of floating point that are harder to represent.  Floating point has a certain *state* associated with it, with rounding mode, flags, trap handlers, etc. This state must be read and written to, and must be preserved across subroutines.

`NaN`s also cause issue. The reflexive property is typically taken as invariant since it is in integer math, but it can be disastrous with `NaN`s. One massive consequence of the inclusion of `NaN`s is that floating point numbers *cannot have a total order*. This is because `NaN`s are explicitly unordered with respect to other floating point numbers. This has cascading consequences for the implementation of comparison operators like <, >, and =.

## "Optimization"

Compilers make many optimizations to programs in order to increase performance or reduce instruction count. These optimizations are usually made with certain invariants in mind so that the accuracy of the program is not affected. However, because floating point has different invariants than integer math, compilers can make mistakes when optimizing floating point math. For example,

```c
float ε = 1;
do {
    ε = 0.5 * ε;
} while (ε + 1 > 1);
```

will estimate \\(ε\\). However, a compiler might notice that `ε + 1 > 1` for integer math is equivalent to `ε > 0` and make that optimization, without considering that the expression \\(ε ⊕ 1\\) has a special meaning that is different for floating point numbers than a test for positivity.

In general, many algorithms with floating point will exhibit expressions that, upon first blush, seem to be redundant in integer math. Having consideration for the ways in which floating point math can change values is important.

## Exceptions

Trap handlers are empowered to be able to access variables in programs. However, computers which have parallel arithmetic may not necessarily be able to easily identify which operation threw an exception. Trap handlers must also be able to identify programs.

The reordering of certain instructions can also cause issues. Compilers will often change the order of certain arithmetic instructions when they are exact so that the semantics don't change. However, if the operation traps, then it is not as easy to identify the operation that trapped, since the operation that happened in parallel will modify the trapped arithmetic.

One solution to this is *presubstitution* where a user that knows that an exception can create their own handler for an exception and substitute the value themselves beforehand. However, because it goes against IEEE-754, it's unlikely to be proliferated.

*All this information comes from the landmark paper [What Every Computer Scientist Should Know About Floating-Point Arithmetic](https://docs.oracle.com/cd/E19957-01/800-7895/800-7895.pdf)*
