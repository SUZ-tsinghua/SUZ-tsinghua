---
title: Group Equivariant Deep Learning Lecture 1.7
description: Group convolutions are all you need!
slug: GEDL-notes-1.7
date: 2024-01-23 12:00:00+0800
lastmod: 2024-01-23
math: true
hidden: false
categories:
    - 学习笔记
tags:
    - DL
---

# Lecture 1.7 Group convolutions are all you need!

这一节要证明的是，对于一个线性映射，如果希望它是 equivariant 的，那它必须是一个 group convolution。

## Classical fully connected layer

先看传统的全连接层：

$$\begin{pmatrix} y\_1 \\\\
y\_2 \\\\
y\_3 \\\\
\vdots
\end{pmatrix} = \varphi\left(\begin{pmatrix}
w\_{11} & w\_{12} & w\_{13} & w\_{14} & w\_{15} & \cdots \\\\
w\_{21} & w\_{22} & w\_{23} & w\_{24} & w\_{25} & \cdots \\\\
w\_{31} & w\_{32} & w\_{33} & w\_{34} & w\_{35} & \cdots \\\\
\vdots & \vdots & \vdots & \vdots & \vdots & \ddots
\end{pmatrix}\begin{pmatrix}
x\_1 \\\\
x\_2 \\\\
x\_3 \\\\
x\_4 \\\\
x\_5 \\\\
\vdots
\end{pmatrix} + \begin{pmatrix}
b\_1 \\\\
b\_2 \\\\
b\_3 \\\\
\vdots
\end{pmatrix}\right)$$

其中 $\varphi$ 为某个非线性函数。如果希望全连接层是 equivariant 的，则需要类似有如下的形式：

$$\begin{pmatrix} y\_1 \\\\
y\_2 \\\\
y\_3 \\\\
\vdots
\end{pmatrix} = \varphi\left(\begin{pmatrix}
w\_{1} & w\_{2} & w\_{3} & 0 & 0 & \cdots \\\\
0 & w\_{1} & w\_{2} & w\_{3} & 0 & \cdots \\\\
0 & 0 & w\_{1} & w\_{2} & w\_{3} & \cdots \\\\
\vdots & \vdots & \vdots & \vdots & \vdots & \ddots
\end{pmatrix}\begin{pmatrix}
x\_1 \\\\
x\_2 \\\\
x\_3 \\\\
x\_4 \\\\
x\_5 \\\\
\vdots
\end{pmatrix} + \begin{pmatrix}
b\_1 \\\\
b\_2 \\\\
b\_3 \\\\
\vdots
\end{pmatrix}\right)$$

这其实就是一维的卷积。

## Linear maps in continuous space

对于从 $\mathbb{R}^n$ 到 $\mathbb{R}^m$ 的线性映射，我们知道其必须是 matrix-vector multiplication：

$$y\_j = \sum\_i K\_{i,j} x\_i$$

对于一个 feature map $f\in \mathbb{L}\_2(X)$，我们也可以将其视为一个 vector，维数就是 $f$ 的定义域 $X$ 的大小 $|X|$。考虑将一个 feature map $f^{in}\in \mathbb{L}\_2 (X)$ 线性映射到另一个 feature map $f^{out}\in \mathbb{L}\_2 (Y)$，当定义域是连续的时候，仿照 matrix-vector multiplication，可知线性映射有如下形式：

$$ f^{out}(y) = (Kf)(y) = \int\_X k(y,x) f(x) \mathrm{ d}x $$

对于连续空间中的线性映射，有如下定理：

> Theorem (G-convs are all you need):
>
> $\mathscr{K}: \mathbb{L}\_2(X)\rightarrow \mathbb{L}\_2(Y)$ 是一个线性映射，且 $X$ 和 $Y$ 都是群 $G$ 的 homogeneous space。
>
> 随意取 $y\_0\in Y$，令 $H=Stab\_G(y\_0)$，那么 $Y\equiv G/H$。对于 $\forall y\in Y$，令 $g\_y\in G$ 为某个满足 $y=g\_y y\_0$ 的元素。
>
> $\mathscr{K}$ is equivariant iff:
>
> 1. $\mathscr{K}$ 是一个 group convolution: $[\mathscr{K}f] (y)=\int\_{X} \frac{1}{|\det g\_y|}k(g\_y^{-1}x)f(x)\mathrm{ d}x$
>
> 2. kernel 满足对称约束: $\forall h\in H, \frac{1}{|\det h|}k(h^{-1}x) = k(x)$

定理中的 $|\det g\_y|$ 即为 $g\_y$ 的雅可比行列式的绝对值，理论上来说其会随 $x$ 而变，但在大部分实际情况中，$|\det g\_y| = 1$。

以二维图像的 convolution 为例子来解释一下这个定理。此时 $X$ 与 $Y$ 均为 $\mathbb{R}^2$， $G$ 即为二维 translation group。若希望 linear map 是 equivariant 的，必须有 $[\mathscr{K}f] (y)=\int\_{X} k(x - y)f(x)\mathrm{ d}x$ 这样的卷积形式（平移变换的雅可比行列式大小为1）。

> Proof:
>
> 只证明一边，即 equivariant 的 $\mathscr{K}$ 必须满足定理中的两个条件。
>
> 已知 $\mathscr{K}$ 具有形式 
>
> $$[\mathscr{K}f] (y) = \int\_X \tilde{k} (y,x)f(x)\mathrm{ d} x \tag{1}$$
>
> 希望 $\mathscr{K}$ is equivariant，即
>
> $$\forall g\in G, \forall f\in\mathbb{L}\_2(X),\quad (\mathscr{K}\circ \mathscr{L}\_g^{G\rightarrow \mathbb{L}\_2(X)})(f)=(\mathscr{L}\_g^{G\rightarrow \mathbb{L}\_2(Y)}\circ \mathscr{K})(f)$$
>
> 代入 $\mathscr{K}$ 和 $\mathscr{L}$ 的表达式
>
> $$\int\_{X}\tilde{k}(y,x) f(g^{-1}x)\mathrm{ d}x=\int\_{X}\tilde{k}(g^{-1}y,x) f(x)\mathrm{ d}x$$
>
> 在 RHS 中用 $g^{-1}x$ 替换 $x$
>
> $$\int\_{X}\tilde{k}(y,x) f(g^{-1}x)\mathrm{ d}x=\int\_{X}\tilde{k}(g^{-1}y,g^{-1}x) f(g^{-1}x)\mathrm{ d}(g^{-1}x)$$
>
> 代入雅可比行列式
>
> $$\int\_{X}\tilde{k}(y,x) f(g^{-1}x)\mathrm{ d}x=\int\_{X}\tilde{k}(g^{-1}y,g^{-1}x) f(g^{-1}x)\frac{1}{|\det g|}\mathrm{ d}x$$
>
> 这对任意 $f$ 都要成立，故
>
> $$\tilde{k}(y,x) = \frac{1}{|\det g|} \tilde{k}(g^{-1}y,g^{-1}x)$$
>
> 又已知 $G$ acts transitively on $Y$，$\exists g\_y,$ s.t. $y=g\_y y\_0$
>
> $$\tilde{k}(y,x) = \tilde{k}(g\_y y\_0,x) = \frac{1}{|\det g\_y|} \tilde{k}(y\_0,g\_y^{-1}x)$$
>
> 定义 $k(g\_y^{-1}x) := \tilde{k}(y\_0,g\_y^{-1}x)$，则有
>
> $$\tilde{k}(y,x) = \frac{1}{|\det g\_y|} k(g\_y^{-1}x)$$
>
> 代回到 (1) 式中，就得到了
>
> $$[\mathscr{K}f] (y)=\int\_{X} \frac{1}{|\det g\_y|}k(g\_y^{-1}x)f(x)\mathrm{ d}x \tag{2}$$
>
> $\forall h\in H, k(h^{-1}x) = \tilde{k}(y\_0, h^{-1}x) = |\det h| \tilde{k}(h y\_0, x) = |\det h| k(x)$，即
>
> $$\forall h\in H, \quad \frac{1}{|\det h|}k(h^{-1}x) = k(x) \tag{3}$$

## 例子

我们以几个例子来更好地理解这个定理的内容及应用。

### Classical convolution

当 $G$ 为 translation group 的时候，传统的二维图像卷积 kernel 满足对称约束。因为此时 $H=\\{e\\}$，显然有 $k(ex)=k(x)$。

当 $G$ 为 roto-translation group 的时候，传统的二维图像卷积就不满足对称约束了。对于任意 $y\_0$ ，roto-translation group 中有很多元素可以保持 $y\_0$不变。因此构造出来的 $H$ 中除了 $e$ 还有别的元素，这些元素不能满足 $k(hx)=k(x)$，所以 $G$ 为 roto-translation group 的时候，传统卷积操作不再具有等变性。

### Lifting convolution & Group convolution

Lifting convolution 和 group convolution 的 $Y$ 均为 $SE(2)$，对于 roto-translation group，$H=\\{e\\}$。所以对称约束始终满足，不对 $k$ 有任何约束。二者均具有等变性。

## 结论

因此，当我们希望构造一个 $G$-equivariant convolution 时，最好的办法就是直接让 $Y=G$，这样的话 $H=\\{e\\}$，对称约束始终满足，kernel $k$ 不受任何约束，most expressive。

Group convolutions are all you need!