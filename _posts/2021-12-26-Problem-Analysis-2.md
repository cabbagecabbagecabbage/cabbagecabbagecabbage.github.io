---
title: "Problem Analysis 2"
date: 2021-12-26 12:00:00 +0000
categories: [Academic]
tags: [competitive-programming, problem-analysis-series]
math: true
---

# [Odašiljači](https://dmoj.ca/problem/coci20c2p2) (Transmitters)

Croatian Open Competition in Informatics '20 Contest 2 Problem #2


## Problem Statement

Sadly, this is the last time Sean will play James Bond.
His mission is to network $n$ antennas that are scattered across a vast desert, which can be represented as a 2D plane. He will set the transmission radius of each antenna to be the **same** non-negative real number $r$. The range of an antenna is defined as the set of all points whose distance to the antenna is at most $r$. If ranges of two antennas have a common point, those antennas can directly communicate. Also, if antennas $A$ and $B$ can communicate, as well as antennas $B$ and $C$, then antennas $A$ and $C$ are also able to communicate, through antenna $B$.
Sean wants to network the antennas, i.e. make it possible for every two antennas to communicate. Since M has limited his spending for this mission, and larger radii require more money, Sean will choose the **smallest possible radius** $r$. Help him solve this problem!

### Input

The first line contains an integer $n$ $(1 \leq n \leq 1000)$, the number of antennas.
Each of the following $n$ lines contains integers $x_i$ and $y_i$ $(0 \leq x_i,y_i \leq 10^9)$, coordinates of the $i$-th antenna.


### Output

Output the minimal radius.
Your answer will be considered correct if its absolute or relative error doesn't exceed $10^{-6}$. 


## Analysis

The problem asks for the smallest real number $r$ such that all the antennas are reachable from each other. When we are looking for the optimal value (especially when the answer is not limited to the set of integers) such that a condition is satisfied, the technique of binary searching should always be considered. In case one is unfamiliar with binary searching on monotonic functions, I give several examples [in this lesson](https://www.youtube.com/watch?v=_w0Y0jbbf2Y) (skip through the irrelevant parts).

Consider the minimal value of $r$ such that all the antennas are reachable from each other. Every value smaller than this value would not satisfy this condition, otherwise the assumption that it is minimal is false. Every value greater than this value would satisfy this condition, but would no longer be minimal since it is greater. This is why we can binary search for our answer.

​How would we know if a value of $r$ would allow all the antennas to communicate with each other? To answer this, let's visualize the problem with a diagram (from the original problem) of one of the sample inputs.

![Sample input diagram](https://raytonc.weebly.com/uploads/1/1/0/7/110796697/702400321.png)

Each of the antennas have a certain subset of antennas they can reach. This motivates us to see the antennas as nodes that are connected to other nodes. We know that two antennas can communicate only if their ranges share a common point. The furthest that two connected antennas can be is when their ranges just barely touch (sharing only 1 point), which is when the distance between them is exactly $2r$ (see antennas at (0,1) and (2,3), sharing the point (1,2)). Therefore, two antennas can communicate if and only if the distance between them is less than or equal to $2r$.

To calculate the distance between any two points on the coordinate plane, we use the distance formula

$$d=\sqrt{(x_2-x_1)^2+(y_2-y_1)^2}$$ 

which can be derived from the Pythagorean theorem (hint: draw a triangle using the two points and a third point using the x-coordinate of one and the y-coordinate of the other).

To check whether all the antennas can communicate, we can use any graph traversal algorithm such as Depth-first Search. Instead of going through an adjacency list, we loop through all other unvisited antennas. When we arrive at an antenna $u$ located at $(x_u,y_u)$, we check whether we can reach antenna $y$ by evaluating

$$\sqrt{(x_u-x_v)^2+(y_u-y_v)^2} \leq 2r$$

which can also be written as

$$(x_u-x_v)^2+(y_u-y_v)^2 \leq 4r^2$$

to make the implementation easier.


## Final Algorithm

Piecing together all the elements, we have the following final algorithm:

### Sample Implementation in C++

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxn = 1e3 + 5;
double n,x[maxn],y[maxn];
bool visited[maxn];

double dist(int u, int v){
	//returns the square of the distance between u and v
	double dx = x[u] - x[v], dy = y[u] - y[v];
	return dx*dx + dy*dy;
}

void dfs(int u, double r){
	//depth-first search to traverse the graph
	visited[u] = true;
	for (int v = 1; v <= n; ++v){
		//only visited unvisited nodes that are close enough
		if (!visited[v] and dist(u,v) <= 4*r*r) dfs(v,r);
	}
}

bool valid(double radius){
	dfs(1,radius); //start the traversal anywhere
	bool ans = true;
	for (int i = 1; i <= n; ++i){
		if (!visited[i]) ans = false; //if any node is unreached, the radius is invalid
		visited[i] = false; //set unvisited for next call
	}
	return ans;
}

int main(){
	cin >> n;
	for (int i = 1; i <= n; ++i){
		cin >> x[i] >> y[i];
	}
	//binary search for the minimum radius
	double lo = 0, hi = 1e9;
	while (hi - lo > 1e-7){
		double mid = (lo + hi) / 2;
		if (valid(mid)) hi = mid;
		else lo = mid;
	}
	cout << fixed << setprecision(7) << lo << "\n";
	return 0;
}
```

The time complexity is $O(\log{10^9}) \times O(n^2) = O(n^2\log{10^9})$, which fits in the time limit.

___


## Comments

This problem combines ideas of binary search, geometry, and graph theory smoothly. Upon hearing "binary search", many people think of trivial applications such as "finding whether a value exists in a sorted array of elements"; it is seen as a basic algorithm. As a result, many contestants neglect practicing binary search problems because they are supposedly simple, until they are unable to solve a non-trivial binary search problem in a contest (note: there certainly exists binary search problems much harder than this one!). Its importance is often overlooked - being able to find the optimal value (not limited to the set of integers!) in any monotonic function makes it one of the most useful algorithms.