# **An Introduction to Multi-Threading in C++ 20**

## **Multithreading for Scalability**

Amdahl's Law

$$
S = \dfrac{1}{1 - p + \dfrac{p}{n}}
$$

- $S$ = Maximum speed-up multiplier.
- $p$ = Fraction/percentage that can be parallelized.
- $n$ = Number of processor cores.

## **Built-in parallel algorithms** 

Since C++17, many standard C++ algorithms have a built-in parallel version (execution policies),
use them if there is one for that specific algorithm.

ex:

```c++
#include <algorithm>
#include <execution> // with GCC/libstdc++ you also need to link Intel TBB (-ltbb)

std::vector<MyData> data = { ... };
std::sort(
    std::execution::par,
    data.begin(), data.end(),
    MyComparator{}
);
```

Only use this at appropriate times, like in the example above
if we have a huge number of `MyData` elements then it's better to
use the parallel version of `std::sort` (for small inputs the overhead of
starting and synchronizing threads makes it slower than the sequential version).