---
date: '2026-03-28T23:03:34+02:00'
draft: false
title: 'Minimum XOR Path in a Grid'
tags: ['Dynamic Programming', 'Array', 'Bit Manipulation', 'Matrix', 'Word-level Parallelism', 'Rolling Array', 'Go']
leetcodeProblemSlug: 'minimum-xor-path-in-a-grid'
leetcodeSolutionLink: 'https://leetcode.com/problems/minimum-xor-path-in-a-grid/solutions/7711791/idiomatic-go-dynamic-programming-rolling-qjy6'
---

# Intuition

This problem is the third question in the biweekly LeetCode contest 179, and I was unable to solve it. This is a Grid Path Dynamic Programming problem, but with a twist that breaks the classical Dynamic Programming approach, because the cost of the function is **XOR**, not addition/subtraction.

During the contest, I “solved” (more like attempted) the problem in 3 ways.

**Top-down/Bottom-up Dynamic Programming** storing a single value per cell; however, since we are dealing with XOR, storing a single value does not work.

For example: If two paths reach `(x-1, y)` with XOR values `3` and `7`, and `grid[i][j] = 5`:

- `3 XOR 5 = 6`
- `7 XOR 5 = 2`

**The path with the larger XOR (7) produced the smaller result**. The minimum XOR at a cell does not guarantee the minimum XOR at the next cell. (Optimal substructure is broken).

**Backtracking/Brute Force**

Correct idea, but the number of paths is exponential. Too slow.

**`dp[y][x][]int{}` Out of Memory**

This was actually the right direction, but I was not able to optimize it during the contest.

# Approach: Storing All XOR Values

The backtracking approach tracks for each cell all the possible XOR values it can have, and we can leverage this thinking by using `dp[y][x][]int{}`, but that will result in an **Out of Memory runtime error**. What is making the `[]int{}` big, and why is it causing an **Out of Memory** runtime error?

We were storing too many values in `[]int{}`, and some of them were duplicates, but XORing 2 equal numbers with another number will result in the same value, which means we do not actually care about duplicate values; which means we can use a hash map (set) to make the memory manageable.

So we need to:

> `R` is the length of `grid`, and `C` is the length of `grid[0]`.

* **Create** the array `dp` of type `[][]map[int]bool`.
* **Set** `dp[0][0][grid[0][0]]` to `true`.
* **Loop** (`i`) over grid:
  * **Loop** (`j`) over grid[i]:
    * **If** `i` is greater than `0`:
      * **For** each number in `dp[i-1][j]`, set:
        * `dp[i][j][grid[i][j]^previous] = true`, where previous is `dp[i-1][j][x]`.
    * **If** `j` is greater than 0:
      * **For** each number in `dp[i][j-1]`, set:
        * `dp[i][j][grid[i][j]^previous] = true`, where previous is `dp[i-1][j][x]`.
* **Return** the minimum of `dp[R-1][C-1]`.

## Implementation

```go
func minCost(grid [][]int) int {
	R, C := len(grid), len(grid[0])
	dp := make([][]map[int]bool, R)
	for i := range grid {
		dp[i] = make([]map[int]bool, C)
		for j := range C {
			dp[i][j] = make(map[int]bool)
		}
	}
	dp[0][0] = map[int]bool{grid[0][0]: true}

	for i := range grid {
		for j := range grid[i] {
			if i > 0 {
				for previous, _ := range dp[i-1][j] {
					dp[i][j][grid[i][j]^previous] = true
				}
			}

			if j > 0 {
				for previous, _ := range dp[i][j-1] {
					dp[i][j][grid[i][j]^previous] = true
				}
			}
		}
	}

	result := math.MaxInt
	for res, _ := range dp[R-1][C-1] {
		result = min(result, res)
	}

	return result
}
```

## Optimization 1: Using a fixed array of size 1024

Since grid values are bounded by the constraint: `0 <= grid[i][j] <= 1023`, and since `1023` is a 10-bit number. What does the XOR of 10-bit numbers always produce?

Correct, 10-bit numbers.

We can then replace the HashMap with a boolean array, where `array[i]` indicates whether `i` is a valid number to XOR with.

So we need to:

* **Create** the array `dp` of type `[][]map[int]bool`.
* **Set** `dp[0][0][grid[0][0]]` to `true`.
* **Loop** (`i`) over `grid`:
  * **Loop** (`j`) over `grid[i]`:
    * **If** `i` is greater than `0`:
      * **For** each number in `dp[i-1][j]`, if `true` set:
        * `dp[i][j][grid[i][j]^previous] = true`, where `previous` is `dp[i-1][j][x]`.
    * **If** `j` is greater than `0`:
      * **For** each number in `dp[i][j-1]`, if `true` set:
        * `dp[i][j][grid[i][j]^previous] = true`, where `previous` is `dp[i-1][j][x]`.
* **Return** the minimum of `dp[R-1][C-1]`.

### Implementation

```go
func minCost(grid [][]int) int {
	R, C := len(grid), len(grid[0])
	dp := make([][][1024]bool, R)
	for i := range grid {
		dp[i] = make([][1024]bool, C)
	}
	dp[0][0][grid[0][0]] = true

	for i := range grid {
		for j := range grid[i] {
			if i > 0 {
				for previous, ok := range dp[i-1][j] {
					if ok {
						dp[i][j][grid[i][j]^previous] = true
					}
				}
			}

			if j > 0 {
				for previous, ok := range dp[i][j-1] {
					if ok {
						dp[i][j][grid[i][j]^previous] = true
					}
				}
			}
		}
	}

	result := math.MaxInt
	for num, ok := range dp[R-1][C-1] {
		if ok {
			result = min(result, num)
		}
	}

	return result
}
```

## Optimization 2: Rolling Array

Each cell in the grid depends on the cell above it and the cell to its left, if present. That means if we are at row `2`, we do not need or care about row `0` from now on.

We can then save space by using only 2 rows, not R rows.

So we need to:

* **Create** `prev` array of size `C` with a type of `[][1024]bool`.
* **Initialize** `prev` with `grid[0]` values since row `0` does not depend on a previous row.
  * **Set** `prev[0][grid[0][0]]` to `true`.
  * **Loop** (`i`) starting from `1 until C-1`:
    * **Loop** over `prev[i-1]`, if `true` set:
      * `prev[i][previous^grid[0][i]` to `true`, where `previous` is `prev[i-1][x]`.
* **Create** `curr` array of size `C` with a type of `[][1024]bool`.
  * **Loop** (`i`) starting from `1 until R-1`:
    * **Loop** (`j`) over `grid[i]`:
      * **Loop** over `prev[j]`, if `true` set:
        * `curr[j][previous^grid[i][j]` to `true`, where `previous` is `prev[j][x]`.
      * **If** j is greater than 0:
        * **Loop** over curr[j-1], if true set:
          * `curr[j][previous^grid[i][j]` to `true`, where `previous` is `curr[j-1][x]`.
    * **Set** `prev` to `curr`.
    * **Reassign** `curr`.
* **Return** the minimum of `prev[C-1]`.

### Implementation

```go
func minCost(grid [][]int) int {
	R, C := len(grid), len(grid[0])

	// Initializing row 0 of the grid
	// as prev and updating the XOR
	// values since it does not need
	// a top row.
	prev := make([][1024]bool, C)
	prev[0][grid[0][0]] = true
	for i := 1; i < C; i++ {
		for previous, ok := range prev[i-1] {
			if ok {
				prev[i][previous^grid[0][i]] = true
			}
		}
	}

	curr := make([][1024]bool, C)
	for i := 1; i < R; i++ {
		for j := 0; j < C; j++ {
			for previous, ok := range prev[j] {
				if ok {
					curr[j][previous^grid[i][j]] = true
				}
			}

			if j > 0 {
				for previous, ok := range curr[j-1] {
					if ok {
						curr[j][previous^grid[i][j]] = true
					}
				}
			}
		}

		prev = curr
		curr = make([][1024]bool, C)
	}

	result := math.MaxInt
	for num, ok := range prev[C-1] {
		if ok {
			result = min(result, num)
		}
	}

	return result
}
```

## Optimization 3: Word-level Parallelism

With `[1024]bool`, each element is **1 byte**. To find set values, you loop `1024` times, touching `1024` separate bytes.

With `[16]uint64`, 64 booleans are **packed into one 64-bit integer**. The key speedup is:

```txt
if word == 0 { skip all 64 values instantly }
```

A single comparison skips 64 values. That's word-level parallelism, **processing 64 bits with one CPU instruction** instead of 64 separate checks.

That means instead of `[1024]bool` we can use an array of `uint64` of size `16` covering all the numbers from `0` to `1023`.

Each unsigned number in the array will hold `64` numbers starting from `0` until `63`, then from `64` until `127`, and so on until `1023`.

If `v` is the value we want to set in the array, we can set it using:

```txt
bits[v/64] |= 1 << (v % 64)
```

And to retrieve each number, we can use a for loop with an offset and a counter with bit shifting.

```go
// Example
bits := uint64(1 << 7) // Setting the number 7 in uint64
bits |= uint64(1 << 4) // Setting the number 4 in unint64
counter := 0
for bits != 0 {
	if bits&1 != 0 {
		fmt.Println(counter) // 4 then 7
	}
	counter++
	bits >>= 1
}
```

So we need to:

* **Define** a helper function `updateNextUsingXOR(original, toUpdate, currentNumber)`.
  * **Loop** `(offset)` in `original`:
    * **Extract** `bits := original[offset]`.
    * **Initialize** `counter = 0`.
    * **While** `bits` is not zero:
      * **If** the lowest bit is set:
        * **Compute** the actual previous XOR value:
          * `previous = offset * 64 + counter`
        * **Apply** XOR with current number:
          * `newValue = previous ^ currentNumber`
        * **Set** the corresponding bit in `toUpdate`:
          * `toUpdate[newValue/64] |= 1 << (newValue % 64)`
      * **Right** shift `bits` by 1.
      * **Increment** `counter`.
* **Create** `prev` array of size `C` with a type of `[][16]uint64` (bitmask instead of boolean DP).
  * **Initialize** `prev` using the first row since it does not depend on a previous row.
    * **Set** the bit for `grid[0][0]` in `prev[0]`.
    * **Loop** `(i)` from `1` to `C-1`:
      * **Call** `updateNextUsingXOR(prev[i-1], prev[i], grid[0][i])` to propagate XOR values.
* **Create** `curr` array of size `C` with a type of `[][16]uint64`.
  * **Loop** `(i)` from `1` to `R-1`:
    * **Loop** `(j)` over columns:
      * **From** top (prev row):
        * **Call** `updateNextUsingXOR(prev[j], curr[j], grid[i][j])`
      * **From** left (current row):
        * **If** `j > 0`:
          * Call `updateNextUsingXOR(curr[j-1], curr[j], grid[i][j])`
    * **After** finishing the row:
      * **Assign** `prev = curr`
      * **Reinitialize** `curr`.
* **Compute** the result from `prev[C-1]`:
  * **Loop** over each `offset` (block of 64 bits):
    * **Extract** all set bits.
    * **Convert** each bit position into its actual XOR value:
      * `value = offset * 64 + counter`
    * **Track** the minimum value.
* **Return** the minimum XOR value found.

### Implementation

```go
func minCost(grid [][]int) int {
	R, C := len(grid), len(grid[0])

	prev := make([][16]uint64, C)
	// You can ommit the | in |=,
	// since there's always excatly 1
	// distinct XOR value per cell,
	// because there's exactly one forced path.
	prev[0][grid[0][0]/64] |= 1 << (grid[0][0] % 64)
	for i := 1; i < C; i++ {
		updateNextUsingXOR(&prev[i-1], &prev[i], grid[0][i])
	}

	curr := make([][16]uint64, C)
	for i := 1; i < R; i++ {
		for j := 0; j < C; j++ {
			// Above
			updateNextUsingXOR(&prev[j], &curr[j], grid[i][j])

			// Left
			if j > 0 {
				updateNextUsingXOR(&curr[j-1], &curr[j], grid[i][j])
			}
		}

		prev = curr
		curr = make([][16]uint64, C)
	}

	result := math.MaxInt
	for offset, bits := range prev[C-1] {
		counter := 0
		for bits != 0 {
			if bits&1 != 0 {
				value := counter + (offset * 64)
				result = min(result, value)
			}

			bits >>= 1
			counter++
		}
	}

	return result
}

func updateNextUsingXOR(original, toUpdate *[16]uint64, currentNumber int) {
	for offset, bits := range original {
		counter := 0

		// Early stop
		// We can use bits & (1 << v) != 0
		// for v  in [1, 64], but by using
		// counter + bits != 0 we can early
		// stop if there's only one bit set
		// in bits.
		for bits != 0 {
			if bits&1 != 0 {
				previous := counter + (offset * 64)
				previous ^= currentNumber

				toUpdate[previous/64] |= 1 << (previous % 64)
			}

			bits >>= 1
			counter++
		}
	}
}
```

### Complexity

* Time Complexity: $O(M \times N)$.
  * We visit each cell once.
  * We compute a constant number of time (`1024`) in `updateNextUsingXOR`.
* Space Complexity: $O(N)$.
  * The rolling array keeps only `2` rows.
