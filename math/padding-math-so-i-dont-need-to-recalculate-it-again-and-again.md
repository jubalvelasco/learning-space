# Math used in Padding

so I don't have to recalculate it over and over again if I need it.

> [!WARNING] 
> Assumes $n$ and $m$ are integers and $n > 0$ and $m > 0$.
>
> The modulo operator `%` (symbol might be different in other programming language) for other languages might behave differently in how they treat negative values.

## Alignment Padding / Boundary Padding

from a number ($n$), get the next multiple of $m$ greater than $n$:

$n^+ = n + m - (n \bmod m)$

amount to add to get the next multiple:

$pad = m - (n \bmod m)$

This can be used in alignment padding to find the extra bytes/elements needed to be added at the end of the array so that the array length is a multiple of $m$.

> [!NOTE] 
> when talking about arrays, $n$ is the current size of the array, $m - (n \bmod m)$ is the total elements to add, and $n^+$ is the new size of the padded array.
>
> also if your array is already divisible by the block size $m$, this equation will still give you the next multiple (a whole extra block is added).
>
$$
\text{If } n \bmod m = 0,\quad m - (n \bmod m) = m,\quad\text{so}\quad n^{+} = n + m.
$$

If instead you want to round up to the nearest multiple of $m$ that is **greater than or equal** to $n$ (no padding when already aligned):

$$
n^{+} = \left\lceil \dfrac{n}{m} \right\rceil \cdot m = \left\lfloor \dfrac{n + m - 1}{m} \right\rfloor \cdot m,
\qquad
pad = (m - (n \bmod m)) \bmod m
$$

