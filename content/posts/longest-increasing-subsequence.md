---
title: Longest Increasing Subsequence
date: 2025-12-20T06:50:17.530Z
tags:
  - algorithms
  - leetcode
---

## Problem statement

Given a list of strings, return the length of the longest [lexicographically] increasing subsequence (LIS).
A **subsequence** is a a sequence derived from the original by deleting zero or more elements without changing the relative order of the remaining items.

Example:
* Input: `["A1", "B2", "B1", "C3", "C2"]`
* Output: `3`
* Explanation: The longest subsequence in which each subsequent element is strictly greater (lexicographically) then the element before it is of length 3, e.g. `["A1", "B2", "C3"]`

## Solution

The first method is brute-force. The idea is that we generate all possible subsequences and return the legnth of the largest one. 

As for generating all possible subsequences, we can consider that for any element in the input list, we need to decide if it will be kept or skipped in the subsequence. 

Formalizing that thought, we arrive at a recursive Depth-First Search (DFS) algorithm.

### DFS Algorithm

```python
def lengthOfLIS_dfs(codes):
    len_lis = 0
    seq = []

    def dfs(i):
        nonlocal len_lis
        if  i == len(codes):
            return 0
        # keep
        if len(seq) == 0 or codes[i] > seq[-1]:
            seq.append(codes[i])
            len_lis = max(len_lis, len(seq))
            dfs(i+1)
            seq.pop()
        # skip
        dfs(i+1)
    
    dfs(0)
    return len_lis
```

We maintain a variable `len_lis` to keep track of the length of the longest increasing subsequence we've found so far. Notice that in Python this variable has to be declared `nonlocal` at the start of the `dfs` function so that it can continue to reference the outer scope. 

Inside the `dfs` function, we have a base condition to stop once we reach the end of the input list. 

And then we consider keeping the current element if it's greater than the last element in the subsequence. We do this by appending it to the current subsequence and continuing the recursion down that branch of the decision tree. Once we've explored that branch fully, we'll return back to the current callstack and pop off the current element from the subsequence.

The we can continue with the 'skip' branch in the same way, except we don't need to modify the subsequence list. 

Runtime for this is exponential (O(2^n)) because at each element we make a binary choice (to keep or skip) and this results in a decision tree of height n. 
<hr>

To improve the runtime, we can consider solving a subproblem: what's the length of the longest increasing subsequence (LIS) for a smaller list.

This gets us to Dynamic-Programming (DP).

### DP Algorithm

We can formalize the subproblem as follows:

```
dp[i] = length of LIS ending at i
```

Or more explicitly:

```
dp[i] = 1 + max(dp[j] for j in 1..i-1 where codes[j] < codes[i])
```

That brings us to the following code:

```python
def lengthOfLIS_dp(codes):
    dp = [1] * len(codes)
    for i in range(1, len(codes)):
        for j in range(i):
            if codes[j] < codes[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)
```

Runtime for this algorithm is O(n^2) because of the double for-loop. 
<hr>

### Optimal Algorithm

The optimal solution requires a different line of thinking. Instead of asking "what's the best subsequence ending at i" we need to ask "what's the smallest possible tail for subsequences of each length".

That means, for 2 subsequences of equal length, we prefer the one with the smaller element at the end as that subsequence is more easily extendable.

So we can maintain an array `tails` as follows:

```
tails[k] = smallest possible tail of an increasing subsequence of length k+1
```

Then, for each new element, we find where it fits in tails (using binary search since tails is sorted) and then either replace an existing element in tails or extend tails.

After iterating through the input list in this way, `tails` contains the smallest possible tail for subsequences of each length, meaning that the length of tails is the length of the longest increasing subsequence.

This leads us to the following code:

```python
from bisect import bisect_left
def lengthOfLIS(codes):
    tails = []
    for code in codes:
        # find position where 'code' should go
        idx = bisect_left(tails, code)
        if idx == len(tails):
            tails.append(code)
        else:
            tails[idx] = code
    return len(tails)
```

Runtime for this algorithm is O(n log n).

#### Example Walkthrough

Input:
```
["A1", "B2", "B1", "C3", "C2"]
```

Output:
```
tails = []
A1 → [A1]
B2 → [A1, B2]
B1 → [A1, B1]   # replaces B2
C3 → [A1, B1, C3]
C2 → [A1, B1, C2]  # replaces C3
```

Result: `3`

<hr>

Check out all the code [here](https://github.com/salmanfs815/neetcode/blob/main/longest_increasing_subsequence.py).
