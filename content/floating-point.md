+++

title = "Floating Point"

date = 2022-05-28



[taxonomies]

categories = ["Computer Architecture"]

tags = ["floating point"]

+++

[What Every Computer Scientist Should Know About Floating-Point Arithmetic](https://docs.oracle.com/cd/E19957-01/800-7895/800-7895.pdf) is one of the most important papers in computer science for understanding one of its most fundamental concepts.

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
