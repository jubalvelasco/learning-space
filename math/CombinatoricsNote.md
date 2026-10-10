# Combinatorics Note

## Sum of Geometric Series

a sequence of numbers where the ratio between any two consecutive numbers is constant. For example, $3,6,12,24$ is a geometric progression with a common ratio $k = 2$, its first term is $a = 3$ and $b = 24$ is the last term.

$$
s = a + ak + ak^2 + ak^3 + ... + b = \frac{bk - a}{k - 1}, \quad (k \neq 1)
$$

## Total Vertices Of A Perfect K-Ary/N-Ary Tree

A K-ary or N-ary tree is a tree where each node has a maximum of $k$ children, it generalizes binary trees ($k = 2$), ternary trees ($k = 3$), etc.

A **perfect** k-ary tree is where all the nodes starting from the root up to the nodes at $d - 1$ depth have a fixed number of $k$ child nodes, only until $d - 1$ because the nodes at depth $d$ do not have children since they are the leaf nodes, we can use the formula below to determine the total vertices a perfect k-ary tree has.

$$
v = \frac{k^d - 1}{k - 1}, \quad (k \gt 1 \quad \land \quad d \ge 0)
$$

**NOTE**: some textbooks might say that the root node is at depth 0, or count the depth via the number of edges going from the root to a specific node, but for this example we define the root node being at depth 1, and any of its children would be at depth 2, so on and so forth.

This formula for finding the total number of vertices/nodes of a perfect k-ary tree (with depth $d$) comes from the geometric series (level $i$ has $k^{i-1}$ nodes, so there is one term per level and the number of terms is $m = d$).

$$
\begin{align}
v &= \dfrac{bk - a}{k - 1} \\
&& \text{in the geometric series } a, ak, ak^2, ..., b \\
&& \text{if there are } m \text{ terms, } \\
&& \text{we could say that the last term } b = ak^{m-1} \\
&& \text{and if we substitute it for } b \text{ we would get} \\
v &= \dfrac{(ak^{m-1})k - a}{k - 1} \\
&& \text{simplify } (ak^{m-1})k \text{ via exponential rule} \\
v &= \dfrac{ak^m - a}{k - 1} \\
&& \text{factor out } a \text{ from } ak^m - a \\
v &= \dfrac{a(k^m - 1)}{k - 1} \\
&& \text{and since } a \text{ is the starting term} \\
&& \text{and in a tree there is only one root node} \\
&& \text{therefore } a = 1 \\
v &= \dfrac{1(k^m - 1)}{k - 1} \\
v &= \dfrac{k^m - 1}{k - 1} \\
&& \text{and since } m = d \\
v &= \dfrac{k^d - 1}{k - 1} \\
\end{align}
$$

## N-th Triangular Number

The n-th triangular number is the shortcut of the **summation of incrementing numbers** starting from $1$ up to $n$ (eg: $1 + 2 + 3 + ... + n$).

We can also use the N-th triangular number to **find the total number of sub-arrays** since there are $n$ subarrays of length $1$, there are $n - 1$ subarrays of length $2$... so on and so forth, as we can see this is similar to the incrementing number sequence so we can use the N-th triangular number to find the total number of sub-arrays too.

$$
s = \frac{n(n+1)}{2}
$$

Another nice application of n-th triangular number: Given ($n$) players in a chess tournament (or any 1v1 game) in a round-robin format (each player needs to play against all other players), we can find the total ($m$) number of games or total matchups using the n-th triangular number **but we need to subtract $1$ from $n$** first because a player cannot play against itself.

$$
\begin{align}
m &= \frac{(n - 1)(n - 1 + 1)}{2} \\
m &= \frac{(n - 1)n}{2} \\
m &= \frac{n(n - 1)}{2}
\end{align}
$$

(_similarly we can also use this to calculate the total number of connections/edges in a fully connected network/graph with_ $n$ _nodes and each node strictly has no self connection or self loop._)

(_if you also want to include self loops for all nodes then just don't subtract 1 and use the n-th triangular formula as is._)

## Arithmetic Progression Sum

The n-th triangular number is actually a case of the sum of arithmetic progression, an arithmetic progression is a sequence of numbers where the difference between any two consecutive numbers is constant, for example $3, 7, 11, 15$, we can calculate the sum of arithmetic progression using the same formula.

$$
m = \frac{n(a+b)}{2}
$$

where $a$ is the starting number, $b$ is the last number and $n$ is the total count of the given numbers, if we look at the summation formula in the n-th triangular number, in the $(n + 1)$, the number $1$ is the starting number and $n$ is the last number too.

## Total Number of Subset

Given a set of size $n$, how many subsets ($s$) does it have including the improper subset and the empty set?

$$
s = 2^n
$$

Remember this ($s = l + m + n + o$) equation:
- $l = 1$ representing the number of empty set
- $m = 1$ representing the number of improper subset or the set itself
- $n$ is number of all possible sets with cardinality of only 1
- and $o$ is every other set that is not an empty, not an improper subset, and sets that has a cardinality greater than 1.

## Permutation vs Combination

Consider the following scenario, there are 5 different kinds of fruit and 3 boxes.

- $n = 5$
- $r = 3$

### Permutation - order matters.

You are tasked to find out how many ways you can put one fruit (you can choose) in each of the boxes (there are only one fruit per kind, so you cannot repeat putting apples in multiple boxes), to get the total ways to do it you can use the permutation formula below

$$
{}^{n}P_{r} = P(n,r)=\frac{n!}{(n-r)!},\quad(0\le r\le n).
$$

### Combination - order does not matter (unordered).

You are tasked to count how many ways to place one fruit in each of the 3 boxes. That is, the arrangement is the same no matter which box
holds which fruit, for example, (Box1: apple, Box2: orange, Box3: mango) is identical to (Box1: mango, Box2: apple, Box3: orange),
to get the total number of ways, you can use the combination formula below.

$$
{}^{n}C_{r} = C(n,r)=\frac{n!}{(n-r)! \cdot r!},\quad(0\le r\le n).
$$

**NOTE:** in some sources $k$ is used instead of $r$.

## Finding all possible number of pairs from the elements of set A & B (order DOES NOT matter)

(these formulas assume the sets are disjoint, i.e. they don't share elements)

```py
A = { a, b }, B = { c }
C = {{ a, c }, { b, c }}
```

$$
n = |A| \cdot |B |
$$

## Finding all possible number of pairs from the elements of set A & B (order matters)

```py
A = { a, b }, B = { c }
C = {{ a, c }, { b, c }, { c, a }, { c, b }}
```

$$
n = 2! \cdot (|A| \cdot |B|)
$$

**NOTE:** If you have 3 sets and you want to get the total number of possible ordered triples, then the formula would look like this.

$$
n = 3! \cdot (|A| \cdot |B| \cdot |C|)
$$

Notice that you could extend it to multiple sets using the same idea.

$$
n = s! \cdot (\prod_{i=1}^{s} |S_i|)
$$

Here the $s$ is the number of sets, $S_i$ is a set we get when we iterate over all of the sets, and for $|S_i|$ we just get the size/cardinality of the current set during iteration.

## Permutations of *n* distinct elements

**Question.** Given a sequence of *n* **distinct** elements, how many different sequential arrangements (permutations or *P*) are possible?

**Answer:** $P = n!$

**Example:** `ABC`

$$
P = 3! = 6
$$

Which are `ABC`, `ACB`, `BAC`, `BCA`, `CAB`, `CBA`.

**Notes:**

* This assumes all elements are distinct.
* By convention ($0! = 1$).
* If you want the number of ordered arrangements of length (sub) (without repetition) from (n) distinct elements, use the permutation formula

$$
P(n,sub)=\frac{n!}{(n-sub)!},\quad(0\le sub\le n).
$$

## Permutations of an *n*-element sequence with repeated values

**Question.** Given a sequence of (n) elements that may include repeated values, how many distinct sequential arrangements (permutations) are possible?

**Answer:**
Say the sequence contains $(k)$ distinct symbols and the multiplicities (counts) of each symbol are represented by the elements of the vector $M$, where $(|M| = k)$.

$$
M_1,M_2,\dots,M_k,\quad(\sum_{i=1}^k M_i = n)
$$

then the number of distinct permutations is the multinomial formula

$$
P=\dfrac{n!}{M_1! M_2! \cdots M_k!}
$$

Here, dividing by $M_1! M_2! \cdots M_k!$ corrects for the overcounting caused by swapping identical items of the same symbol.

**Example:** sequence `AABCC`.

* $(n=5, \quad k=3)$.
* Distinct symbols: $(A,B,C)$ with multiplicities $(M_1=2, \quad M_2=1, \quad M_3=2)$ or count of each unique symbol.

$$
P=\frac{n!}{M_1! \cdot M_2! \cdot M_3!}=\frac{5!}{2!\cdot1!\cdot2!}=\frac{120}{2\cdot1\cdot2}=\frac{120}{4}=30.
$$

## Number of possible *n*-digit sequences in base (B)

**Question.** For a digit system of base (B), how many different (n)-digit sequences are possible? (Here an "n-digit sequence" means an ordered string of (n) digits; leading zeros are allowed.)

**Answer:** $C = B^n$

**Example:** for base 10 $(B=10)$ and $(n=2)$ digits, there are $C = 10^2 = 100$, possible sequences, namely `00` through `99`.

**Note:** There might be tricky questions that disguise themselves as a **set** type of question but they are **not**. For example, given a set $Y = \\{ a, b, c \\}$, if you're tasked to find the total number of **ordered** selections (sequences) of length $2$ where elements can be repeated, this is the same problem: each element of the set can be considered as a digit, the cardinality of the set is the base $B$, and the length is $n$, so the answer is $3^2 = 9$ (`aa, ab, ac, ba, bb, bc, ca, cb, cc`).

But be careful: if the order does **not** matter (e.g. `ab` and `ba` count as the same selection, i.e. multisets / combinations with repetition), $B^n$ over-counts. The correct count is:

$$
\binom{B + n - 1}{n}
$$

For the same example, $\binom{3 + 2 - 1}{2} = \binom{4}{2} = 6$ (`aa, bb, cc, ab, ac, bc`).

If digits cannot repeat and $(n \le B)$, the count is the permutation of $(B)$ taken $(n)$ at a time:

$$
P(B,n)=\frac{B!}{(B-n)!}.
$$
