# Test cases for measuring the ulp error of a libm

This repository collects **hard-to-round test cases** for floating-point
elementary functions, together with the **correctly rounded reference result**
of every case.  They are meant for

* checking whether an implementation really is correctly rounded,
* measuring how far from correctly rounded it is (its ulp error),
* and locating the inputs on which it is wrong, ranked by how hard they are.

A case is a list of binary64 inputs plus the correctly rounded binary64
result, so testing an implementation means feeding it the inputs and comparing
the output **bit for bit**.

## What a test case looks like

```
x  y   expected x^y
0.6373378311753669 0.8925692758391124 0.6689388252228013
0.06055475955589429 0.7694172187919126 0.11560161989752284
...
```

* each line holds the binary64 arguments followed by the expected result;
* numbers are printed as the **shortest decimal string that reads back as
  exactly the same double** (Python `repr`), so parsing with `strtod` or
  `float()` recovers the exact bit pattern;
* `expected x^y` is the correctly rounded value of `x ** y` for the exact
  double arguments `x` and `y`, in round-to-nearest, ties-to-even;
* a case **fails** for a library when its result differs from the expected one
  by even one bit.

## Contents

| file | cases | ordering | difficulty |
|---|---|---|---|
| `hard_pow_0_1.txt` | 1,029,956 | hardest first | almost every case has the exact result within 2^-11 ulp of a rounding boundary |
| `hard_pow_0_1_small.txt` | 87 | hardest first | evenly graded over the whole difficulty range of the full file |
| `hard_pow_wide_y_1000000.txt` | 1,171,493 | largest result first (see below) | arguments span many binades (x from about 1e-304 to 1e304, second argument up to 1e6) while the result stays in the normal range; most cases are within 2^-4 ulp of a rounding boundary |
| `hard_pow_wide_y_1000000_small.txt` | 72 | largest result first (see below) | evenly graded over both the magnitude of the result and the distance to the rounding boundary |
| `hard_sin_0_1.txt` | 1,009,124 | hardest first | almost every case has the exact result within 2^-12 ulp of a rounding boundary |
| `hard_sin_0_1_small.txt` | 95 | hardest first | evenly graded over the whole difficulty range of the full file |
| `hard_sin_wide.txt` | 1,000,126 | largest argument first (see below) | the argument `\|x\|` spans the whole binary64 range, from 1 up to about 1e308, so this is where the argument reduction is tested; most cases are within 2^-12 ulp of a rounding boundary, the deepest one within 2^-34 ulp |
| `hard_sin_wide_small.txt` | 100 | smallest argument first (see below) | 16 steps of 2^64 in `\|x\|`, the same number of cases from every step, and the hardest case of each step among them |
| `hard_cos_0_1.txt` | 1,005,949 | hardest first | almost every case has the exact result within 2^-12 ulp of a rounding boundary |
| `hard_cos_0_1_small.txt` | 92 | hardest first | evenly graded over the whole difficulty range of the full file |
| `hard_cbrt.txt` | 1,000,011 | hardest first | the argument covers the whole binary64 range with both signs, so it exercises the full exponent and sign space; most cases are within 2^-12 ulp of a rounding boundary, the deepest one within 2^-40 ulp |
| `hard_cbrt_small.txt` | 97 | hardest first | evenly graded over the whole hardness range of the full file: the same number of cases on every level k, from k=11 up to the deepest level the full file reaches (see below) |
| `hard_pow_dyadic_small.txt` | 301 | hardest first | `y` takes 14 dyadic values (2, 3, 4, 5, 8, -1, -2, -3, 0.5, 0.25, 0.125, 1.5, -0.5, -1.5) and `x` covers the whole binary64 range with both signs; every case has the exact result within 2^-17 ulp of a rounding boundary, the deepest within 2^-28 ulp, and the cases are evenly graded over that whole range |

The file names record the argument range.  `_0_1` means the arguments are
taken from the open interval (0,1); `_wide_y_1000000` means the arguments cover
a wide range, with the second argument up to 10^6; `_wide` in a `sin` file
means the argument covers the whole binary64 range; `hard_cbrt.txt` likewise
covers the whole binary64 range, but as its own file name says nothing about
the range, it is documented here; and `hard_pow_dyadic_small.txt` keeps `y` on a
fixed list of dyadic values while `x` ranges over the whole binary64 range with
both signs.  That last file has no full-size counterpart: it is itself the
graded set.

A case is the harder the closer its exact result sits to a **rounding
boundary**, that is, to the midpoint between the two consecutive floating-point
numbers that surround it.  The three `_0_1` files, `hard_cbrt.txt` and
`hard_pow_dyadic_small.txt` are sorted so that the hardest cases come first.

In a wide argument range there is a second, independent difficulty axis: the
magnitude of the result, |y log x|, which is what makes an implementation lose
accuracy once the arguments get large.  The two `hard_pow_wide_y_1000000`
files are therefore ordered by that axis first (largest result first), and by
the distance to the rounding boundary inside it.

For the sine the corresponding axis is the magnitude of the **argument**
itself: reducing sin(x) needs about log2|x| extra bits of pi, so a library
whose reduction carries only a few dozen bits of pi is wrong on a large
fraction of the arguments here, usually by an enormous number of ulps.  The
full `hard_sin_wide.txt` is ordered by that axis first and by the distance to
the rounding boundary inside it: the largest `|x|` comes first (the file opens
at the top binary exponent and works its way down), and within one exponent the
cases closest to a rounding boundary come first, so the first block of lines is
always the hardest portion of that magnitude.  The small file does the same thing more coarsely and the other way round: `|x|`
is cut into steps of 2^64, the file starts at the *small* end, every step
contributes the same number of cases, and inside a step the hardest cases come
first.  Read from top
to bottom, a library that goes wrong from some argument magnitude onwards goes
wrong from some line onwards, which is what makes the small file useful for
locating the break rather than just detecting it.

The cube root has no such second axis.  `cbrt` is exactly scale covariant:
`cbrt(2^(3e) * x) == 2^e * cbrt(x)` holds in binary64, so the hardness of a case
depends only on the mantissa and on the exponent modulo 3, never on the
magnitude.  `hard_cbrt.txt` still covers the whole binary64 range and both
signs, but it is ordered by the distance to the rounding boundary alone, and in
its small file every level of difficulty contributes about the same number of
cases.  The same
scale covariance is the reason this function needs no `wide` variant: a handful
of binades would have produced the same difficulty distribution.

`hard_pow_dyadic_small.txt` varies `x` over the whole binary64 range, with both
signs, while `y` comes from a fixed list of dyadic values.  The cases are
ordered by how close the exact result comes to a rounding boundary, and every
level from k=16 up to k=27 contributes about the same number of them, so an
implementation can be graded simply by how far down the file it stays correct.

## Using the files

```c
/* pseudocode */
for each line (arguments..., expected) of the file:
    got = my_func(arguments...);
    if (memcmp(&got, &expected, 8) != 0)
        report a mismatch (optionally: how many ulps off)
```

What the results tell you:

* **mismatch count = 0** - the implementation is correctly rounded on this
  range, at least as far as the file reaches.
* **mismatches only near the top of the file** - the implementation is *nearly*
  correctly rounded: it goes wrong only when the exact result comes very close
  to a rounding boundary.  How deep the deepest mismatch lies is a direct
  measure of the implementation's effective accuracy.
* **mismatches spread deep into the file** - the implementation's error exceeds
  half an ulp on a substantial fraction of inputs.

For reference, mainstream libm implementations disagree with the correctly
rounded result on roughly 30 % of the cases in `hard_pow_0_1.txt`.

Directed rounding modes are not covered by these files: `expected` is always
the round-to-nearest (ties-to-even) result.
