---
date: '2026-03-29T14:19:22+03:00'
draft: false
title: 'Check if Strings Can Be Made Equal With Operations II'
tags: ['String', 'Array', 'Hash Table', 'Go']
leetcodeProblemSlug: 'check-if-strings-can-be-made-equal-with-operations-ii'
leetcodeSolutionLink: 'https://leetcode.com/problems/check-if-strings-can-be-made-equal-with-operations-ii/solutions/7713923/idiomatic-go-detailed-solution-o1-space-ya8yj'
---

# Intuition

We are asked to see if the 2 strings can be equal. The description states that we can swap 2 indices `i` and `j` if `i < j`, and the difference between `i` and `j` is **even**.

Since the difference must be even for a swap to be valid, even and odd characters will never be swaped with one another since if you subtract an odd number from an even number the result will always be odd, `3 - 2 = 1`.

The result parity of `odd - odd = even`, and `even - even = even`.

That means each string is split between even and odd characters.

# Approach

We can count the frequencies of the characters of the strings in 2 hashmaps (even and odd). If the index is even, we will increase `s1[i]` and decrease `s2[i]` from the even hash map, and wel will do the same thing for the odd indices.

Then we can loop over the even, and odd frequencies, and see if any frequency is not equal to zero to return `false`.

However, since we are dealing with the english lowercase letters, we are limited to `26` characters for even indices, and `26` characters for odd indices. Then, we can create an array of size `52`, and the first `26` numbers from the array will be for even frequencies, and the rest for odd frequencies.

So we need to:

* **Create** `parity` array of size `52`.
* **Loop** (`i`) from `0` until `len(s1)-1`:
    * **Set** `offset` to `0` (for even indices).
    * **If** `i` is odd:
        * **Set** `offset` to `26`.
    * **Increase** `parity[int(s1[i]-'a')+offset]` by one.
    * **Decrease** `parity[int(s2[i]-'a')+offset]` by one.
* **Loop** (`i`) over `parity`:
    * **If** `parity[i]` is not `0`:
        * **Return** `false`.
* **Return** `true` otherwise. (all parity numbers are `0`).

# Complexity

- Time complexity: $O(N)$
    - We visited each character in both strings once: $O(N)$.
    - We looped over parity: $O(52) \rightarrow O(1)$.

- Space complexity: $O(1)$
    - We created the parity array of size `52`: $O(52) \rightarrow O(1)$.
    - We create the offset variable: $O(1)$.

# Code
```go
func checkStrings(s1 string, s2 string) bool {
	parity := [52]int{}

	for i := range s1 {
		offset := 0
		if i&1 != 0 {
			offset = 26
		}

		parity[int(s1[i]-'a')+offset]++
		parity[int(s2[i]-'a')+offset]--
	}

	for _, freq := range parity {
		if freq != 0 {
			return false
		}
	}

	return true
}
```
