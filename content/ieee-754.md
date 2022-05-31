+++
title = "IEEE-754"

date = 2022-05-31



[taxonomies]

categories = ["Computer Architecture"]

tags = ["floating-point"]

+++

**IEEE-754** is the standard for floating point computation around the world. Thus, any real-world discussion of floating point must include it.

<!-- more -->

## Format

IEEE-754 floating point numbers are always in base 2 binary; this makes sense as it minimizes wobble which scales with \\(β\\). It also allows for a special optimization that can add an extra bit of precision without increasing the size of the number. To demonstrate that, we can look at the number \\(001010.0011\\). The normalized floating point representation of this is \\(1.0100011 × 2^3\\). Notice how the digit in front of the decimal point is always 1. It can't be 0, because then we could just shift the decimal point and throw away the zero as we did with the initial number. Therefore, IEEE-754 assumes that the first bit is always 1 and only encodes the bits after the decimal point, granting an extra 1. There is an obvious problem with this: the number 0 cannot be represented. In IEEE-754, if the significand and exponent are all 0, then the number is the special case of 0.

IEEE-754 allows for four possible precisions: single, double, single-extended, and double-extended:

| Parameter         | Single | Single-Extended | Double | Double-Extended |
| ----------------- | ------ | --------------- | ------ | --------------- |
| \\(p\\)           | 24     | 32              | 53     | 64              |
| \\(e_{max}\\)     | +127   | +1023           | +1023  | +16383          |
| \\(e_{min}\\)     | -126   | -1022           | -1022  | -16382          |
| bits for exponent | 8      | 11              | 11     | 15              |
| total # of bits   | 32     | 43              | 64     | 79              |

The usefulness of extended precision is that it grants a large amount of guard digits for internal calculations, for example in a calculator. There are fast algorithms for common transcendental functions like \\(\log\\), but most of them have a large amount of possible error. By using extended precision internally then rounding to regular precision on display, calculators can quickly compute transcendental functions for users accurately.

The exponent value is calculated using a *bias* so it can represent negative values. The initial unbiased exponent is subtracted by \\(2^{e - 1} - 1\\), e.g. 127 for single precision. The reason that the \\(e_max\\) is always bigger than \\(e_min\\) is to prevent overflow when the reciprocal of the smallest numbers are taken. In turn, the reciprocal of the largest numbers will create underflow. In general, underflow is preferable to overflow, for reasons that will be clear when we discuss overflow in more depth.

Operations between floating point numbers in IEEE-754 are always exactly rounded. Having a guard digit can reduce error, but it does not guarantee the same result as exact rounding, so it cannot be used. There are methods using two guard digits and a "sticky" bit that can be used for efficient exact rounding, however. Exact rounding is important so that the result of two operations can always be guaranteed to be the same, regardless of hardware. The operations defined under IEEE to be exactly rounded are +, -, ×, /, √, %, and conversion with integers. Conversion between binary and decimal is *not* necessarily exactly rounded, because the most efficient conversion algorithms are not exactly rounded. Transcendental functions are also not specified as exactly rounded because there is no efficient algorithm that works across all hardware.

## `NaN` and ∞

If you've used a calculator before, chances are you have seen both of these terms before. But what do they mean in the context of floating point?

Not all bit patterns in IEEE-754 follow the rules that were laid out earlier for significands and exponents. Some aren't even valid floating point numbers! There are special values that are possible:

| Exponent                    | Fraction | Represents                                 |
| --------------------------- | -------- | ------------------------------------------ |
| \\(e = e_{min} - 1\\)       | f = 0    | ±0                                         |
| \\(e = e_{min} - 1\\)       | f ≠ 0    | \\(0.f × 2^{e_{min}}\\) (**denormalized**) |
| \\(e_{min} ≤ e ≤ e_{max}\\) | —        | \\(1.f × 2^e\\) (**normalized**)           |
| \\(e = e_{max} + 1\\)       | f = 0    | ±∞                                         |
| \\(e = e_{max} + 1\\)       | f ≠ 0    | `NaN`                                      |

The middle row represents normalized numbers, which we are already familiar with. We will discuss each of the other special values.

As mentioned earlier, 0 is an exception where the significand and exponent are both 0. But wait, why can 0 be positive *and* negative? IEEE-754 also defines that \\(+0 = -0\\) so the distinction seems useless. The reason both are allowed is because it preserves the sign of infinity when they interact. It also allows for functions that are only defined for positive or negative numbers to be able to safely include 0 in them.

*Denormalized numbers* are numbers with 0 as the hidden bit instead of 1. The reason they need to exist and break these rules is because small numbers being subtracted can end up underflowing to 0. It breaks a common invariant in code that the difference between two unequal numbers is not 0, and can cause bugs. Denormalized numbers gradually underflow to 0 so this problem does not occur. It also significantly reduces relative error for very small numbers.

The idea of infinity is an important one to measure for floating point. Unlike with integer arithmetic, division by zero is not an immediate error which aborts the operation. Rather, overflow can be an expected result which is handled for by using infinity. If the infinity ends up in a denominator, then the computation can underflow to 0 which can be an expected result.

`NaN` stands for "Not a Number" and is perhaps the most unique special value`NaN` does not represent any computable value, and it is "infectious"; any operation with`NaN` will result in an `NaN` and `NaN`s always compare to false, even with other `NaNs`. They are a result of computations like \\(0/0\\) or \\(\sqrt{-1}\\) which are not well-defined in real numbers, and also cannot be represented by infinity. These are all the operations that will produce an `NaN`:

| Operation | Production                     |
| --------- | ------------------------------ |
| +         | \\(∞ + (-∞)\\)                 |
| ×         | \\(0 × ∞\\)                    |
| /         | \\(0/0\\), \\(∞/∞\\)           |
| %         | \\(x % 0\\), \\(∞ \% y\\)      |
| √         | \\(\sqrt{x}\\) for \\(x < 0\\) |

Unlike with denormalized numbers, the value in the significand does not necessarily represent any specific values. Often, some information about the operation will be placed in the significand as a signal. If an operation is done between a real number and an `NaN`, it will not change the significand of the `NaN`.

## Exceptions

`NaN` and ∞ allow floating point calculations to continue in the face of special circumstances without aborting immediately. However, this behavior is not always appropriate. Implementations of IEEE-754 typically have **trap handlers** that will handle **exceptions** generated by such circumstances. There are five classes of exception that can each be set by a status flag: overflow, underflow, division by zero, invalid operations, and inexact operations. Invalid operations are any that involve `NaN` except for those when one of the operands are already `NaN`.

Operations in IEEE-754 must be exact, so if an operation is performed that is inexact, it will raise an exception. However, this raises an issue. Inexact exceptions can happen extremely often, and telling the OS to summon the trap handler every time harms performance. In order to prevent this, a software flag is typically enabled upon inexact exception to mask off future exceptions until the flag is reset.

Although trap handlers can abort the algorithm, they can also exhibit other behavior which can aid algorithms in being more efficient and accurate. For example, IEEE-754 will wrap around overflowing numbers by dividing the computed result by \\(2^α\\). \\(α\\) is 192 for single precision and 1536 for double precision. This is useful in partial products when the operation might overflow and underflow at several points. By allowing the products to continue without aborting, the final product might cancel out under/overflows and be in range without having to stop the OS at every point.

## Rounding

We stated earlier that round to even is the best and most commonly accepted method of rounding. This is the default mode of IEEE-754. However, there are other acceptable rounding modes that can be enabled for certain operations. These are round to 0, to +∞, and to -∞. These modes turn rounding into floor/ceiling operations, which are useful for computing intervals.

We can represent floating point results as an interval between two numbers where the first is rounded to -∞ and the second is rounded to +∞. The exact result is somewhere between those two numbers. This representation can be useful to get an idea of how wide your error is when calculating a result. If the result with single precision has a very wide interval, then you might redo the result in double precision and so on to shrink the interval to some acceptable level of error.

*All this information comes from the landmark paper [What Every Computer Scientist Should Know About Floating-Point Arithmetic](https://docs.oracle.com/cd/E19957-01/800-7895/800-7895.pdf)*
