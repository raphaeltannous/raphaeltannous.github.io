---
date: '2026-04-12T19:19:19+03:00'
draft: false
title: 'Angles of a Triangle'
tags: ['Mathematics', 'Geometry', 'Sorting', 'Go']
leetcodeProblemSlug: 'angles-of-a-triangle'
leetcodeSolutionLink: 'https://leetcode.com/problems/angles-of-a-triangle/solutions/7881217/idiomatic-go-detailed-solution-geometry-qf25r'
---

# Intuition

We are giving 3 sides of a supposed triangle and we want to see if we can form a triangle, and return the angles in degrees.

To create a triangle from these 3 sides, we want the sum of any two sides to be greater than the third side (Triangle Inequality Theorem). If this condition is not met that means we cannot create a triangle from these 3 sides.

> Wikipedia article on the theorem: https://en.wikipedia.org/wiki/Triangle_inequality

For sides `a`, `b`, `c` to form a triangle, **all three** must hold:

1. $a + b > c$
2. $a + c > b$
3. $b + c > a$

We can further and only check the condition $a + b > c$ if we sort the sides.

> **Why  sorting reduces it to one check?**
>
> After sorting so that $a ≤ b ≤ c$ (`c` is the largest). Can conditions 2 and 3 ever fail?
>
> Since `c` is the largest $a + c > b$ is automatically satisfied. Same goes for $b + c > a$.

To get the angles after we are sure that we can create the triangle, we can use law of cosines:

$$
\begin{aligned}
a^{2} &= b^{2} + c^{2} - 2bc\cos(A)\\[4pt]
b^{2} &= a^{2} + c^{2} - 2ac\cos(B)\\[4pt]
c^{2} &= a^{2} + b^{2} - 2ab\cos(C)
\end{aligned}
$$

Gives us:

$$
\begin{aligned}
A &= \arccos\!\left(\frac{b^{2} + c^{2} - a^{2}}{2bc}\right) \\[6pt]
B &= \arccos\!\left(\frac{a^{2} + c^{2} - b^{2}}{2ac}\right) \\[6pt]
C &= \arccos\!\left(\frac{a^{2} + b^{2} - c^{2}}{2ab}\right)
\end{aligned}
$$

> Wikipedia article on the law: https://en.wikipedia.org/wiki/Law_of_cosines

# Approach

- **Sort** `sides`.
- **If** `a+b <= c`.
    - **Return** `[]`.
- **Return** the angles using the law of cosines.

# Complexity

- Time complexity: $O(1)$.

- Space complexity: $O(1)$.

# Code

```go {linenos=inline}
import (
	"math"
	"slices"
)

func internalAngles(sides []int) []float64 {
	slices.Sort(sides)

	a, b, c := sides[0], sides[1], sides[2]

	if a+b <= c {
		return []float64{}
	}

	// We can form a triangle
	a2, b2, c2 := float64(a*a), float64(b*b), float64(c*c)

	result := []float64{
		math.Acos((b2+c2-a2)/float64(2*b*c)) * 180 / math.Pi,
		math.Acos((a2+c2-b2)/float64(2*a*c)) * 180 / math.Pi,
		math.Acos((a2+b2-c2)/float64(2*a*b)) * 180 / math.Pi,
	}

	return result
}
```
