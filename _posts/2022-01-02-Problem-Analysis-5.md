---
title: "Problem Analysis 5"
date: 2022-01-02 12:00:00 +0000
categories: [Academic]
tags: [competitive-programming, problem-analysis-series]
math: true
---

# [King Gruff](https://dmoj.ca/problem/cco14p2)
Canadian Computing Competition: 2014 Stage 2, Day 1, Problem 2


## Problem Statement

### Task Description

King Gruff the Wolf rules over a happy, prosperous land inhabited by adorable Foxen. Unfortunately for them, he is not a nice king at all, and wants to make their lives miserable!

His country has $N$ $(1 \leq N \leq 10^5)$ cities, with $M$ $(1 \leq M \leq 10^5)$ roads running amongst them. The $i$-th road allows one to travel from city $X_i$ to a different city $Y_i$ $(1 \leq X_i, Y_i \leq N)$, in that direction only, and has a length of $L_i$ $(1 \leq L_i \leq 10^4)$ and a shutdown cost of $C_i$ $(1 \leq C_i \leq 10^4)$. There may be multiple roads running between a pair of cities, even in the same direction.

King Gruff particularly dislikes the Foxen living in two different cities $A$ and $B$ $(1 \leq A,B \leq N)$, and would like to make it more inconvenient (or even impossible) to travel from city $A$ to city $B$. In particular, he'll select a distance value $D$ $(1 \leq D \leq 10^9)$, and then simultaneously shut down every single road in his kingdom which is part of at least one path from city $A$ to city $B$ with total length no more than $D$. For each such road, however, he'll have to dig into his royal treasury and pay its shutdown cost. A path consists of a sequence of roads such that each road (except the first) starts at the city that the previous road ended at, and may visit a city or road multiple times.
​
Gruff is having trouble making up his mind about what value of $D$ to choose, however - a larger value would make things more inconvenient for his Foxy subjects, but might cost him more money as well! As such, he'll consider $Q$ $(1 \leq Q \leq 10^5)$ different values, $D_1, D_2, ..., D_Q$. For each one, he'd like to know how many tax dollars would need to be spent to shut down all roads which lie on at least one sufficiently short path from city $A$ to city $B$. Since you don't like Foxen either, you've agreed to help the Wolf write a program to calculate this!


### Input

The first line will contain 4 integers, each separated by a space:
- $N$ $(1 \leq N \leq 10^5)$, the number of cities, followed by
- $M$ $(1 \leq M \leq 10^5)$, the number of roads, followed by
- $A$ $(1 \leq A \leq N)$, the starting city, followed by
- $B$ $(1 \leq B \leq N)$, the ending city.

Each of the next M lines contain four space-separated integers $X_i$, $Y_i$, $L_i$, and $C_i$ describing the road from $X_i$ to $Y_i$ with length $L_i$ and shutdown cost $C_i$, where $1 \leq X_i, Y_i \leq N$, $1 \leq L_i, C_i \leq 10^4$.
The next line will contain $Q$ $(1 \leq Q \leq 10^5)$, the number of different distance values to consider.
The next $Q$ lines each contain one integer $D_i$ $(1 \leq D_i \leq 10^9)$ which is the distance value to consider in shutting down roads.


### Output

The output consists of $Q$ lines, each with one integer, representing the total cost required to shut down all necessary roads given a distance value of $D_i$, for $i = 1 ... Q$.


## Analysis

### Summarizing and Scouting

Since the problem statement is quite long and hard to understand at first glance, it's helpful to summarize the key points of the problem statement.
- We are given a directed graph with edges that have two attributes: length and shutdown cost.
- We are given $Q$ queries as follows: Given a value $D$, consider every path from node $A$ to node $B$: if the path has total length not exceeding $D$, then every edge on that path should be shut down. Find the total shutdown cost (the sum of the shutdown cost of each edge that should be shut down).

Looking at the constraints on $Q$, it is clear* we need to answer each of the queries in sublinear time. Since there is a linear amount of edges and we cannot process a linear amount of objects in sublinear time, we will need to do some preprocessing before attempting to answer the queries.

*$N$, $M$, and $Q$ go up to $10^5$, so if our algorithm is quadratic or worse, it likely will not pass within 1 second: $(10^5)^2 > 10^8$ (refer to [Time Complexity · USACO Guide](https://usaco.guide/bronze/time-comp?lang=cpp)).

### Part 1

Consider an edge in the graph. When determining whether it is part of a path from node $A$ to node $B$ with length not exceeding $D$ (for some given value $D$), it suffices to know the length of the shortest such path through that edge, and compare that with $D$. Therefore, we want to find this shortest length for all edges in subquadratic time.

Let $dist(x,y)$ denote the length of the shortest path from node $x$ to node $y$.

Recall that each edge goes in one direction from one node to another. Let's call the starting node $S$ and the ending node $E$.

A path from node $A$ to node $B$ going through the edge can be broken down into 3 parts:
1. The path from node $A$ to node $S$
2. The path from node $S$ to node $E$ (the edge itself)
3. The path from node $E$ to node $B$

![Path decomposition diagram](https://raytonc.weebly.com/uploads/1/1/0/7/110796697/604816541_orig.png)

Since the shortest path problem has [optimal substructure](https://en.wikipedia.org/wiki/Optimal_substructure), the shortest path must be comprised of sub-paths with the shortest distance.

Therefore, the shortest path from node $A$ to node $B$ going through the edge is equivalent to the sum of:
1. $dist(A,S)$
2. The length of the edge (which is constant for a given edge)
3. $dist(E,B)$

Now we just need to be able to retrieve $dist(A,S)$ and $dist(E,B)$ for any given $S$ and $E$.

Running an all-pairs shortest path on the graph is not viable here because the graph is fairly large. Luckily, we have a fixed node for each of our $dist(x,y)$ queries - $A$ and $B$ are constant for every edge we process. 

For $dist(A,S)$, notice that this is equivalent to the single-source shortest path (SSSP) problem, rooted at A.

For $dist(E,B)$, we now have the destination fixed instead of the beginning. Here, notice that if we reverse all the edges in a graph $G$ to obtain $G'$, $dist(x,y) = dist'(y,x)$. Therefore, this is equivalent to the single-source shortest path on the reverse graph.

An algorithm that solves the SSSP in subquadratic time is Dijkstra's Algorithm. Now we can calculate the shortest distance from node $A$ to node $B$ through any edge in constant time. From now on, this will be referred to as the value​ of the edge.

In case one is unfamiliar with Dijkstra's Algorithm, [this video](https://www.youtube.com/watch?v=pVfj6mxhdMw) explains it well.


### Part 2

Let us call an edge valid if its value does not exceed a given value of $D$, and invalid otherwise.
Each query has now been reduced to the following:
- Given edges with value and cost and a value of $D$, find the sum of the costs of all valid edges.

Going through all the edges for each query would take linear time, which is not permissible for a linear amount of queries.

However, if we sort the array of edges by non-decreasing value, notice that we will always be taking a prefix of these edges. Consider the valid edge in the array that furthest to the right: every edge to the left of it (including itself) must be valid, and every edge to the right of it must be invalid (or else the assumption that it is furthest to the right is false). This edge can be found by binary search in $O(\log N)$ time per query.

It remains to sum up the shutdown costs of these edges. Similarly, iterating through these edges for each query would take linear time, which is impermissible. Instead, notice that since we are computing the sum of a contiguous segment of values, a prefix sum array can be precomputed and used to find the sum of the prefixes in $O(1)$ time. This completes the solution.


## Final Algorithm

Piecing together parts 1 and 2, we have the following algorithm:

### Pseudocode

```
Run Dijkstra's algorithm from A on the original graph
Run Dijkstra's algorithm from B on the reverse graph
Calculate the value of each edge
Sort the edges by non-decreasing value
Precompute a prefix sum array on the shutdown cost of the sorted edges
For each query:
    Binary Search for the valid edge furthest to the right
    Return the prefix sum at that index
```

### Sample Implementation in C++

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxn = 1e5 + 5;
int n,m,a,b,dist[maxn],dist2[maxn];
struct edge{
	int x,y,l,c;
} edges[maxn];
vector<pair<int,int>> adj[maxn],adj2[maxn],valcost;

int main(){
	cin >> n >> m >> a >> b;
	for (int i = 0; i < m; ++i){
		int x,y,l,c; cin >> x >> y >> l >> c;
		edges[i] = {x,y,l,c};
		adj[x].emplace_back(y,l);
		adj2[y].emplace_back(x,l);
	}
	
	//first dijkstras
	memset(dist,0x3f,sizeof dist);
	dist[a] = 0;
	priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
	pq.push({0,a});
	while (!pq.empty()){
		int cost, node;
		tie(cost,node) = pq.top(); pq.pop();
		if (cost != dist[node]) continue;
		for (pair<int,int> k: adj[node]){
			if (dist[k.first] > cost + k.second){
				dist[k.first] = cost + k.second;
				pq.push({cost+k.second,k.first});
			}
		}
	}
	
	//second dijkstras
	memset(dist2,0x3f,sizeof dist2);
	dist2[b] = 0;
	pq.push({0,b});
	while (!pq.empty()){
		int cost, node;
		tie(cost,node) = pq.top(); pq.pop();
		if (cost != dist2[node]) continue;
		for (pair<int,int> k: adj2[node]){
			if (dist2[k.first] > cost + k.second){
				dist2[k.first] = cost + k.second;
				pq.push({cost+k.second,k.first});
			}
		}
	}
	
	//process edges
	int inf = 0x3f3f3f3f;
	for (int i = 0; i < m; ++i){
		int a = dist[edges[i].x], b = edges[i].l, c = dist2[edges[i].y];
		if (a != inf && c != inf){
			valcost.emplace_back(a+b+c,edges[i].c);
		}
	}
	
	//sort {value, edge} pairs
	sort(valcost.begin(),valcost.end());
	
	//prefix sum array
	for (int i = 1; i < valcost.size(); ++i){
		valcost[i].second += valcost[i-1].second;
	}
	
	//handle queries
	int q; cin >> q;
	while (q--){
		int d; cin >> d;
		//binary search
		int lo = 0, hi = valcost.size()-1, ans = -1;
		while (lo <= hi){
			int mid = (lo + hi) / 2;
			if (valcost[mid].first <= d){
				ans = mid;
				lo = mid + 1;
			}
			else {
				hi = mid - 1;
			}
		}
		if (ans == -1) cout << "0\n";
		else cout << valcost[ans].second << "\n";
	}
	return 0;
}
```

Our final time complexity is linearithmic ($N$, $M$, and $Q$ have the same constraints).

___


## Comments

This problem presents several useful concepts:
1. Breaking down a path through an edge and taking advantage of the optimal substructure of shortest paths
2. Dijkstra's algorithm to solve the single-source shortest path, and reversing the edges to solve the single-destination shortest path problem
3. Combining sorting, preprocessing (prefix sum arrays), and binary search to answer each query in logarithmic time

It should be noted that it is also possible to use the two-pointers method to do the final step instead of binary search + prefix sum arrays.