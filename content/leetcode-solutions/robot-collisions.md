---
date: '2026-04-01T22:28:56+03:00'
draft: false
title: 'Robot Collisions'
tags: ['Array', 'Stack', 'Sorting', 'Simulation', 'Go']
leetcodeProblemSlug: 'robot-collisions'
leetcodeSolutionLink: 'https://leetcode.com/problems/robot-collisions/solutions/7744818/idiomatic-go-detailed-solution-by-raphae-y7lt'
---

# Intuition

We have a bunch of robots that are moving in a horizontal line going either left (**L**), or right (**R**).

Robots are numbered from **1** until **n**.

Robot `1` is at `position[0]` with `health[0]` and `direction[0]`. Since each robot might differ in position, that means the positions array might not be ordered.

At first we need to order the arrays and keep hold the original index since the result array require us to return in the same order that the robots were given.

To do that we can create a new `struct` that will hold the `index` (original index), `position`, `health`, and `direction`.

Then we can join all the robots data into one array of type that holds the newly struct for each robot, and go on and solve the problem.

To solve the problem we can, iterate over and over and over until there's no more collision, but that will be inefficient because of the constraints.

Can we think of an efficient solution? If each robots are moving at the same time and they differ by a fixed position, the only robots that will collide are the robots that are going `L` and the one that are going `R` towards it.

```
--Robot 1--> <--Robot 2-- (Collision)
<--Robot 1-- --Robot 2--> (No Collision)
<--Robot 1-- <--Robot 2-- (No Collision)
--Robot 1--> --Robot 2--> (No Collision)
```

The only time that a collision happens is when the left robot is going to the right and the right robot is going to the left.

We can then use a `stack` to keep track of the top robot and the next robot.

**Why we can use a stack?** We can since the robots are moving at the same speed.

Then when do we update the stack? When we only have the top of the stack going right and the next robot is going left, like the first line in the diagram.

# Approach


- **Create** a `robot` struct with fields: `index`, `position`, `health`, and `direction`.
- **Create** a list `robots` of size `n`.
- **Loop** `i` from `0` to `len(positions) - 1`:
    - **Set** `dir` to `1`. (1 means going right).
    - **If** `directions[i] == 'L'`, set `dir` to `-1`.
    - **Create** a robot with:
        - `index` = `i + 1`
        - `position` = `positions[i]`
        - `health` = `healths[i]`
        - `direction` = `dir`
    - **Append** robot to `robots`.
- **Sort** `robots` by position in **increasing** order.
- **Create** an empty `stack`.
- **Loop** over each robot `r` in `robots`:
    - **If** `r.direction == 1`:
        - **Push** `r` onto the stack.
        - **Continue** to next robot.
    - **Set** `toAdd = r`.
    - **While** `stack` is not empty:
        - **Let** `top` be the last element of the stack.
        - **If** `top.direction != 1`:
            - **Break** the loop.
        - **Pop** `top` from the stack.
        - **If** `top.health > toAdd.health`:
            - **Decrease** `top.health` by `1`.
            - **Set** `toAdd = top`.
            - **Break** the loop.
        - **Else if** `top.health < toAdd.health`:
            - **Decrease** `toAdd.health` by `1`.
            - **Continue** the loop.
        - **Else** (equal health):
            - **Set** both `top.health` and `toAdd.health` to `0`.
            - **Set** `toAdd = nil`.
            - **Break** the loop.
    - **If** `toAdd` is not `nil`:
        - **Push** `toAdd` onto the stack.
- **Sort** `stack` by original index in increasing order.
- **Create** `result` array of size equal to stack length.
- Loop `i` from `0` to `len(stack) - 1`:
    - Set `result[i] = stack[i].health`.
- Return `result`.

# Complexity

- Time complexity: $O(NLog(N))$
    - Going through the input is $O(N)$.
    - Sorting the created `robots` and `stack` arrays is $O(NLog(N))$.

- Space complexity: $O(N)$
    - The required space for the `robots` and `stack` array is $O(N)$.

# Code

```go
import "slices"

type robot struct {
	index     int
	position  int
	health    int
	direction int
}

func survivedRobotsHealths(positions []int, healths []int, directions string) []int {
	robots := make([]*robot, 0, len(positions))
	for i := range positions {
		dir := 1 // Right
		if directions[i] == 'L' {
			dir = -1
		}

		r := &robot{
			index:     i + 1,
			position:  positions[i],
			health:    healths[i],
			direction: dir,
		}

		robots = append(robots, r)
	}
	slices.SortFunc(robots, func(a, b *robot) int {
		return a.position - b.position
	})

	stack := make([]*robot, 0, len(robots))
	for _, r := range robots {
		// ->
		if r.direction == 1 {
			stack = append(stack, r)
			continue
		}

		toAdd := r
		for len(stack) > 0 {
			top := stack[len(stack)-1]

			// -> <-
			if top.direction == 1 {
				stack = stack[:len(stack)-1]

				// -> greater than <-
				if top.health > toAdd.health {
					top.health -= 1
					toAdd = top
					break
				} else if top.health < toAdd.health {
					toAdd.health -= 1
				} else { // ==
					// Drop both
					toAdd.health = 0
					top.health = 0
					toAdd = nil
					break
				}
			} else { // <-
				break
			}
		}

		if toAdd != nil {
			stack = append(stack, toAdd)
		}
	}

	slices.SortFunc(stack, func(a, b *robot) int {
		return a.index - b.index
	})

	result := make([]int, len(stack))
	for i, r := range stack {
		result[i] = r.health
	}

	return result
}
```
