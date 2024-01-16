---
title: 'Lecture 2: Divide and Conquer'
date: 2024-01-16 08:41:38
tags:
- Algorithm
- MIT OCW
categories:
- '6.046J-2015Sping'
toc: true
katex: true
cover: https://cdn.jsdelivr.net/gh/shuyuHU328/picx-images-hosting@master/image.1g58e0c69ycg.webp
---

# Paradigm

对于分而治之问题的时间复杂度计算：对于数据规模为$n$的问题，如果分解成规模为$\frac{n}{b}$的子问题，且$a\ge 1,b > 1$，那么有
$$
T(n)=a T\left(\frac{n}{b}\right)+[work\ for\ merge]
$$

我们会在下一节课通过*Master Theorithm*得到具体的时间复杂度

# Convex Hull

在平面中给定$n$个点，假定任意两个点的x坐标与y坐标都不相同，且没有三个点出现在同一直线上。
$$
S = \{(x_i, y_i)|i = 1, 2,...,n\}
$$
**Convex Hull (CH(S))**: 包含$S$中所有点的最小多边形

![convex hull](https://cdn.jsdelivr.net/gh/shuyuHU328/picx-images-hosting@master/image.22glmow0h8jk.png)

CH(S)可以通过一个双向链表由边界上的点序列按顺时针顺序表示：

<img src="https://cdn.jsdelivr.net/gh/shuyuHU328/picx-images-hosting@master/image.2jhmuiodvm4.webp" alt="dot sequence" style="zoom:67%;" />

## Brute force for Convex Hull

直接遍历每一条线段是否为CH(S)的一条边：

- 若其它所有点都在这条边的一侧，那么这条线段属于CH(S)
- 否则，则不是CH(S)的一条边

$O(n^2)$ edges, $O(n)$ tests ⇒ $O(n^3)$ complexity （*怎样优化？*）

## Divide and Conquer Convex Hull

Sort points by x coord (once and for all, $O(nlogn)$)

For input set *S* of points:

- Divide into left half *A* and right half *B* by x coords
- Compute CH(*A*) and CH(*B*)
- Combine CH’s of two halves (merge step)

## How to Merge?

![merge example](https://cdn.jsdelivr.net/gh/shuyuHU328/picx-images-hosting@master/image.5w6r7hdiba80.webp)

### Finding Tangents

```pseudocode
i=1, j=1
while (y(i, j + 1) > y(i, j) or y(i − 1, j) > y(i, j))
	if (y(i, j + 1) > y(i, j)) -> move right finger clockwise
		j = j + 1(mod q)
	else
		i = i − 1(mod p) -> move left finger anti-clockwise
	return (a_i, b_j) as upper tangent
```

# Median Finding
