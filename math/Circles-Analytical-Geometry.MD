# **Circles - Analytical Geometry**

A **circle** is the set of all points on a plane that are
**equidistant** from a fixed point on the plane.
The fixed point is called the center, and the distance
from the **center** to any point of the circle is
called the **radius**. 

Let the center of a circle be at the fixed point $C(h,k)$,
and let the radius be equal to $r$.

## **Radius Formula**

Used to check if a given point $P(x,y)$ is a part of a circle, if it is
the answer should be the same as the given radius.

$$\sqrt{(x-h)^2+(y-k)^2} = r$$

## **Center-Radius Form**

$$(x-h)^2+(y-k)^2 = r^2$$


If the center is $C(0,0)$ then we can denote the equation
of the circle as $x^2+y^2=r^2$.

## **General Form**

$$
x^2+y^2+Dx+Ey+F=0
$$

## **General Form to Center-Radius Form**

Group the $x$ and $y$ terms, move $F$ to the right side, then complete the square for both $x$ and $y$
(add $\left(\dfrac{D}{2}\right)^2$ and $\left(\dfrac{E}{2}\right)^2$ to both sides):

$$
\begin{align}
x^2 + Dx + y^2 + Ey &= -F \\
\left(x^2 + Dx + \dfrac{D^2}{4}\right) + \left(y^2 + Ey + \dfrac{E^2}{4}\right) &= \dfrac{D^2}{4} + \dfrac{E^2}{4} - F \\
\left(x + \dfrac{D}{2}\right)^2 + \left(y + \dfrac{E}{2}\right)^2 &= \dfrac{D^2 + E^2 - 4F}{4}
\end{align}
$$

So the center is $C\left(-\dfrac{D}{2}, -\dfrac{E}{2}\right)$ and the radius is $r = \dfrac{\sqrt{D^2 + E^2 - 4F}}{2}$.

- If $D^2 + E^2 - 4F > 0$ it is a real circle.
- If $D^2 + E^2 - 4F = 0$ it is just a single point (the center).
- If $D^2 + E^2 - 4F < 0$ there is no real circle.