+++
title = "Floating Point"

date = 2022-05-28



[taxonomies]

categories = ["Computer Architecture"]

tags = ["floating point"]

+++

**Floating point representation** is a complex topic full of nuance. To understand it, we must focus on the abstract method used to represent non-integer numbers.

<!-- more -->

## Numbers in Computers

Math is infinite. Or at least it can be. Numbers like \\(5\\) can be represented finitely, but a number like \\(π\\) is not so simple to represent. \\(3.14159265358979\dots\\) but that still isn't enough. It'll never be enough.

Unfortunately, computers are not infinite. Although we would like to hold every digit of \\(π\\) in our computer's memory, the fact is that there will always be limits to how numbers can be represented in computers.

Integers are numbers like \\(3\\), \\(15\\), \\(-154\\), etc. They have no decimal point and can be negative. Representing these numbers in a computer is fairly simple, with the limit only being placed on the size of the number.

However, not all real numbers play so nicely. How do we represent the value \\(12.5\\) in memory? We can't have half bits. Can we go infinitely precise on decimals, like \\(7.2028301324\\)? What about the size? How do we set these bounds?

## Floating Point

We have to have a different standard to define how these numbers will be represented in a computer. This standard is called **floating point**.

Generally, a floating point representation is composed of three parts: the **base** \\(β\\), the **significand** \\(p\\), and the largest/smallest allowed exponents \\(e_{max}\\) and \\(e_{min}\\). This is the complete formula for floating point (ignoring specific details):

$$
\lceil \log_2{(e_{max} - e_{min} + 1)} \rceil + \lceil \log_2{β^p \rceil + 1}
$$

For example, let's say we had a floating point representation in base 2 which allowed for exponents of sizes up to \\(+128\\) and down to \\(-127\\), and had twenty-three bits of precision for the significand. This representation would need 32 bits.

A number written in floating point representation might look like this:

$$
1.010001 × 2^{7}
$$

where the first part is the significand, the base of the exponent is the base, and it's raised to its exponent. Floating point numbers are written like scientific notation to be *normalized*, where there is only one nonzero digit in front of the decimal point—this will become important for IEEE 754.

## Imprecision

I mentioned bits of precision, which is finite. Some decimal numbers can't be represented by a finite amount of precision. For example, the number \\(0.3\\) is finitely representable in base 10, but becomes \\(\overline{1.001} × 2^{-2}\\) in binary. This number will *never* be perfectly represented in binary, so we have to have approximations.

In order to understand how much error we encounter, we must be able to measure it. There are two techniques to measure error in floating point: `ulps` or "units in the last place" and *relative error*.

Let's use an example here, with \\(β = 10\\) and \\(p = 3\\). We want to approximate \\(2.781828\\) to a floating point number with this precision. Since we can only encode 3 digits of precision, we end up with \\(2.78 × 10^0\\). If we imagine a special decimal point after the precision stops for floating point, then the difference between these two is \\(0.1828\\). This is difference for `ulps`. **The closest floating point number can still have a `ulps` error of up to \\(\frac{β}{2} \cdot β^{-p}\\). This number is known as the machine epsilon \\(ε\\).**

Relative error is a familiar concept in most sciences. It is simply the difference between measured and actual, divided by the actual value. In this case, our floating point approximation \\(2.78 × 10^0\\) is the "measured" value and our real number \\(2.781828\\) is our actual value. In this case, it is \\(0.001828 / 2.781828 = 0.0006\\). Since this number can be small, it is often expressed in terms of \\(ε\\). In this example, \\(ε = 5 × 10^{-3} = 0.005\\) so our relative error is \\(0.13ε\\).

Since we know the maximum `ulp` error for the closest floating point number, we should find the same for relative error. One difference about relative error is that it changes based on the size of the numbers, not just the absolute error in the significand. The largest possible error is \\(\frac{β}{2}β^{-p} × β^e\\).

However, the relative error changes based on the size of the real number in question, and all real numbers within the same exponent range will have this error. So, the relative error for a number closer to \\(1.0 × β^e\\) will be larger than for a number closer to \\(β × β^e\\). This variation is called **wobble**. The relative error is always bounded by \\(ε\\), as in the prior example. Depending on the significand, however, the wobble can be up to \\(β\\) for the same exponent. Crucially, this wobble can be expressed in either relative error or `ulps`, as long as the other is held fixed.

Typically, relative error is used, because it is more useful for compounding operations whereas `ulps` can vary wildly within \\(β\\).

An important concept to understand here is **contaminated digits**. These are the least significant digits of a floating point number which may be error-prone and are therefore untrustworthy. The number of contaminated digits is \\(\log_β{n}\\) where \\(n\\) is the factor of the relative error to \\(ε\\), like \\(0.13\\) in the earlier example.

## Error Mitigation

We have found how to measure error; now how do we reduce it? One method is to use **guard digits**. When performing calculations, a computer can "extend" the calculation by a few digits so that the extended digits are contaminated by the floating point operation. These digits will then be rounded out of the final result, reducing the contamination of the result.

Without a guard digit, the relative error can be large as \\(β - 1\\), which can put every digit in error! However, adding just *one* guard digit bounds the relative error to *less than \\(2ε\\)!*

Another method is to reduce the number of **cancellations**. A cancellation occurs when nearby quantities are subtracted. When this happens, the most significant and uncontaminated digits cancel out and the less significant, more contaminated digits are left. Cancellations can seriously magnify rounding errors in a series of cancellations; this is called *catastrophic cancellation*. However, cancellation can also be *benign*. If the two quantities being subtracted are exactly known, then the subtraction will have a tiny relative error if done with a guard digit.

Sometimes, we can rearrange formulas to have less or more benign cancellations. For example, the formula \\(x^2 - y^2\\) has a catastrophic cancellation because \\(x^2\\) and \\(y^2\\) both suffer from rounding error. However, this formula can be rearranged to \\((x ⊕ y) ⊗ (x ⊖ y)\\). The cancellation is now benign because \\(x\\) and \\(y\\) are presumably exact values without rounding error yet.

> Notice that the operands in that formula are circled. This is a notation to indicate that these operations are performed by a computer and therefore may accrue rounding error, whereas the ordinary operands are used for exact calculations.

However, the impact of making a cancellation benign can be limited if the inputs to the equation are already inexact. This is common when converting between decimal and binary numbers. **It is always worth eliminating a cancellation, though.**

## Exact Rounding

Although guard digits mitigate error, they will in many cases give a different value than the **exactly rounded** result. That is the result if the floating point number was computed exactly, then rounded to precision. Many algorithms require exact rounding to work properly.

The nature of rounding is controversial. The most commonly accepted rounding mode is *round to even*. This will round numbers that end in the half digit to whichever direction makes the number even. For example, \\(12.5\\) will round to \\(12\\), not \\(13\\) because \\(12\\) is even. This achieves the result of rounding up and down being equal chance for the half digit, 5 in this case.

Exact rounding is useful for holding certain invariants in floating point calculation. For example, one way to increase precision in calculations is to split a multiple precision number into an array of single precision numbers. Adding these numbers back together with exact rounding will recover the multiple precision numbers.

We can represent a multiplication of two double precision floating point numbers \\(x\\) and \\(y\\) using this method. We can split \\(x\\) into \\(x_h\\) and \\(x_l\\), each of which are single precision, and \\(y\\) into \\(y_h\\) and \\(y_l\\) likewise. Then, we can represent the multiplication as so:

$$
x \cdot y = (x_h + x_l)(y_h + y_l) = x_hy_h + x_hy_l + x_ly_h + x_ly_l
$$

In this way, a multiplication of two double precision numbers can be represented as the sum of multiplications of single precision numbers. Because the numbers are exactly rounded, the number is completely recoverable. This means that double precision multiplication can be possible on a computer which only supports single precision multiplication.

*All this information comes from the landmark paper [What Every Computer Scientist Should Know About Floating-Point Arithmetic](https://docs.oracle.com/cd/E19957-01/800-7895/800-7895.pdf)*
