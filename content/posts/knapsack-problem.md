---
title: Knapsack Problem
date: 2026-05-04T06:50:17.530Z
tags:
  - algorithms
  - leetcode
---

0/1 Knapsack Problem: Given weights and profits of n items, maximize the profit without exceeding knapsack capacity. Each item can only be chosen at most once.

## Problem statment

You are given a list of items, each with a weight and a profit, along with a backpack with a specified maximum capacity. Your goal is to calculate the maximum profit you can achieve without exceeding the backpack's capacity. You must select items such that the total weight of the items is less than or equal to the backpack's capacity. You can select at most one of each item.

## Solution

The general idea is for a given item, we **either include it or skip it**, and we maximize the profit while ensuring accumulated weights don't exceed capacity.

### DFS (brute-force)

The first method I tried to implement this DFS algorithm was with the following subproblem:

```
dfs(i, currWeight, currProfit)
```

This would track the accumulated profit and weight. 
While this works, it's better to minimize our parameters, as it'll help us find optimizations.

The pseudocode for the above looks like this:

```
inputs: profit[], weight[], capacity
def dfs(i, currWeight, currProfit):
    if i == len(profit): return currProfit
    if currWeight > capacity: return -1
    skip = dfs(i + 1, currWeight, currProfit)
    take = dfs(i + 1, currWeight + weight[i], currProfit + profit[i])
    return max(skip, take)
```

And we begin the recursion with:

```
dfs(0, 0, 0)
```

We can explain the subproblem `dfs(i, currWeight, currProfit)` in terms of the weight and profit we've accumulated considering items 0 to i-1. This function call will return the max profit after considering items i to n. 

The issue with this is that we're tracking the accumulated profit thus far as a parameter even though we will be returning the profit as well. 

So, we can instead reframe the subproblem as `dfs(i, remCap)` in terms of the remaining capacity in the backpack/knapsack after having considered items 0 to i-1. This function call will return the max profit only considering items i to n, without their accumulated weights exceeding `remCap` (remaining capacity). 

The pseudocode for this DFS algorithm looks like this:

```
inputs: profit[], weight[], capacity
def dfs(i, remCap):
    if i == len(profit): return 0
    skip = dfs(i + 1, remCap)
    take = 0
    if remCap - weight[0] >= 0:
        # we can also handle exceeding capacity here instead of as a base case
        take = profit[i] + dfs(i + 1, remCap - weight[i])
    return max(skip, take)
```

In this case, we begun the recursion with:

```
dfs(0, capacity)
```

The DFS algorithm is exponential in runtime. In this case, it is O(2^n). 

However, this second way of framing the subproblem easily lends itself to caching with minimal modifications to the existing code.

### DFS with memoization

We have 2 parameters to the `dfs` function: `i` and `remCap`. Therefore, we can create a 2D array where we store solutions to subproblems in case we come across the same subproblem multiple times. This dramatically improves our runtime, bringing it down to O(n * m), where n is the number of items and m is the capacity of the backpack.

This DFS approach is a "top-down" method. We can easily convert this into a "bottom-up" Dynamic Programming (DP) algorithm.

### DP

For this DP algorithm, we initialize a 2D array `dp`, similar to the 2D array `cache` from the previous algorithm. But, instead of working *down* from the main problem, we work our way *up* from the smallest subproblem. 

While this has the same time (O(n * m)) and space (O(n * m)) complexity as DFS with memoization, it brings with it the potential for further optimization.

### Memory-Optimized DP

The way to arrive at this optimization is from observing the main computation that fills each cell in `dp`:

```
[...]
            skip = dp[i-1][c]
            take = profit[i] + dp[i-1][c - weight[i]] if c - weight[i] >= 0 else 0
            dp[i][c] = max(skip, take)
[...]
```

At any given time, we are only concerned with the i-th row in `dp` and the one above it. This means that we don't need to maintain the entire 2D array in memory, only the current and previous rows.

Our code then will look like:

```
n = len(profit)
prev = [0] + ([-1] * (capacity))
for i in range(n):
    curr = [0] + ([-1] * (capacity))
    for c in range(1, capacity + 1):
        skip = prev[c]
        take = profit[i] + prev[c - weight[i]] if c - weight[i] >= 0 else 0
        curr[c] = max(skip, take)
    prev = curr
return prev[-1]
```

While this doesn't improve the time complexity, which is still O(n * m), it brings down the space complexity to O(m), where m is the capacity of the knapsack.

<hr>

Check out all the code [here](https://github.com/salmanfs815/neetcode/blob/main/zero_one_knapsack_problem.py).

<hr>

## Variant: Unbounded Knapsack Problem

The problem statement for the Unbounded Knapsack Problem is nearly identical to the 0/1 Knapsack Problem above: given profits and weights of items and knapsack capacity, we need to maximize profit without exceeding capacity; however, any item can be chosen up to an unlimited number of times (as opposed to "at most once" in the 0/1 variation above).

The solution is also nearly identical with one key distinction: if we take the *i*th item, we don't increment `i` on the recursive call so that we can leave open the opportunity to take the *i*th item again. That's it.

Check out the code for the full solution [here](https://github.com/salmanfs815/neetcode/blob/main/unbounded_knapsack_problem.py).
