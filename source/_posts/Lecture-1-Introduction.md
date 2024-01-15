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
mathjax: false
---

## P & NP

- **P**: class of problems solvable in polynomial time. *O*($n^k$) for some constant *k*. 

  Shortest paths in a graph.

- **NP**: class of problems verifiable in polynomial time. 

​	Verifying a cycle is hamiltonian.

- **NP-complete**: problem is in NP and is as hard as any problem in NP.

  如果NPC问题可在多项式时间内被解决，那么所有的NP问题都可以在多项式时间内被解决

## Interval Scheduling / 区间调度问题

Requests $1,2, . . . , n$, single resource

$s(i)$ start time, $f(i)$ finish time, $s(i) < f(i)$ (start time must be less than finish time for a request)

Two requests $i$ and $j$ are compatible if they don’t overlap, i.e., $f(i) ≤ s(j)$ or $f(j) ≤ s(i)$.

<img src="https://cdn.jsdelivr.net/gh/shuyuHU328/picx-images-hosting@master/intervalScheduling.5vmv6e5bjes0.png" alt="intervalScheduling" style="zoom:70%;" />

**Goal**: Select a compatible subset of requests of maximum size.

## Weighted Interval Scheduling

