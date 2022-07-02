---
title: "Problem Analysis 3"
date: 2021-12-28 12:00:00 +0000
categories: [Academic]
tags: [competitive-programming, problem-analysis-series]
math: true
---


# [Count Triangles](https://codeforces.com/problemset/problem/1355/C)
Codeforces Round #643 Div.2 C


## Problem Statement

### Task Description

Like any unknown mathematician, Yuri has favourite numbers: $A$, $B$, $C$, and $D$, where $(A \leq B \leq C \leq D)$. Yuri also likes triangles and once he thought: how many non-degenerate triangles with integer sides $x$, $y$, and $z$ exist, such that $(A \leq x \leq B \leq y \leq C \leq z \leq D)$ holds?

Yuri is preparing problems for a new contest now, so he is very busy. That's why he asked you to calculate the number of triangles with described property.

The triangle is called non-degenerate if and only if its vertices are not collinear.


### Input

The first line contains four integers: $A$, $B$, $C$ and $D$ $(1 \leq A \leq x \leq B \leq y \leq C \leq z \leq D \leq 5 \times 10^5)$ - Yuri's favourite numbers.


### Output

Print the number of non-degenerate triangles with integers sides $x$, $y$ and $z$ such that the inequality $(A \leq x \leq B \leq y \leq C \leq z \leq D)$ holds.


## Analysis

The first thing we need to find out is the condition that our side lengths $x$, $y$ and $z$ must satisfy such that the vertices of the triangle will not be collinear. Consider the longest side of the triangle (A in diagram) and imagine the other two sides (B and C in diagram) each connected at the ends of the longest side. If the longest side is longer than the combined length of the two sides, they cannot close to form an angle no matter how we swing them.

![Triangle inequality diagram](https://raytonc.weebly.com/uploads/1/1/0/7/110796697/793056179_orig.png)

From this we conclude that the sum of the lengths of any two sides of a triangle must be greater than the remaining side. This is known as the triangle inequality. In the context of the problem, we are given $x \leq y \leq z$, so it suffices to ensure that $x + y > z$.

The problem now becomes "find the number of triples $(x,y,z)$ satisfying the constraints such that $x + y > z$. The method we will use to solve this problem is to repeatedly optimize a naïve solution until we obtain a valid solution.

Let's first consider the obvious brute force solution: iterate through all possible values of $x$, $y$ and $z$ and add $1$ to our answer whenever $x + y > z$.

### Pseudocode 1

```
for each x value in [A,B]:
    for each y value in [B,C]:
        for each z value in [C,D]:
            if x + y > z:
                add 1 to the answer
print the answer
```

Although this solution would give the correct answer, it is too slow. Notice that $A$, $B$, $C$ and $D$ can go up to $5 \times 10^5$, so it is possible for each of the intervals to have length $10^5$. Since we have 3 nested loops, we could be performing $(10^5)^3=10^{15}$ operations, which is way too many (see [Time Complexity · USACO Guide](https://usaco.guide/bronze/time-comp?lang=cpp); most importantly, C++ can handle about $10^8$ operations per second). We need to optimize this solution so that it can fit into the time limit.

Let's start by fixing a value of $z$. For this value of $z$, how many triples of $(x,y,z)$ satisfy $x + y > z$? Since $z$ is fixed, the answer depends entirely on $x$ and $y$. The question then becomes, "how many pairs $(x,y)$ satisfy $x + y > z$?"

To answer this question efficiently, instead of looping through every single pair for every fixed value of $z$, we use the idea of precomputation. If we precompute an array defined as $arr[i]$ stores the number of pairs $(x,y)$ that satisfies $x + y = i$, we can loop through $arr[z+1]$ to $arr[B+C]$ instead. Notice that $z$ is at least $C$, so the interval is at worst $[C+1,B+C]$ which has length $B$.

To precompute this array, we can loop through all pairs of $x$ and $y$ and add $1$ to $arr[x+y]$ for every pair. The reason this optimizes our solution is that we are doing this exactly once before we loop through the fixed values of $z$, instead of doing it every time.

### Pseudocode 2

```
for each x in [A,B]:
    for each y in [B,C]:
        add 1 to arr[x+y]
for each z in [C,D]:
    for each i in [z+1,B+C]:
        add arr[i] to answer
print the answer
```

Again, let's approximate the number of operations we perform in our solution. In the first nested for-loop, we could perform $10^5 \times 10^5 = 10^{10}$ operations. In the second nested for-loop, we could perform $10^5 \times 10^5 = 10^{10}$ operations as well. In total, we could perform a factor of $10^{10}$ operations, which is much better than before but still impermissible (greater than $10^8$).

Now that our solution is split into two parts: precomputation and using the array to compute the answer by fixing values of $z$. Let's optimize them separately with data structures. These data structures are better explained by the linked articles, so I encourage you to go read up on them first if you are unfamiliar with them.

The optimization of the second part is easier to see: we are repeatedly performing static range sum queries on $arr$ for every fixed value of $z$, so we can precompute a Prefix Sum Array to handle these queries in constant time.

The optimization of the first part uses a very similar data structure. This time, notice that we are repeatedly performing range updates (more specifically, increments) for every fixed $x$ from $arr[x+B]$ to $arr[x+C]$. This can be optimized with the usage of a Difference Array by adding $1$ to $arr[x+B]$ and subtracting $1$ from $arr[x+C+1]$ for every $x$, then performing a linear scan (identical to precomputing the Prefix Sum Array) afterwards.

### Pseudocode 3

```
for every x in [A,B]:
    add 1 to arr[x+B]
    subtract 1 from arr[x+C+1]
for every i in [A+B, END):
    arr[i] = arr[i] + arr[i-1]
for every i in [A+B, END):
    arr[i] = arr[i] + arr[i-1]
for every z in [C,D]:
    add arr[END-1] - arr[z] to answer
print the answer
```

Finally, let's calculate the number of operations again.
The first for-loop could perform a factor of $10^5$ operations
The second and third for-loops (they are identical) could each perform a factor of $10^5$ operations
The fourth for-loop could perform $10^5$ operations

In total, we will not perform more than $10^6$ operations, which fits comfortably within the limit of $10^8$ operations per second. This completes the solution.


## Final Algorithm

### Sample Implementation in C++

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxn = 2e6 + 5;
long long a,b,c,d,arr[maxn];
 
int main() {
	cin >> a >> b >> c >> d;
	for (int x = a; x <= b; ++x){
		++arr[x+b];
		--arr[x+c+1];
	}
	//to unfold the difference array
	for (int i = a+b; i < maxn; ++i){
		arr[i] += arr[i-1];
	}
	//precompute the psa
	for (int i = a+b; i < maxn; ++i){
		arr[i] += arr[i-1];
	}
	long long ans = 0;
	for (int z = c; z <= d; ++z){
		//sum up arr[z+1] to the end
		ans += arr[maxn-1] - arr[z];
	}
	cout << ans << "\n";
	return 0;
}
```

___


## Comments

- This analysis presents the idea of repeatedly optimizing a slow brute-force solution to obtain a solution that is fast enough for the time limit (as well as how to calculate the operations). The two key methods here are optimizing using precomputation and optimizing using data structures, both of which are applicable in many problems.

- It should be noted that there is another solution to the problem. Using the same idea of fixing the value of $z$, we can mathematically determine the number of pairs $(x,y)$ such that $x+y > z$ using the bounds on $x$ and $y$.