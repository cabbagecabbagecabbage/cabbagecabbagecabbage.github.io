---
title: "Problem Analysis 4"
date: 2021-12-31 12:00:00 +0000
categories: [Academic]
tags: [competitive-programming, problem-analysis-series]
math: true
---

# [Modulo Sum](https://codeforces.com/problemset/problem/577/B)
Codeforces Round #319 Div.2 B​

## Problem Statement

### Task Description

You are given a sequence of numbers $a_1, a_2, \dots, a_n$ and a number $m$.

Check if it is possible to choose a non-empty subsequence $a_i \dots a_j$ such that the sum of the numbers in this subsequence is divisible by $m$.


### Input

The first line contains two numbers, $n$ and $m$ $(1 \leq n \leq 10^6, 2 \leq m \leq 10^3)$ - the size of the original sequence and the number such that [the] sum should be divisible by it.

The second line contains $n$ integers $a_1, a_2, \dots, a_n (0 \leq a_i \leq 10^9)$.


### Output

In a single line print "YES" (without the quotes) if there exists the sought subsequence, or "NO" (without the quotes) if such subsequence doesn't exist.


## Analysis

### Part 1

When we are tasked to choose some subsequence of a sequence such that it satisfies some condition, the technique of Dynamic Programming comes to mind (e.g. [Longest increasing subsequence - Wikipedia](https://en.wikipedia.org/wiki/Longest_increasing_subsequence)). The general idea of a subsequence-related Dynamic Programming solution is to iterate through the elements and update the current state based on the addition of the current element onto the previous state, using additional dimensions as needed (depending on the conditions of the problem).

Let's refer to the sum of the elements in a subsequence as its **value**.
​
Let's call a subsequence **valid** if its value is divisible by $m$.

Let's start off by defining our state. If we let $DP[i]$ store whether it is possible to choose a valid subsequence from the first $i$ elements, we'll find it difficult to transition effectively since we need to know the value of the subsequence modulo $m$ (see [Modular arithmetic - Wikipedia](https://en.wikipedia.org/wiki/Modular_arithmetic); a basic understanding of modular arithmetic is recommended for understanding the rest of the analysis).

For example, if a subsequence has a value of 6, the element to be appended is 4 and $m$ happens to be 10, then the addition of the element would combine to satisfy to conditions of a valid subsequence; notice that we needed to know that the subsequence had value 6.

Therefore, we need an additional dimension in our state to keep track of whether it is possible to obtain a subsequence that sums to $j$ modulo $m$. We now set our state as $DP[i][j]$ stores whether it is possible to choose a subsequence that sums to $j$ using the first $i$ elements.

As we add an element to an existing subsequence, $a_i$ is added to the value. To obtain a sum of exactly $j$ modulo $m$, the state we transition from must have a value of $j - a_i$ modulo $m$. Therefore, the transition is $DP[i][j] = DP[i-1][(j-a_i+m) \% m] $ (adding $m$ before applying the modulo operation again to prevent a negative index).

It might seem like we solved the problem with Dynamic Programming, but there's an important catch. The state needs to store up to $DP[n][m]$; looking at the constraints, $10^6  \times  10^3 = 10^9 > 10^8$ would exceed both the time and memory limits. This means we cannot use the Dynamic Programming solution directly.

### Part 2

Notice that the problem only requires us to find out whether a valid sequence exists - we do not need to give an example of such a sequence.​ This motivates us to consider whether there is a situation where a valid sequence always exists.

Let's consider an arbitrary sequence $a$. Let $p_i$ store the sum of the elements from $a_1$ to $a_i$, modulo $m$. The key observation is that if there are two values $p_i$ and $p_j$ are equal for some $i < j$, then the sum of the elements from $a_i$ exclusive to $a_j$ inclusive is divisible by $m$. Note it is important that $i ≠ j$, since we are looking for non-empty subsequences.

To understand this, let's review the usage of [Prefix Sums](https://usaco.guide/silver/prefix-sums?lang=cpp). The idea is that every contiguous section of  elements is the difference between two prefixes, the first ending at the right inclusive bound and the second ending at the left exclusive bound (read the linked article for a more detailed explanation). In this case, if two distinct prefix sums modulo $m$ are equal, that means the contiguous section of elements between them is exactly 0 modulo $m$, which by definition means that it is divisible by $m$.

It remains to determine the conditions such that we are guaranteed to have two distinct prefixes with equal sums modulo $m$. Notice that we can only obtain values from 0 to $m-1$ if we operate under modulo $m$ - this is exactly $m$ values. By the [Pigeonhole Principle](https://en.wikipedia.org/wiki/Pigeonhole_principle), if we have more than $m$ values, at least two of them must be equal. Therefore, if we have a sequence longer than length $m$, then we will always be able to choose a valid sequence.

From here, if we separate the case where $n > m$ to always have the answer "YES", we will only perform Dynamic Programming when $n \leq m$. Now the maximum number of states becomes $10^3  \times  10^3 = 10^6 < 10^8$, which is permissible. This completes the solution.


## Final Algorithm

Piecing together Parts 1 and 2, we have the following solution:

### Pseudocode

```
If n > m, the answer is "YES" and we are done.
Otherwise,
    Initialize our base case to DP[0][0] = true since 0 elements will have a sum of 0 modulo m.
    For every a_i in the sequence
        For every j value from 0 to m
            DP[i][j] = DP[i-1][j] || DP[i-1][(j-a[i]+m)\%m]
    Output "YES" if DP[n][m] is true, otherwise output "NO".
```


### Sample Implementation in C++

```c++
#include <bits/stdc++.h>
using namespace std;

long long n,m;
bool dp[1005][1005];

int main(){
	cin >> n >> m;
	if (n > m) cout << "YES\n";
	else {
		dp[0][0] = true;
		for (int i = 1; i <= n; ++i){
			long long a; cin >> a; a %= m;
			for (int j = 0; j <= m; ++j){
				dp[i][j] = dp[i-1][j] || dp[i-1][(j-a+m)%m];
			}
		}
		cout << (dp[n][m] ? "YES" : "NO") << "\n";
	}
	return 0;
}
```

The final time complexity is $O(m^2)$.

___


## Comments

Part of this problem is standard Dynamic Programming, which is great for practice, but the cool part is recognizing the result of the $n > m$ case. This application of Pigeonhole Principle might've seemed contrived, but Pigeonhole Principle is often used in problems where we deal with values under a certain modulo (especially in math). In fact, this exact method of using prefix sums to prove this case was mentioned in Problem-Solving Strategies by Arthur Engel in the "Box Principle" chapter (p.s. this is a great book for fellow math enthusiasts).