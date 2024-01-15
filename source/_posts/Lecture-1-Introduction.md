---
title: 'Lecture 1: Introduction'
date: 2024-01-15 09:22:05
tags:
- Algorithm
- MIT OCW
categories:
- Algorithm
- 6.046J 2015 Spring
toc: true
mathjax: true
---

## P & NP

- **P**: class of problems solvable in polynomial time. $O(n^k)$ for some constant $k$. 

  Shortest paths in a graph.

- **NP**: class of problems verifiable in polynomial time. 

​	Verifying a cycle is hamiltonian.

- **NP-complete**: problem is in NP and is as hard as any problem in NP.

  如果NPC问题可在多项式时间内被解决，那么所有的NP问题都可以在多项式时间内被解决

## Interval Scheduling / 区间调度问题

Requests $1,2, . . . , n$, single resource

$s(i)$ start time, $f(i)$ finish time, $s(i) < f(i)$ (start time must be less than finish time for a request)

Two requests $i$ and $j$ are compatible if they don't overlap, i.e., $f(i) \le s(j)$ or $f(j) \le s(i)$.

<img src="https://cdn.jsdelivr.net/gh/shuyuHU328/picx-images-hosting@master/intervalScheduling.5vmv6e5bjes0.png" alt="intervalScheduling" style="zoom:70%;" />

**Goal**: Select a compatible subset of requests of maximum size.

### Greedy Algorithm

贪心算法是一种在每一步选择中都采取在当前状态下最好或最优（即最有利）的选择，从而希望导致结果是最好或最优的算法。

1. 使用某种方法选择一个request $i$.
2. 拒绝request中冲突的部分
3. 重复这一步骤直到所有的request被处理

### Rules

**Claim 1.** Greedy algorithm outputs a list of intervals
$$
< s(i_1), f(i_1)>, < s(i_2), f(i_2)>, . . . , < s(i_k), f(i_k) >
$$
such that
$$
s\left(i_{1}\right)<f\left(i_{1}\right) \leq s\left(i_{2}\right)<f\left(i_{2}\right) \leq \ldots \leq s\left(i_{k}\right)<f\left(i_{k}\right)
$$
**Claim 2.** Given list of intervals $L$, greedy algorithm with **earliest finish time** produces $k^*$ intervals, where $k^*$ is optimal.

证明可使用数学归纳法，从$k^*=1$的base case归纳，可以得到$k^*$一直是最优的序列，详细过程略。

## Weighted Interval Scheduling

