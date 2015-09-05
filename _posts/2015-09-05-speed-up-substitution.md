---
layout: post
title: Speeding up byte substitution
---

Let us speed up the following program:

```c
char translate[256] = {substituion table};
const char* input = "input string";
int len = length of input string;
char* output = malloc(len);
for (int i = 0; i < len; ++i) {
    output[i] = translate[input[i]];
}
```

This code solves many practical problems:

 * [substitution cipher][cipher],
 * [complementary DNA calculation][complement].

<!-- more -->
<a name="cut" id="cut"></a>

To speed up this code approximately twofold, replace
`char translate[256]` table with
 `short short_translate[256*256]`.
This table has size `256 * 256 * 2 = 131072` bytes and
is generated once per a substitution table.

```c
int x, y;
for (x = 0; x < 256; x++) {
    for (y = 0; y < 256; y++) {
        unsigned short in_xy = (x << 8) | y;
        unsigned short out_xy =
            ((unsigned short)(translate[x]) << 8) |
            (unsigned short)(translate[y]);
        short_translate[in_xy] = out_xy;
    }
}
```

And use `short` instead of `char` in the code:

```c
short* short_input = (short*)input;
short* short_output = (short*)output;
int short_len = (len + 1) / 2;
for (int i = 0; i < short_len; ++i) {
    short_output[i] = short_translate[short_input[i]];
}
```

That's all.

It works two times faster because it needs two times less
table lookups.

Note: we can't use `int` instead of `short` because
a substitution table for `int` would occupy
`256 ^ 4 * 4 = 16G`. A table of that size is too large to
fit in CPU cache.

See [the code and the benchmark][github].

[cipher]: https://en.wikipedia.org/wiki/Substitution_cipher
[complement]: https://en.wikipedia.org/wiki/Complementary_DNA
[github]: https://github.com/starius/dna-complement
