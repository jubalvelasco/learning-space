# Algebra Notes For Equation Manipulation

These notes cover techniques for manipulating algebraic equations — from
fundamental rules to advanced numerical strategies. The goal is not just
to solve equations, but to reshape them into forms that are cleaner,
more insightful, or more computationally practical.

---

# Fundamentals

## Removing Divisor In One Side

You can remove a divisor from one side by multiplying both sides by that divisor.

$$
\begin{align}
m &= \dfrac{l + r}{2} \\
\\
\\
m \cdot 2 &= \dfrac{l + r}{2} \cdot 2 \\
\\
\\
2m &= l + r
\end{align}
$$

## Adding or Subtracting the Same Value on Both Sides

Any term can be moved to the other side of the equation by adding or subtracting
the same value from both sides.

$$
\begin{align}
x + 5 &= 12 \\
\\
\\
x + 5 - 5 &= 12 - 5 \\
\\
\\
x &= 7
\end{align}
$$

## Multiplying or Dividing Both Sides

You can scale both sides by any non-zero constant without changing the equation's meaning.

$$
\begin{align}
3x &= 15 \\
\\
\\
\dfrac{3x}{3} &= \dfrac{15}{3} \\
\\
\\
x &= 5
\end{align}
$$

Never divide by zero. If the expression you are dividing by contains a variable,
you must first verify it cannot be zero.

## Transposing Terms

Moving a term across the equals sign flips its sign. This is shorthand for
adding or subtracting from both sides.

$$
\begin{align}
x + b &= c \\
\\
\\
x + b - b &= c - b \\
\\
\\
x &= c - b
\end{align}
$$

## Distributive Property

Expanding or factoring expressions on either side can reveal structure or simplify further steps.

**Expanding:**

$$
\begin{align}
2(x + 3) &= 14 \\
\\
\\
2x + 6 &= 14 \\
\\
\\
2x + 6 - 6 &= 14 - 6 \\
\\
\\
2x &= 8 \\
\\
\\
\dfrac{2x}{2} &= \dfrac{8}{2} \\
\\
\\
x &= 4
\end{align}
$$

**Factoring in reverse:**

$$
\begin{align}
2x + 2y &= 10 \\
\\
\\
2(x + y) &= 10 \\
\\
\\
\dfrac{2(x + y)}{2} &= \dfrac{10}{2} \\
\\
\\
x + y &= 5
\end{align}
$$

## Combining Like Terms

Simplify each side by combining terms that share the same variable and exponent
before performing other manipulations.

$$
\begin{align}
3x + 2x - 4 &= 11 \\
\\
\\
5x - 4 &= 11 \\
\\
\\
5x - 4 + 4 &= 11 + 4 \\
\\
\\
5x &= 15 \\
\\
\\
\dfrac{5x}{5} &= \dfrac{15}{5} \\
\\
\\
x &= 3
\end{align}
$$

## Substitution

If one equation defines a variable, you can substitute its expression into another
equation to reduce the number of unknowns.

Given:

$$y = 2x + 1$$

Substitute into $3x + y = 10$:

$$
\begin{align}
3x + y &= 10 \\
\\
\\
3x + (2x + 1) &= 10 \\
\\
\\
5x + 1 &= 10 \\
\\
\\
5x + 1 - 1 &= 10 - 1 \\
\\
\\
5x &= 9 \\
\\
\\
\dfrac{5x}{5} &= \dfrac{9}{5} \\
\\
\\
x &= \dfrac{9}{5}
\end{align}
$$

---

# Common Pitfalls

## Forgetting to Apply an Operation to the Entire Side

When multiplying or dividing, the operation must apply to the whole side, not just one term.

**Wrong** — the $2$ in the numerator cancels but $b$ is left untouched:

$$2 \cdot \dfrac{a + b}{2} = 2a + b$$

**Right** — the entire expression $a + b$ is what remains after cancellation:

$$2 \cdot \dfrac{a + b}{2} = a + b$$

## Sign Errors When Distributing a Negative

Negatives must be distributed to every term inside the parentheses.

**Wrong:**

$$-(x - 3) = -x - 3$$

**Right:**

$$-(x - 3) = -x + 3$$

## Dividing Both Sides by a Variable Without Checking for Zero

Dividing by an expression containing a variable can silently discard solutions
where that variable equals zero.

**Wrong** — dividing both sides by $x$ loses the solution $x = 0$:

$$
\begin{align}
x^2 &= 3x \\
\\
\\
\dfrac{x^2}{x} &= \dfrac{3x}{x} \\
\\
\\
x &= 3
\end{align}
$$

**Right** — move all terms to one side and factor instead:

$$
\begin{align}
x^2 &= 3x \\
\\
\\
x^2 - 3x &= 0 \\
\\
\\
x(x - 3) &= 0
\end{align}
$$

Therefore $x = 0$ or $x = 3$.

## Squaring Both Sides Introduces Extraneous Solutions

Squaring both sides to eliminate a square root can introduce solutions that do not
satisfy the original equation. Always verify by substituting back.

$$
\begin{align}
\sqrt{x} &= -3 \\
\\
\\
\left(\sqrt{x}\right)^2 &= (-3)^2 \\
\\
\\
x &= 9
\end{align}
$$

Checking: $\sqrt{9} = 3 \neq -3$, so $x = 9$ is an extraneous solution. The original equation has no solution.

## Incorrectly Cross-Multiplying

Cross-multiplication is only valid when **both** sides are single fractions.
When one side has extra terms, isolate the fractions first.

**Wrong** — the left side is not a single fraction:

$$\dfrac{a}{b} + c = \dfrac{d}{e} \implies ae + c = db$$

**Right** — isolate the fraction, then cross-multiply:

$$
\begin{align}
\dfrac{a}{b} + c &= \dfrac{d}{e} \\
\\
\\
\dfrac{a}{b} &= \dfrac{d}{e} - c \\
\\
\\
\dfrac{a}{b} &= \dfrac{d - ce}{e} \\
\\
\\
ae &= b(d - ce)
\end{align}
$$

## Losing Solutions When Taking a Square Root

Taking the square root of both sides requires a $\pm$ sign, since both a positive
and negative root are valid.

**Wrong** — only the positive root is considered:

$$
\begin{align}
x^2 &= 25 \\
\\
\\
x &= 5
\end{align}
$$

**Right:**

$$
\begin{align}
x^2 &= 25 \\
\\
\\
\sqrt{x^2} &= \sqrt{25} \\
\\
\\
x &= \pm 5
\end{align}
$$

---

# Advanced

## Equation Scaling or Numerical Stabilization

Say for example you want to find a form of the equation that would benefit the calculation you want to do,
for example given the mid point formula and you're calculating with very large values and you want
to manipulate the equation to a form that minimizes the overflow in calculators and computers as much as possible.

A good example is the mid point formula where $l + r$ can produce a very big number if $l$ and $r$ are also a very big value the sum can overflow.

$$
m = \dfrac{l + r}{2}
$$

Let us get rid of the divisor on the right hand side first.

$$
2m = l + r
$$

### Forcefully Expanding An Expression In The Other Side Of The Equation

We can then forcefully expand a very simple expression in one side of the equation and (this feels non-sense sometimes but we can) try to get an advantage out of it, like the example below.

$$
\begin{align}
2m &= l + r \\
\\
\\
2m &= l + l - l + r \\
\\
\\
2m &= 2l - l + r \\
\\
\\
2m - 2l &= -l + r \\
\\
\\
2m - 2l &= r - l
\end{align}
$$

We can then continue finding a form of the equation that will benefit our goal.

$$
\begin{align}
2m - 2l &= r - l \\
\\
\\
2(m - l) &= r - l \\
\\
\\
\dfrac{2(m - l)}{2} &= \dfrac{r - l}{2} \\
\\
\\
m - l &= \dfrac{r - l}{2} \\
\\
\\
m - l + l &= \dfrac{r - l}{2} + l \\
\\
\\
m &= l + \dfrac{r - l}{2}
\end{align}
$$

Now the final form of this equation is way less susceptible to overflows (assuming $0 \le l \le r$, which is the case for array indices in binary search, $r - l$ can never exceed $r$, so no intermediate value is larger than the inputs).

## Completing the Square

Completing the square rewrites a quadratic into a perfect square plus a constant.
This is useful for solving quadratics, deriving the quadratic formula, and converting
conic equations into standard form.

Starting from the general quadratic:

$$ax^2 + bx + c = 0$$

Move the constant to the right side:

$$
\begin{align}
ax^2 + bx + c &= 0 \\
\\
\\
ax^2 + bx + c - c &= 0 - c \\
\\
\\
ax^2 + bx &= -c
\end{align}
$$

Divide both sides by $a$ to normalize the leading coefficient ($a \neq 0$, otherwise it is not a quadratic):

$$
\begin{align}
\dfrac{ax^2 + bx}{a} &= \dfrac{-c}{a} \\
\\
\\
x^2 + \dfrac{b}{a}x &= -\dfrac{c}{a}
\end{align}
$$

Add $\left(\dfrac{b}{2a}\right)^2$ to both sides to complete the square on the left:

$$
\begin{align}
x^2 + \dfrac{b}{a}x + \left(\dfrac{b}{2a}\right)^2 &= -\dfrac{c}{a} + \left(\dfrac{b}{2a}\right)^2 \\
\\
\\
\left(x + \dfrac{b}{2a}\right)^2 &= \dfrac{b^2}{4a^2} - \dfrac{c}{a} \\
\\
\\
\left(x + \dfrac{b}{2a}\right)^2 &= \dfrac{b^2}{4a^2} - \dfrac{4ac}{4a^2} \\
\\
\\
\left(x + \dfrac{b}{2a}\right)^2 &= \dfrac{b^2 - 4ac}{4a^2}
\end{align}
$$

Take the square root of both sides (remember the $\pm$):

$$
\begin{align}
\sqrt{\left(x + \dfrac{b}{2a}\right)^2} &= \pm\sqrt{\dfrac{b^2 - 4ac}{4a^2}} \\
\\
\\
x + \dfrac{b}{2a} &= \dfrac{\pm\sqrt{b^2 - 4ac}}{2a} \\
\\
\\
x + \dfrac{b}{2a} - \dfrac{b}{2a} &= \dfrac{\pm\sqrt{b^2 - 4ac}}{2a} - \dfrac{b}{2a} \\
\\
\\
x &= \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a}
\end{align}
$$

This is the quadratic formula.

## Variable Isolation via Change of Subject

Sometimes the goal is not to find a numeric value, but to rewrite an equation so that
a different variable is expressed in terms of the others. This is called **changing the subject**.

For example, the formula for kinetic energy is:

$$E = \dfrac{1}{2}mv^2$$

To make $v$ the subject, first eliminate the fraction by multiplying both sides by $2$:

$$
\begin{align}
E &= \dfrac{1}{2}mv^2 \\
\\
\\
E \cdot 2 &= \dfrac{1}{2}mv^2 \cdot 2 \\
\\
\\
2E &= mv^2
\end{align}
$$

Then isolate $v^2$ by dividing both sides by $m$:

$$
\begin{align}
2E &= mv^2 \\
\\
\\
\dfrac{2E}{m} &= \dfrac{mv^2}{m} \\
\\
\\
\dfrac{2E}{m} &= v^2
\end{align}
$$

Finally, take the square root of both sides (normally this needs a $\pm$, but we only keep the positive root here because $v$ is a speed, which cannot be negative):

$$
\begin{align}
v^2 &= \dfrac{2E}{m} \\
\\
\\
\sqrt{v^2} &= \sqrt{\dfrac{2E}{m}} \\
\\
\\
v &= \sqrt{\dfrac{2E}{m}}
\end{align}
$$

This technique is especially important in physics and engineering, where the
same formula is reused in many different configurations.

## Introducing a Substitution to Simplify Structure

When an equation has a repeated or complex sub-expression, substituting it with
a single variable can make the structure far easier to work with.

For example:

$$\left(\dfrac{x-1}{x+1}\right)^2 + 3\left(\dfrac{x-1}{x+1}\right) - 4 = 0$$

Let $u = \dfrac{x-1}{x+1}$, so the equation becomes:

$$
\begin{align}
u^2 + 3u - 4 &= 0 \\
\\
\\
(u + 4)(u - 1) &= 0
\end{align}
$$

Therefore $u = -4$ or $u = 1$. Now substitute back and solve each case.

**Case 1:** $u = 1$

$$
\begin{align}
\dfrac{x - 1}{x + 1} &= 1 \\
\\
\\
x - 1 &= 1 \cdot (x + 1) \\
\\
\\
x - 1 &= x + 1 \\
\\
\\
-1 &= 1
\end{align}
$$

This is a contradiction, so there is no solution in this case.

**Case 2:** $u = -4$

$$
\begin{align}
\dfrac{x - 1}{x + 1} &= -4 \\
\\
\\
x - 1 &= -4(x + 1) \\
\\
\\
x - 1 &= -4x - 4 \\
\\
\\
x + 4x &= -4 + 1 \\
\\
\\
5x &= -3 \\
\\
\\
\dfrac{5x}{5} &= \dfrac{-3}{5} \\
\\
\\
x &= -\dfrac{3}{5}
\end{align}
$$

This technique, sometimes called **auxiliary substitution**, is especially powerful
with trigonometric, exponential, or nested expressions.
