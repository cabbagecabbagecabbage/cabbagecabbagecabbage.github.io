---
title: "Problem Analysis 6"
date: 2022-01-04 12:00:00 +0000
categories: [Academic]
tags: [competitive-programming, problem-analysis-series]
math: true
---


# [Atlantis](https://open.kattis.com/problems/atlantis)
2017 ACM ICPC North Central North America Regional Contest​


## Problem Statement

### Task Description

​You may have heard of the lost city of Atlantis. As legend goes, Atlantis was a city of great wealth and power. Then the Gods became displeased with Atlantis and sank it into the ocean. What you may not have heard is the story of Demetrios, the only person in Atlantis with a ship at the time the city was pushed underwater.

Demetrios was an incredibly selfish man and thought of only one thing when he noticed the water level in the city rising: collecting the wealth of accessible gold in Atlantis and transporting it into his ship before the treasure was lost forever. Luckily for Demetrios, when the water level began rising, the many guards keeping safe the gold stores abandoned their posts to seek safety.

Demetrios knows the location of every gold store in Atlantis and how long it will take him to get to a particular store and back to his ship. He also knows the altitude at which each gold store resides, which determines when the store will be swallowed by the rising sea. The trouble is that he’s not sure he’ll have time to make it to every store before they become submerged. He now wonders the maximum number of stores he can visit prior to their respective submersion, if he picks his schedule optimally.

During the 2017 NCNA Regional, the following clarification was posted: “Important: The gold store must remain above water during the ENTIRE trip to and from the store.”


### Input

The first line of input will contain the integer $n$, $(1 \leq n \leq 200000)$, the number of gold stores in Atlantis. The next $n$ lines will contain two integers on each line, $t_i$ and $h_i$, $(1 \leq t_i,h_i \leq 10^9)$, the round-trip time in seconds it will take Demetrios to visit store $i$ and return to his ship with the gold, and the feet above sea level of store $i$, respectively.


### Output

Output the maximum number of gold stores Demetrios can visit such that each is visited prior to it becoming submerged. Assume sea level rises by one foot every second, starts at height $0$, and begins rising immediately.​


## Analysis

### Summary of the Problem Statement

Let us extract the key information from the problem:
-  The sea level rises one foot every second, starting at height $0$.
- A store is defined by the time it takes to visit and its height.
- If we visit a store, we must be able to return before it is submerged (i.e., the sea level rises above its height).
We are given $n$ of these stores, and we must choose the largest possible subset of these stores to visit satisfying the condition above.
​
With a clear grasp of the problem, we can begin to make a few observations.

### Part 1

Since we are choosing some subset of stores to visit in some order, it is natural to consider whether the order in which we visit the stores matters with regards to whether it is possible to visit all the stores in the subset.

Suppose there are 2 stores we want to visit, Store A and Store B. 
- Store A takes 3 seconds to visit and has height 4
- Store B takes 1 second to visit and has height 2

If we visit the stores in that order, we will find that Store B is already submerged when we return from Store A because it took 3 seconds to go to the first store, but it only took 2 seconds for Store B to submerge.

However, if we visit Store B first, we will be able to visit both stores because for Store A $1 \leq 2$ and for Store B $1+3 \leq 4$. Thus, we conclude that the order is indeed significant when determining whether it is possible to visit some set of stores.

Now let's consider a general case. Suppose there are 2 stores we want to visit, Store A and Store B.
- Store A takes $t_A$ seconds to visit and has height $h_A$.
- Store B takes $t_B$ seconds to visit and has height $h_B$.

Again, there are 2 possible orders in which we can visit both.
1. If we visit Store A then Store B, $t_A  \leq  h_A$ and $t_A+t_B  \leq  h_B$ must hold. Let's call this Order 1\.
2. If we visit Store B then Store A, $t_B  \leq  h_B$ and $t_B+t_A  \leq  h_A$ must hold. Let's call this Order 2\.
​

[Without loss of generality](https://en.wikipedia.org/wiki/Without_loss_of_generality), let us assume that $h_A ≥ h_B$. We can then separate into two cases:
1. If $t_B > h_B$
    1. Order 1 cannot visit both stores because $t_A+t_B  \leq  h_B$ is not possible.
    2. Order 2 cannot visit both stores because $t_B  \leq  h_B$ is not possible.
2. If $t_B  \leq  h_B$
    1. Order 1 can visit both stores if and only if $t_A+t_B  \leq  h_B$ (note that $t_A  \leq  h_A$ is a subset of this constraint because $h_A ≥ h_B$).
    2. Order 2 can visit both stores if and only if $t_B+t_A  \leq  h_A$.
​
We can see that in the first case the 2 orders are equivalent with respect to visiting both stores. However, in the second case, $t_B+t_A  \leq  h_A$ is at least as "loose" of a constraint as $t_A+t_B  \leq  h_B$ because of our assumption $h_A ≥ h_B$. Therefore, Order 2 is at least as good as, if not better than Order 1 when determining whether it is possible to visit both stores.

Note: in the case that visiting both stores is not possible in either order and we are only visiting one of the stores, the order does not matter, which means our conclusion still holds.
​
With this example, we have shown that given any two stores, it is optimal to consider them in order of non-decreasing height. In layman's terms, we are essentially saying "we should visit the stores that will submerge faster first", which is actually pretty intuitive.

Therefore, we should sort all $n$ stores in order of non-decreasing height and consider whether or not to include them in our subset in that order. With this insight, we can now rephrase the problem: Given the sorted sequence of $n$ stores, find the longest subsequence of stores such that every store in the subsequence can be visited before it submerges.
​
Formally, we now need to find the longest sequence $i_1, i_2, ..., i_l$ $(1 \leq l \leq n)$ such that

$$\sum_{k=1}^{j} t_{i_k} \leq h_{i_j}$$

for all $1  \leq  j  \leq  l$.
​

### Part 2

A quick glance at the constraints on $n$ $(1 \leq n \leq 2 \times 10^5)$ and the values of $t_i$ and $h_i$ $ (1  \leq  t_i, h_i  \leq  10^9) $ rules out any brute force or dynamic programming solutions. Therefore, it is natural to wonder whether it is possible to greedily decide whether or not to include a store in our subsequence as we iterate through the $n$ sorted stores.

Suppose we had 3 stores we can choose from to visit (sorted in non-decreasing height), as follows:
- Store A takes 3 seconds to visit and has height 3.
- Store B takes 2 seconds to visit and has height 4.
- Store C takes 3 seconds to visit and has height 5.

If we consider the stores in order and include a store whenever it is feasible, we would greedily visit Store A since $3  \leq  3$. However, when we get to Store B we will find that $3 + 2 > 4$ and when we get to Store C we will find that $3 + 3 > 5$; neither can be visited after visiting Store A.

Instead, if we skipped over Store A and considered the rest of the stores in order, we will find that it is possible to visit both Store B and Store C since $2 \leq 4$ and $2 + 3  \leq  5$. From this, we can conclude that the naïve greedy strategy is not optimal.

However, the idea of skipping over stores that hinder our optimality may prove very valuable to us. What if we include every store in our subsequence, then skip over as few stores as possible such that our subsequence satisfies the condition (see end of Part 1)? (Note: we want to remove as few stores as possible because we want the longest possible subsequence as our answer)

Two questions that arise from this idea would be
- "When do we need to skip over stores?"
- "Which stores should we skip over to optimize our answer?"

Suppose we are iterating through the stores and including every store in our subsequence as described. At some point, we may find that the sum of all the visiting times exceeds the height of the current store. This means that it is impossible to visit all the stores in our subsequence, and thus we need to skip over one or more of them so that the condition is satisfied.

When deciding which stores should be skipped over, the following insight is crucial: every store contributes equally to our answer (the length of the subsequence), but the more time it takes to visit a store, the more it hinders our ability to include more stores because it increases the total time that is used to compare against the heights. Therefore, we should skip over the stores that take the longest to visit first until our subsequence satisfies the condition again.

Furthermore, we can observe that if we continuously maintain a valid subsequence, we will never have to skip over more than one store whenever we include a new store. This is because we are considering the maximum time including the time to visit the store we just included, and since we started with a valid subsequence, skipping over this store or any store that takes at least as long to visit as this store will ensure that we result in a valid subsequence again.



## Final Algorithm

Piecing together the observations from Parts 1 and 2, we have the following algorithm outline:

### Pseudocode

```
Sort the stores in non-decreasing order.
Iterate through the stores in order
For each store:
    Include the store in our subsequence, and increment our answer.
    If the total time exceeds the height of the current store, exclude the store in our subsequence that takes the longest to visit, and decrement our answer.
```

### Choosing a Data Structure
Now we just need a data structure that supports sub-linear time insertions and extractions for maximum. One such data structure is binary heap (implemented as std::priority_queue in C++), which supports $O(\log n)$ inserts and extractions.

### Sample Implementation in C++

```c++
#include <iostream>
#include <algorithm>
#include <queue>
using namespace std;

pair<int,int> stores[200005]; //pairs of {height, time}

int main() {
	int n; cin >> n;
	for (int i = 0; i < n; ++i){
		cin >> stores[i].second >> stores[i].first;
	}
	sort(stores, stores + n);
	int ans = n;
	int totaltime = 0;
	priority_queue<int> pq;
	for (int i = 0; i < n; ++i){
		pq.push(stores[i].second);
		totaltime += stores[i].second;
		if (totaltime > stores[i].first){
			--ans;
			totaltime -= pq.top(); pq.pop();
		}
	}
	cout << ans << "\n";	
	return 0;
}
```

The final time complexity is $O(n\log n)$.

___


## Comments

- The official solution slides can be found here: [NCNA17slides.pdf (uwaterloo.ca)](https://cs.uwaterloo.ca/~bcsandlu/NCNA17slides.pdf). This analysis was inspired by the solution shown on slide 23.
- The process of proving that sorting the stores in order of non-decreasing height is optimal (see Part 1)  is very similar to the proofs in exchange argument dynamic programming problems. see [Lecture #3 — Exchange arguments (sorting with dp) - Codeforces](https://codeforces.com/blog/entry/63533).
- Most other greedy algorithms choose the locally optimal choice at each step of the algorithm, but this greedy algorithm is creative because it chooses all the stores to visit at first, then removes the worst ones, making it much easier to prove correct and implement.
- Like the first problem presented in the series, I discovered this problem through the "CCC Training Camp" problem set curated by <a href="https://dmoj.ca/user/xiaowuc1" target="_blank">Nick Wu</a>. The full list of problems can be found <a href="https://amolina.ca/bojlist/camp-vert" target="_blank">here</a>.