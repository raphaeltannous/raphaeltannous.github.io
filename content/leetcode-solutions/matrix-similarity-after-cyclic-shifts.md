---
date: '2026-03-27T10:04:15+02:00'
draft: false
title: 'Matrix Similarity After Cyclic Shifts'
tags: ['Matrix', 'Mathematics', 'Array', 'Go']
leetcodeProblemSlug: 'matrix-similarity-after-cyclic-shifts'
leetcodeSolutionLink: 'https://leetcode.com/problems/matrix-similarity-after-cyclic-shifts/solutions/7701545/idiomatic-go-detailed-explanation-simple-b6if'
---

# Intuition

**Even** rows are shifted right by `k`, and **odd** rows are shifted left by `k`. At first glance, these seem like two different checks. However, the condition for a row to be unchanged after a right shift by `k` is mathematically equivalent to being unchanged after a left shift by `k` — both reduce to:

$row[j] == row[(j + k) \mod n]$

So we can apply a single uniform check across all rows without distinguishing even from odd.

# Approach

- Reduce `k` to `k modulo n` (number of columns) to handle cases where `k ≥ n`.
- For every element `mat[y][x]`, verify it equals `mat[y][(x+k) % n]`.
- If any element fails the check, immediately return `false`.
- If all elements pass, return `true`.

# Complexity

- Time complexity: $O(M \times N)$
    - We visit every element in the matrix exactly once, where $M$ is the number of rows and $N$ is the number of columns.

- Space complexity: $O(1)$
    - No extra data structures are used beyond a few variables.

# Code

```go {linenos=inline}
func areSimilar(mat [][]int, k int) bool {
	MOD := len(mat[0])
	k %= MOD
	for y, row := range mat {
		for x := range row {
			if mat[y][(x+k)%MOD] != mat[y][x] {
				return false
			}
		}
	}

	return true
}
```
