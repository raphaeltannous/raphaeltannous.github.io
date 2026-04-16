---
date: '2026-04-16T15:22:39+03:00'
draft: false
title: 'Minimum Distance to the Target Element'
tags:
- Array
- Go
leetcodeProblemSlug: 'minimum-distance-to-the-target-element'
leetcodeSolutionLink: ''
---

# Intuition


The problem is asking to return `abs(i - start)` if `nums[i] == target`.
The brute force solution is to check each number starting from `0` until `len(nums)`.

What if we started from `start`[^1] and went to the right and to the left.
We can then check from the right first since `i` will be greater than `start`,
then from the left where `i` will be less that `start`.

If we find `target` we directly return `i`,
because we already have the minimum `abs(i - start)`.

# Approach

- **Loop** (`i`) over `nums`:
  - **If** `start + i < len(nums)` and `nums[start+i] == target`:
    - **Return** `i`.
  - **If** `start - i >= 0` and `nums[start-i] == target`:
    - **Return** `i`.
- **Return** `0`[^2].

# Complexity

- Time complexity: $O(N)$.

- Space complexity: $O(1)$.

# Code

```go
func getMinDistance(nums []int, target int, start int) int {
	for i := range nums {
		if start+i < len(nums) && nums[start+i] == target {
			return i
		}

		if start-i >= 0 && nums[start-i] == target {
			return i
		}
	}

	return 0
}
```

[^1]: We can start from `start` since `start` is in the bounds of `0` and `len(nums)-1`.
[^2]: When we hit this return it means the target number is not in the array, but the target is guaranteed to be present in the description.
