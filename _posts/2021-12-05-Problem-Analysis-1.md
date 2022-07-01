---
title: "Problem Analysis 1"
date: 2021-12-05 12:00:00 +0000
categories: [Academic]
tags: [competitive-programming, problem-analysis-series]
math: true
---

# [Chocolate](https://www.acmicpc.net/problem/7998)
Polish Olympiad in Informatics 2002/2003 Stage 1 Problem 2

## Problem Statement

### Task Description

We are given a bar of chocolate composed of $m \times n$ square pieces. One should break the chocolate into single squares. Parts of the chocolate may be broken along the vertical and horizontal lines as indicated by the broken lines in the picture. A single break of a part of the chocolate along a chosen vertical or horizontal line divides that part into two smaller ones. Each break of a part of the chocolate is charged a cost expressed by a positive integer. This cost does not depend on the size of the part that is being broken but only depends on the line the break goes along. Let us denote the costs of breaking along consecutive vertical lines with $x_1,x_2,\dots,x_{m−1}$ and along horizontal lines with $y_1,y_2,\dots,y_{n−1}$. The cost of breaking the whole bar into single squares is the sum of the successive breaks. One should compute the minimal cost of breaking the whole chocolate into single single squares.

![Sample Diagram](https://raytonc.weebly.com/uploads/1/1/0/7/110796697/713688102.jpg)

For example, if we break the chocolate presented in the picture first along the horizontal lines, and next each obtained part along vertical lines then the cost of that breaking will be

$$y_1 + y_2 + y_3 + 4 \cdot (x_1 + x_2 + x_3 + x_4 + x_5)$$

Write a program which:
- reads the numbers $x_1,x_2,\dots,x_{m-1}$ and $y_1,y_2,\dots,y_{n-1}$,
- computes the minimal cost of breaking the whole chocolate into single squares,
- writes the result.

### Input

In the first line of the standard input there are two positive integers $m$ and $n$ separated by a single space, $2 \leq m,n \leq 1,000$. In the successive $m-1$ lines there are numbers $x_1,x_2,\dots,x_{m-1}$, one per line, $1  \leq  x_i  \leq  1,000$. In the successive $n-1$ lines there are numbers $y_1,y_2,\dots,y_{n-1}$, one per line, $1  \leq  y_i  \leq  1,000$.

### Output

Your program should write to the standard output. In the first and only line your program should write one integer - the minimal cost of breaking the whole chocolate into single squares.


## Analysis

It isn't immediately obvious what the optimal strategy is when cutting the chocolate: Along which line should we cut first? How does a cut across a line affect the cuts afterwards? This is when we "play around" with the problem - its never a bad idea to solve a trivial case by hand to get a feel for the problem.

Consider a $2\times2$  piece of chocolate with $y_1 = 1$ and $x_1 = 2$.

![2 by 2 diagram](https://raytonc.weebly.com/uploads/1/1/0/7/110796697/published/793629628.png?1641064140)

Let's first cut the chocolate horizontally along $y_1$, from which we obtain two $1\times2$ pieces of chocolate. Each of these pieces are then cut along $x_1$, breaking them into single squares as required. The total cost incurred is $y_1 + 2 \times x_1 = 1 + 2 \times 2 = 5 $.

![Diagram for first method](https://raytonc.weebly.com/uploads/1/1/0/7/110796697/332388154.png)

Alternatively, we can cut vertically along $x_1$ first, leaving two $2\times1$ pieces. Each of these pieces are then cut along $y_1$, breaking them into single squares as desired. The total cost incurred is $x_1 + 2 \times y_1 = 2 + 2\times1 = 4$.

![Diagram for second method](https://raytonc.weebly.com/uploads/1/1/0/7/110796697/548226635.png)

This example helps us notice several things:
- We must make a break along each line $y_1,y_2,\dots,y_{n-1}$ and $x_1,x_2,\dots,x_{m-1}$ at least once, otherwise there will be squares that are still connected by that line.
- When we break a piece horizontally, it produces two pieces; now when we try to make a vertical break along a line within each of the pieces, we will need to do it twice instead of once.
- When we break a piece vertically, it produces two pieces; now when we try to make a horizontal break along a line within each of the pieces, we will need to do it twice instead of once.

Intuitively, to achieve the minimum cost, the more costly breaks should be done as little as possible. Since every line needs to be broken along at least once and every break affects all perpendicular breaks that come afterwards, we conclude that we should break along the lines in order of non-increasing cost (that is, the more costly lines should be broken along first).

One final implementation detail is unclear: How will we know the number of necessary breaks along a given line? Recall that every horizontal break adds 1 to the number of breaks along any vertical line in the future (and vice versa). Thus the number of pieces that will need a vertical break along a given vertical line is exactly the number of horizontal breaks already made, plus 1. When implementing the solution, we keep track of whether the line we are processing is vertical/horizontal, and we keep maintain variables storing the number of horizontal breaks and the number of vertical breaks already made, incrementing as we make breaks.

## Final Algorithm

### Pseudocode

```
push horizontal lines into a container
push vertical lines into a container
sort the container in order of non-increasing cost
let verticalBreaks = 0, horizontalBreaks = 0
for every line in the container:
    if the line is horizontal:
        add (verticalBreaks + 1) * cost to answer
        add 1 to horizontalBreaks
    if the line is vertical:
        add (horizontalBreaks + 1) * cost to answer
        add 1 to verticalBreaks
print the answer
```

### Sample Implementation in C++

```c++
#include <bits/stdc++.h>
using namespace std;

const int nax = 1e3 + 5;
int m,n;
pair<int,int> lines[nax*2];

int main() {
	cin >> n >> m;
	for (int i = 0; i < m-1; ++i){
		int c; cin >> c;
		lines[i] = {c,0};
	}
	for (int i = 0; i < n-1; ++i){
		int c; cin >> c;
		lines[m-1+i] = {c,1};
	}
	sort(lines, lines + n + m - 2, greater<pair<int,int>>());
	int h = 1, v = 1, ans = 0;
	for (int i = 0; i < m + n - 2; ++i){
		// O(M+N)
		if (lines[i].second == 0){
			ans += h * lines[i].first;
			++v;
		}
		else if (lines[i].second == 1){
			ans += v * lines[i].first;
			++h;
		}
	}
	cout << ans << "\n";
	return 0;
}
```

The time complexity of the solution is $O((n+m)\log{(n+m)})$ from sorting.


___


## Comments

- This problem teaches the usefulness of solving small cases by hand. The problem may seem difficult at first, but going through the possibilities of breaking up the chocolate makes the observations much more obvious.

- I discovered this problem through the "CCC Training Camp" problem set curated by <a href="https://dmoj.ca/user/xiaowuc1" target="_blank">Nick Wu</a>. The full list of problems can be found <a href="https://amolina.ca/bojlist/camp-vert" target="_blank">here</a>.