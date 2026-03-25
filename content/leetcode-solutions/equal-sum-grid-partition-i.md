---
date: '2026-03-25T09:56:32+02:00'
draft: false
title: 'Equal Sum Grid Partition I'
tags: ['Matrix', 'Prefix Sum', 'Array', 'Go']
leetcodeProblemSlug: 'equal-sum-grid-partition-i'
leetcodeSolutionLink: 'https://leetcode.com/problems/equal-sum-grid-partition-i/solutions/7690535/golang-clean-code-idiomatic-go-simple-so-oo13'
---

# Intuition

A valid partition exists if a cut can divide the grid such that both resulting sections have equal sums. By converting rows or columns into their individual sums, the 2D problem reduces to finding a partition point in a 1D array where the left sum equals the right sum.

# Approach

The solution checks both horizontal and vertical partitions:

- **Calculate row sums**: Sum each row to create an array representing potential horizontal cuts.

- **Create** `canPartitionArray` helper function to check whether we can partition a 1D array.

- **Check row partition**: Use `canPartitionArray` to find if any cut point divides the row sums equally.

- **Calculate column sums**: Sum each column to create an array representing potential vertical cuts.

- **Check column partition**: Use `canPartitionArray` to find if any cut point divides the column sums equally.

- **Return** `true` if either partition works.


# Complexity

- Time complexity: $O(M × N)$

    - Computing row sums is $O(M × N)$. Computing column sums is $O(M × N)$. Checking partitions is $O(M) + O(N)$. Total: $O(M × N)$.

- Space complexity: $O(M+N)$

    - Storage for rowSum ($O(M)$) and colSum ($O(N)$), plus prefix arrays of the same sizes. Overall: $O(M + N)$.


# Code

```go {linenos=inline}
func canPartitionGrid(grid [][]int) bool {
	rowSum := make([]int, len(grid))
	for y, row := range grid {
		sum := 0
		for _, num := range row {
			sum += num
		}
		rowSum[y] = sum
	}

	if canPartitionArray(rowSum) {
		return true
	}

	colSum := make([]int, len(grid[0]))
	for x := range grid[0] {
		sum := 0
		for y := range grid {
			sum += grid[y][x]
		}
		colSum[x] = sum
	}

	return canPartitionArray(colSum)
}

func canPartitionArray(nums []int) bool {
	prefix := make([]int, len(nums))
	prefix[0] = nums[0]
	for i := 1; i < len(nums); i++ {
		prefix[i] = prefix[i-1] + nums[i]
	}

	for i := range prefix {
		if prefix[i] == prefix[len(prefix)-1]-prefix[i] {
			return true
		}
	}

	return false
}
```
