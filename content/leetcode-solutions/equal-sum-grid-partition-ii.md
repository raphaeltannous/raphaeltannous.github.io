---
date: '2026-03-26T15:55:28+02:00'
draft: false
title: 'Equal Sum Grid Partition II'
tags: ['Matrix', 'Prefix Sum', 'Array', 'Hash Table', 'Go']
leetcodeProblemSlug: 'equal-sum-grid-partition-ii'
leetcodeSolutionLink: 'https://leetcode.com/problems/equal-sum-grid-partition-ii/solutions/7698159/idiomatic-go-detailed-explanation-simple-ad3r'
---

# Intuition

We want to find a single horizontal or vertical line that splits the grid into two sections with equal sums, allowing us to remove at most one cell to make them equal.

For each possible cut, we track the prefix sum of the top section and derive the bottom section’s sum as total - prefix. If they differ, we need to discard exactly one cell whose value equals the difference. The catch is that the discarded cell must not disconnect its section.

Connectivity is the interesting part. A multi-row (or multi-column) section is always safe (Removing any single cell from a grid-shaped region leaves it connected). But a single-row section is just a line of cells, so removing anything other than a corner breaks it in two. This gives us a clean rule: if the section has more than one row, any cell with the right value works; if it’s exactly one row, only the two corner cells are valid candidates.

Rather than handling horizontal and vertical cuts separately, we rotate the grid 90 degrees and run the same horizontal-cut logic twice.

# Approach

> `R` is the length of `grid` and `C` is the length of `grid[0]`.

- Compute the total sum of the grid.
- Try a horizontal cut using `canPartition()`. If the grid has only one column, delegate to `canPartitionOne()` since connectivity rules differ there.
- Rotate the grid (transposing rows to columns) and try again (this covers vertical cuts).
- In `canPartition()`:
  * Build a prefix sum over row sums to efficiently compute the top/bottom section sums at each cut position.
  * Maintain `topCount` and `bottomCount` frequency maps to track which values are available for discounting.
  * At each cut i (splitting rows `0..i-1` into **top**, `i..R-1` into **bottom**):
    * If `sums == equal`: return `true` **immediately**.
    * If `top > bottom`: we need to remove `wanted = prefix - (total - prefix)` from the top. If the top is only one row (`i == 1`), only the two corner cells `grid[0][0]` and `grid[0][C-1]` can be removed safely. Otherwise, any cell in the top works (grid stays connected).
    * Symmetric logic for `bottom > top`: we need to remove `total - prefix - prefix` from the bottom. If the bottom is only one row (`i == R-1`), only the two corner cells `grid[R-1][0]` and `grid[R-1][C-1]` can be removed safely. Otherwise, any cell in the bottom works (grid stays connected).
- `canPartitionOne()` handles the single-column edge case: here, removing any non-endpoint cell from a section would disconnect it, so only the global top (`grid[0][0]`), global bottom (`grid[R-1][0]`), or the two cells directly adjacent to the cut are valid candidates.

# Complexity

- Time complexity: $O(M \times N)$
    * We iterate over all cells a constant number of times (sum, rotate, prefix sums, count maps).

- Space complexity: $O(M \times N)$
    * For the rotated grid and the frequency maps.

# Code

```go
func canPartitionGrid(grid [][]int) bool {
	total := 0
	for _, row := range grid {
		for _, num := range row {
			total += num
		}
	}

	// Trying horizontal cut at first, then
	// trying vertical cut by rotating the
	// grid.
	if len(grid) != 1 && canPartition(grid, total) {
		return true
	}

	rotatedGrid := rotate(grid)
	return canPartition(rotatedGrid, total)
}

func canPartition(grid [][]int, total int) bool {
	R, C := len(grid), len(grid[0])

	if C == 1 {
		return canPartitionOne(grid, total)
	}

	// Saving count for each number
	bottomCount := make(map[int]int)
	for _, row := range grid {
		for _, num := range row {
			bottomCount[num]++
		}
	}

	// Creating Prefix Sum of the Rows
	nums := make([]int, R)
	for y, row := range grid {
		sum := 0
		for _, num := range row {
			sum += num
		}
		nums[y] = sum
	}

	topCount := make(map[int]int)
	prefix := 0
	for i := 1; i < R; i++ { // Cut begins from 1 until R-1 inclusive
		prefix += nums[i-1]

		// Decreasing number count in bottomCount
		// and increasing in topCount, as the cut
		// is going down.
		for _, num := range grid[i-1] {
			bottomCount[num]--
			topCount[num]++
		}

		if diff := total - prefix; diff == prefix { // No need for discounting a number
			return true
		} else if diff < prefix { // Discount from the top
			wanted := prefix - (total - prefix)
			if i == 1 { // We are having a cut before 1
				// We can only remove the top edges grid[0][0] and grid[0][C-1]
				if wanted == grid[0][0] || wanted == grid[0][C-1] {
					return true
				}
			} else if topCount[wanted] > 0 { // Removing from top
				return true
			}
		} else if diff > prefix { // Discount from the bottom
			wanted := total - prefix - prefix
			if i == R-1 { // We are having a cut before R-2
				// We can only remove the bottom edges grid[R-1][0] and grid[R-1][C-1]
				if wanted == grid[R-1][0] || wanted == grid[R-1][C-1] {
					return true
				}
			} else if bottomCount[wanted] > 0 { // Removing from bottom
				return true
			}
		}
	}

	return false
}

func canPartitionOne(grid [][]int, total int) bool {
	// Grid with one column
	// We can remove either the top, the bottom,
	// or the before/after the cut.
	R := len(grid)

	prefix := 0
	for y := 1; y < R; y++ {
		prefix += grid[y-1][0]

		if diff := total - prefix; diff == prefix {
			return true
		} else {
			var wanted int
			if diff < prefix {
				wanted = prefix - (total - prefix)
			} else if diff > prefix {
				wanted = total - prefix - prefix
			}

			if wanted == grid[0][0] || wanted == grid[R-1][0] {
				return true
			}

			if wanted == grid[y][0] || wanted == grid[y-1][0] {
				return true
			}
		}
	}

	return false
}

func rotate(grid [][]int) [][]int {
	R, C := len(grid), len(grid[0])

	result := make([][]int, C)
	for y := range result {
		result[y] = make([]int, R)
	}

	for y, row := range grid {
		for x, num := range row {
			result[x][R-1-y] = num
		}
	}

	return result
}
```
