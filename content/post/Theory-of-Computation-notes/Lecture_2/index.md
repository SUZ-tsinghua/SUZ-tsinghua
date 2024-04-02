---
title: Theory of Computation Lecture 2
description: Mathematical Logic (2)
slug: Theory-of-Computation-Lecture-2
date: 2024-04-02 17:00:00
lastmod: 2024-04-02 17:00:00
math: true
hidden: false
categories:
    - 课程笔记
tags:
    - Theory of Computation
---

# Lecture 2 Mathematical Logic (2)

## Informal Predicate Calculus (非形式的谓词演算)

### 量词

对于某些命题，我们无法用第一节课中的方式将其表达为一命题形式，如“所有人都会死”，此时我们必须要引入量词来表达“所有”这样的限制语义：

> **Definition:**
>
> *全称量词 (Universal quantifier)*，对所有的 $x$，$(\forall x)$。
>
> *存在量词 (Existensial quantifier)*，存在 $x$，$(\exists x)$。

“所有人都会死”就可以表示成 $(\forall x)(A(x)\to M(x))$。
此处，$x$ 为一变元，指的是所有的东西，并不只限于人。$A(x)$ 表示 $x$ 是人，$M(x)$ 表示 $x$ 会死。

$\forall$ 和 $\exists$ 之间也可以相互转换。考察句子“不是所有鸟都会飞”，可以表示为 $\sim(\forall x)(B(x)\to F(x))$。句子显然等价于“有些鸟不会飞”，可以表示为 $(\exists x)(B(x)\land \sim F(x))$。

我们知道：

$$
\begin{align*}
\sim(\forall x)(B(x)\to F(x)) &\iff \sim(\forall x)(\sim B(x)\lor F(x))\\\\
& \iff \sim(\forall x)\sim (B(x)\land \sim F(x))
\end{align*}
$$

对比 $\sim(\forall x)\sim (B(x)\land \sim F(x))$ 与 $(\exists x)(B(x)\land \sim F(x))$，不难发现二者有相似之处。事实上，$(\exists x)\mathscr{A} \iff \sim(\forall x)\sim \mathscr{A}$。

### 一阶语言

