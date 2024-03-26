---
title: Group Equivariant Deep Learning Lecture 2.1
description: Steerable kernels/basis functions
slug: GEDL-notes-2.1
date: 2024-01-24 10:00:00+0800
lastmod: 2024-01-24
math: true
hidden: true
categories:
    - 学习笔记
tags:
    - DL
---

# Lecture 2 Steerable group convolutions

Lecture 2 会将 Lecture 1 中的 G-CNN 进行拓展，使其不只是局限于 discreate group。

# Lecture 2.1 Steerable kernels/basis functions

## Steerable basis

先介绍一下 steerable basis。当一个由很多基函数 $Y\_l \in \mathbb{L}\_2(X)$ 组成的向量 $Y(x)=\begin{pmatrix} \vdots \\\\ Y\_l(x) \\\\ \vdots \end{pmatrix}\in\mathbb{K}^L$ 满足以下条件时，我们称 $Y(x)$ is steerable：

$$\forall g\in G,\quad Y(gx)=\rho(g)Y(x)$$

其中，$gx$ 是 $G$ 在 $X$ 上的群作用，$\rho(g)\in\mathbb{K}^{L\times L}$ 是 $G$ 的一个 representation。

Steerable basis 的意思是，要计算 $Y(gx)$ 的值，只需要对 $Y(x)$ 作用一个线性变换 $\rho(g)$ 即可。

## Example: Steerable basis on $S^1$ (circular harmonics)

