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

The `_0_1` suffix records the argument range: both arguments are taken from the
open interval (0,1).

A case is the harder the closer its exact result sits to a **rounding
boundary**, that is, to the midpoint between the two consecutive floating-point
numbers that surround it.  Both files are sorted so that the hardest cases come
first.

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
