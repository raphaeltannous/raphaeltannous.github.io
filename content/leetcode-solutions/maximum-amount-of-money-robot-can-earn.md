---
date: '2026-04-02T23:17:48+03:00'
draft: false
title: 'Maximum Amount of Money Robot Can Earn'
tags: ['Dynamic Programming', 'Array', 'Matrix', 'Go']
leetcodeProblemSlug: 'maximum-amount-of-money-robot-can-earn'
leetcodeSolutionLink: 'https://leetcode.com/problems/maximum-amount-of-money-robot-can-earn/solutions/7756807/idiomatic-go-detailed-explanation-by-rap-s5vn'
---

# Intuition

The problem is a dynamic programming problem with a small twist which is that we can neutralize robbers that rob the robot if the cell is negative at most **2** times.

This will introduce an additional constraint to cell position which is the number of neutralize left to use.

> `R` is `len(coins)` and `C` is `len(coins[0])-1`. `nLeft` is the number of neutralizations left.

Starting from `(0, 0)` and going to `(R-1, C-1)`, we at each step consider two cases:

- **No neutralizing the robber:**
    - We collect `coins[y][x]` and continue moving down or right.
- **Neutralizing the robber (if `coins[y][x] < 0`):**
    - If `nLeft > 0` and `coins[y][x] < 0`, we can neutralize the robber, and "skip" current cell.

We need to consider base cases:

- `y` and `x` are out of bounds.
    - If we are out of bounds we cannot return `0` since the result might be negative. We are left with returning `math.MinInt` ($-\infin$).
- We are at the last cell.
    - If the last cell is negative and we still have left neutralizations we return `0`, otherwise we return the last cell's value.

**Can an overflow happen if we return $-\infin$?**

|Position|`dp(y+1, x)`|`dp(y, x+1)`|`max(...)` result|
|---|---|---|---|
|Middle cell|real value|real value|real value|
|Last row|`MinInt`|real value|real value|
|Last column|real value|`MinInt`|real value|
|Bottom-right|—|—|**caught by base case first**|

The table shows when we return $-\infin$ and the result of the `max` function used to get the max path value.

Notice that `max` never has `math.MinInt, math.MinInt` to result in an integer overflow if we add to it a negative number.

If we do not caught the last cell (bottom right position) an overflow will happen because the table will become:

|Position|`dp(y+1, x)`|`dp(y, x+1)`|`max(...)` result|
|---|---|---|---|
|Middle cell|real value|real value|real value|
|Last row|`MinInt`|real value|real value|
|Last column|real value|`MinInt`|real value|
|Bottom-right|`MinInt`|`MinInt`|**OVERFLOW**|

# Approach

- **Let** `R` and `C` be the number of rows and columns in `coins`.
- **Create** a 3D memoization array `memo[R][C][3]` and initialize all values to negative infinity.
- **Define** a recursive function `dp(y, x, k)`:
  - Represents the **maximum coins collectible starting at cell `(y, x)` with `k` skips remaining**.
- **If** `(y, x)` is out of bounds:
  - **Return** negative infinity.
- **If** `(y, x)` is the bottom-right cell:
  - **If** `k > 0`, **return** `max(0, coins[y][x])`.
  - **Else**, **return** `coins[y][x]`.
- **If** `memo[y][x][k]` is already computed:
  - **Return** the stored value.
- **Set** `current = coins[y][x]`.
- **Compute** the value by taking the current cell:
  - `take = current + max(dp(y+1, x, k), dp(y, x+1, k))`
- **Initialize** `result = take`.
- **If** `k > 0` **and** `current < 0`:
  - **Compute** skipping the current cell:
    - `skip = max(dp(y+1, x, k-1), dp(y, x+1, k-1))`
  - **Update** `result = max(result, skip)`.
- **Store** `result` in `memo[y][x][k]`.
- **Return** `result`.
- **Call** `dp(0, 0, 2)` and **return** the result.

# Complexity

- Time complexity: $O(M \times N)$
- Space complexity: $O(M \times N)$
    - We created the `memo` array to store the state for `[][][3]int`: $O(3(M \times N)) \rightarrow O(M \times N)$.

# Code

```go {linenos=inline}
import "math"

func maximumAmount(coins [][]int) int {
	R, C := len(coins), len(coins[0])

	memo := make([][][3]int, R)
	for y := range R {
		memo[y] = make([][3]int, C)
		for x := range C {
			for k := range 3 {
				memo[y][x][k] = math.MinInt
			}
		}
	}

	var dp func(y, x, nLeft int) int
	dp = func(y, x, nLeft int) int {
		if y > len(coins)-1 || x > len(coins[0])-1 {
			return math.MinInt // Does an overflow might happen? No.
		}

		// Handling the last cell explicitly to
		// prevent overflow, because if we
		// are at the last cell the dp function
		// will do current+dp(math.MinInt, math.MinInt),
		// which will result in an integer overflow.
		if y == len(coins)-1 && x == len(coins[0])-1 {
			if nLeft > 0 {
				return max(0, coins[y][x])
			}

			return coins[y][x]
		}

		if memo[y][x][nLeft] != math.MinInt {
			return memo[y][x][nLeft]
		}

		current := coins[y][x]

		// Having current+ outside of the max function
		// will prevent integer overflow
		// for cells out of bounds.
		v := current + max(dp(y+1, x, nLeft), dp(y, x+1, nLeft))

		if nLeft > 0 && current < 0 {
			v = max(v, dp(y+1, x, nLeft-1))
			v = max(v, dp(y, x+1, nLeft-1))
		}

		memo[y][x][nLeft] = v
		return v
	}

	return dp(0, 0, 2)
}
```
