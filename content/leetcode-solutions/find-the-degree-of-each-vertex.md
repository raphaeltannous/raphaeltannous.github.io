---
date: '2026-04-12T18:47:16+03:00'
draft: false
title: 'Find the Degree of Each Vertex'
tags: ['Matrix', 'Graph Theory', 'Go']
leetcodeProblemSlug: 'find-the-degree-of-each-vertex'
leetcodeSolutionLink: 'https://leetcode.com/problems/find-the-degree-of-each-vertex/solutions/7881075/idiomatic-go-detailed-solution-clean-cod-l11v'
---

# Intuition

The degree of a vertex is the number of edges connected to it, and `matrix[i][j]` is either `1` for connected or `0` for not connected.

We are tasked to return the number of edges connected to `i`, and the sum of `matrix[i]` is the number of edges connected to `i`.


# Approach

- **Initialize** `result` of type `[]int`.
- **Loop** (`i`) over `matrix`:
    - **Loop** (`j`) over `matrix[i]`:
        - **Add** `matrix[i][j]` to `result[i]`.
- **Return** `result`.

# Complexity

- Time complexity: $$O(N^2)$$.
    - We visit every cell once.

- Space complexity: $$O(N)$$.
    - `result` contains $$N$$ elements.

# Code

```go {linenos=inline}
func findDegrees(matrix [][]int) []int {
	result := make([]int, len(matrix))

	for i := range matrix {
		for j := range matrix[i] {
			result[i] += matrix[i][j]
		}
	}

	return result
}
```
