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

The file names record the argument range.  `_0_1` means both arguments are
taken from the open interval (0,1); `_wide_y_1000000` means the arguments cover
a wide range, with the second argument up to 10^6.

A case is the harder the closer its exact result sits to a **rounding
boundary**, that is, to the midpoint between the two consecutive floating-point
numbers that surround it.  Both `_0_1` files are sorted so that the hardest
cases come first.

In a wide argument range there is a second, independent difficulty axis: the
magnitude of the result, |y log x|, which is what makes an implementation lose
accuracy once the arguments get large.  The two `wide` files are therefore
ordered by that axis first (largest result first), and by the distance to the
rounding boundary inside it.

## Using the files

```c
/* pseudocode */
for each line (x, y, expected) of the file:
    got = my_func(x, y);
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
