---
title: "Maximise Water"
author: "strexicious"
date: 2022-10-30T11:38:52+01:00
tags: ["array", "maximise", "leet code", "rectangles", "area", "implicit",
	"tree search"]
readingTime: true
draft: true
---

Problem description at: [Leet Code - 11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/)

---

The problem essentially gives us rectangles and asks to find the one with the
maximum area. The main challenge lies in doing so efficiently, as is typical of
algorithmic problems.

In this problem the rectangles are given to us in an implicit manner. We are
given an array where we can consider each pair of element (assume ith and jth)
to be the two vertical sides. Then with the `shorter length` of these two sides
and the `length i - j` we can form our rectangle. We can see basically from the
given input there exist **quadratic** number of rectangles. In this instance
it means in the worse case we have about 10 billion rectangles.

So we have to be smart to not explicitly traverse the space of all possible
rectangles. I will explain my two failed attempts, an elegant solution
(spoilers, it's a linear algorithm, so no `n log n` or sorting tricks), and then
a few words on demystifying the underlying mathematical intuition.

## Failed: incremental

1. So, we have to visit the rectangles (smartly), which one do we visit first?  
Given two indices it is easy to construct a rectangle here, so I will explain
it in terms of indices. Initially we pick index `l = 0` as a left side, and
`i = 1` as our right side.

2. Now how do we proceed in a fashion that guarentees us linear time?  
My idea was to increment `i` 1 by 1 and regard all sides as right sides.
However when we are done with some `i` as right side, if its length was greater
than current left side `l`'s, then it could be a potential left side for future
explored rectangles.

3. For each `i`, how do we consider different potential left sides?  
The potential left sides are explored in a FIFO fashion. The idea was that when
by using the next candidate we can gain more (or same) area than current max,
we would no longer need to deal with the current or any previous left side.

```python
def maxArea(height: List[int]) -> int:
	max_a = 0
	l = 0
	q = []
	for i, h in enumerate(height[1:], start=1):
		while q and (q[0]-l)*height[l] <= (min(h, height[q[0]])-height[l])*(i-q[0]):
			l = q.pop(0)

		max_a = max(max_a, (i-l)*min(h, height[l]))

		if h - height[l] >= i - l:
			q.append(i)

	return max_a
```

But this algorithm did not work. For example the input `[1,0,3,3,0,1]` gives us
a wrong answer of `3`. But if we look at it by using the sides `l = 0` and
`i = 5` we gain the maximum area of `5`. The problem is when this algorithm
considered `l = 2`, it no longer could go back. It also didn't let us skip
candidates. See what happens with the input `[2,3,4,5,18,17,6]`; it gets stuck
by adding the 4th element as potential side and not being able to use the 5th.
A possible for this particular case is to change the predicate to `h - height[l] > i - l`,
but then see what happens with the input `[1,2,3,4,5,6,7,8,9,10]`.

## Failed: divide and conquer

After spending a few while with the previous approach, I considered a passing by
idea. What if instead of making our invariant "we have correct max are for the
subarray 0..i", we consider arbitrary subarray. We would want to do that so then
we could split our array into two, find max area in those two and finally merge
the solutions. However I quickly I realised the a clean disjoint separation
would leave some rectangles to be explored in the merge phase, and we would end
up exploring all quadratic number of rectangles. And just to not let it slide,
I also tried to look for a optimal substructure to use dynamic programming, but
I abandoned it quickly as well.

## Conclusion and reveal

To not end up weeks being stuck at this problem, I looked the [solution](https://leetcode.com/problems/container-with-most-water/discuss/6100/Simple-and-clear-proofexplanation) up.

You start with the widest rectangle, formed up of the left side `l = 0` and
right side `i = n-1`, then at each step you change the shorter of both sides in
hope that it will increase in length, and so maybe in area. When I saw this
solution, it didn't sit right with me. I found it to be a weird combination of
both my attempts.

Let's look again at the core of this solution:

> Any narrower rectangle must be have taller sides to be able to have more area.[^1]

Aha! My invariant was correct! Or, at least my instincts were. So the main
problem with my algorithm was how I was trying to explore the rectangles space.
Which I tried to change in my second attempt, but had no clear idea.

## Clearing up

At first glance it was obvious that the water containers are rectangles. At
another glance it was obvious the rectables overlap. But what is this
**structure** in their overlapping which was exploited here?

Algorithmically, from our input array we can see that
the generated rectangles space forms a tree-like DAG[^2]. The nodes of the DAG have
a relation based on overlapping[^3]. A descendant node would overlap with their
ancestors. And so at the top, the root node, would be the widest rectangle, made
up of the vertical sides at `0` and `n`.

## Alternative considerations

geometrical vs algebraic etc.

[^1]: A similar notion is: to squeeze a rectangle, you must elongate it to preserve it's area.

[^2]: explain going downards but like they get merged in-between etc.

[^3]: Here the overlapping is restricted to the bottom side of all rectangles,
which sits on the x-axis so to say.
