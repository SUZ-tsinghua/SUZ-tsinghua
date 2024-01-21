---
title: Group Equivariant Deep Learning Lecture 1
description: Regular group convolutional neural networks
slug: GEDL-notes-1
date: 2024-01-20
math: true
hidden: false
categories:
    - 学习笔记
tags:
    - DL
---

最近在做一个将对称性与 RL 结合的工作，其中将 PPO 里的 actor-critic 网络换成了具有对称性的 EMLP (equivariant MLP)。EMLP 的效果就是，对于具有对称性的输入，其保证输出也具有对称性。EMLP 利用 [escnn](https://github.com/QUVA-Lab/escnn) 库实现。我在 escnn 库的 README.md 里找到了 Amsterdam 大学的一个暑期课程 [An Introduction to Group Equivariant Deep Learning](https://uvagedl.github.io/)，感觉是个很有趣的领域，于是打算学习一下并记点笔记。

# Lecture 1 Regular group convolutional neural networks

## Lecture 1.1 Introduction

其实 DL 中的许多问题都要求对于以某种方式对称后的输入，网络的输出也具有某种对称性（或不变性）。比如对于肿瘤细胞的识别：给定一张细胞的图片，要求判断其是否为恶性肿瘤细胞。我们希望图像旋转后，判断的结果保持不变。

![细胞图，第二个由第一个旋转得来](cells.png)

一个最直接的办法就是 data augmentation，对于训练集中的一张图片，将其经过若干种旋转后的图片都加到训练集中。尽管这样可以在一定程度上解决问题，但是这种方法仍没有完全保证输出关于对称输入的不变性 (invariance)，而且其将有限的网络 capacity 用于学习对称性上，在相同的参数量下可能会造成 capacity 的下降。因此我们希望直接在网络层面保证对称性。

我们已经知道 CNN 具有平移对称性，即对于平移一定距离的输入，CNN 的输出也具有相同方向、相同距离的平移。但如下图所示， CNN 并不具备旋转对称性。其中 input 即为输入的图像，feature map 是 CNN 的原始输出，stabilized view 是将 feature map 转回到原来的角度。对于旋转后的图像，CNN 的输出并不稳定，即 stabilized view 并不保持不变。

![Conventional CNN output](conventional_cnn.gif)

我们希望构造一个 CNN 网络结构 (Group equivariant CNN)，使得 CNN 具有旋转对称性，如下图所示。

![Equivariant CNN output](vectorfield.gif)

## Lecture 1.2 Group Theory | The basics

先介绍一下群论里的一些概念。

### Group

一个群 (group) $(G, \cdot)$ 是一个元素集合 $G$ 加上定义在 $G$ 中元素上的二元运算 $\cdot$。二元运算 $\cdot$ 满足以下四条性质：
- **Closure:** 对于 $\forall g, h \in G$, 它们的积 $g\cdot h\in G$。
- **Associativity:** 对于 $\forall g,h,i \in G, \cdot$ 满足结合律，$g\cdot (h\cdot i)=(g\cdot h)\cdot i$。
- **Indentity element:** 存在一个 identity element $e\in G$，满足对于 $\forall g\in G, e\cdot g=g\cdot e=g$。
- **Inverse element:** 对于 $\forall g\in G$，其都有 inverse element $g^{-1}\in G$ s.t. $g^{-1}\cdot g=g \cdot g^{-1}=e$。

#### Translation group $(\mathbb{R}^2, +)$

Translation group 中的元素即为 $\mathbb{R}^2$ 中的点，对于 $g=(\mathbf{x}), g^{\prime}=(\mathbf{x}^{\prime})$，其中 $\mathbf{x},\mathbf{x}^{\prime}\in \mathbb{R}^2$，我们有：

$$g\cdot g^{\prime}=(\mathbf{x}+\mathbf{x}^{\prime})$$
$$g^{-1}=(-\mathbf{x})$$

Translation group 中的一个元素可以被视为一个平移变换 （Lecture 1.6 中会细讲）。

![translation group](translation_group.png)

#### Roto-translation group $SE(2)$ (2D Special Euclidean motion group)

$SE(2)=\mathbb{R}^2\rtimes SO(2)$，其中的每个元素 $g=(\mathbf{x}, \mathbf{R}\_{\theta} )$ 由一个二维向量 $\mathbf{x}\in \mathbb{R}^2$ 以及一个二维旋转矩阵 $\mathbf{R}\_{\theta} \in SO(2)$ 组成。对于 $g=(\mathbf{x}, \mathbf{R}\_{\theta}) , g^{\prime}=(\mathbf{x}^{\prime}, \mathbf{R}\_{\theta^{\prime}} )$，我们有：

$$g\cdot g^{\prime}=(\mathbf{x}, \mathbf{R}\_{\theta}) \cdot (\mathbf{x}^{\prime}, \mathbf{R}\_{\theta^{\prime}})=(\mathbf{R}\_{\theta}\mathbf{x}^{\prime}+\mathbf{x}, \mathbf{R}\_{\theta+\theta^{\prime}} )$$

$$g^{-1}=(-\mathbf{R}\_{\theta}^{-1} \mathbf{x}, \mathbf{R}\_{\theta}^{-1} )$$

$SE(2)$ 中的一个元素可以被视为一个旋转变换加上一个平移变换。

![SE(2) group](SE(2).png)

- **矩阵表示：** $SE(2)$ 中的元素也可以用矩阵表示：

$$g=(\mathbf{x}, \mathbf{R}\_{\theta})\quad \leftrightarrow\quad \mathbf{G}=\begin{pmatrix} \mathbf{R}\_{\theta} & \mathbf{x}\\\\ \mathbf{0}^{\top} & 1 \end{pmatrix} = \begin{pmatrix} \cos\theta & -\sin\theta & x\\\\ \sin\theta & \cos\theta & y\\\\ 0 & 0 & 1 \end{pmatrix}$$

$$\begin{pmatrix} \mathbf{R}\_{\theta} & \mathbf{x}\\\\ \mathbf{0}^{\top} & 1 \end{pmatrix}\begin{pmatrix} \mathbf{R}\_{\theta^{\prime}} & \mathbf{x}^{\prime}\\\\ \mathbf{0}^{\top} & 1 \end{pmatrix}=\begin{pmatrix} \mathbf{R}\_{\theta+\theta^{\prime}} & \mathbf{R}\_{\theta} \mathbf{x}^{\prime}+\mathbf{x}\\\\ \mathbf{0}^{\top} & 1 \end{pmatrix}$$

#### Scale-translation group $\mathbb{R}^2\rtimes\mathbb{R}^+$

$\mathbb{R}^2\rtimes\mathbb{R}^+$ 中的每个元素 $g=(\mathbf{x},s)$ 由一个二维向量 $\mathbf{x}\in\mathbb{R}^2$ 以及一个正标量 $s\in \mathbb{R}^+$ 组成。对于 $g=(\mathbf{x},s), g^{\prime}=(\mathbf{x}^{\prime}, s^{\prime})$，我们有：

$$g\cdot g^{\prime}=(s\mathbf{x}^{\prime}+\mathbf{x}, ss^{\prime})$$

$$g^{-1}=\left(-\frac{1}{s}\mathbf{x}, \frac{1}{s}\right)$$

$\mathbb{R}^2\rtimes\mathbb{R}^+$ 中的一个元素可以被视为一个缩放变换加上一个平移变换。

#### Affine groups $G=\mathbb{R}^{d}\rtimes H$

Affine groups 是某个 group $H$ 和 $\mathbb{R}^d$ 的半直积。对于 $g=(\mathbf{x},h), g^{\prime}=(\mathbf{x}^{\prime}, h^{\prime})$，我们有：

$$g\cdot g^{\prime}=(h\cdot\mathbf{x}^{\prime}+\mathbf{x}, h\cdot h^{\prime})$$
$$g^{-1}=\left(-h^{-1}\cdot\mathbf{x}, h^{-1}\right)$$

上面说的 $SE(2)=\mathbb{R}^2\rtimes SO(2)$ 和 $\mathbb{R}^2\rtimes\mathbb{R}^+$ 都属于 affine groups。Affine groups 中的一个元素可以被视为一个变换 $h$ 加上一个平移变换。

### Representations

一个 representation $\rho: G\rightarrow GL(V)$ 是从 $G$ 到 $GL(V)$ 的一个群同态 (group homomorphism)。

- $GL_n$ 就是由所有 $n\times n$ 的可逆矩阵组成的群，群运算即为矩阵乘法。

这就是说 $\rho(g)$ 是由 $g\in G$ 决定的线性变换，它作用在一个向量 $\mathbf{v}\in V$ 上，具有性质：

$$\rho(g^{\prime})\circ\rho(g)[\mathbf{v}]=\rho(g^{\prime}\cdot g)[\mathbf{v}]$$


![representation is homomorphic](representation.png)


#### Left-regular representations

一个 left-regular representation $\mathscr{L}_g$ 作用在一个函数 $f$ 上，定义为：

$$\mathscr{L}\_g \[f\](x):=f(g^{-1}\cdot x)$$

这么说有点抽象，举个例子。$f\in \mathbb{L}_2(\mathbb{R}^2)$ 是一个二维图像，它给 $\mathbb{R}^2$ 上的每个点赋值。$g\in G=SE(2)$ 可以被看做是一个平移变换加上一个旋转变换。对于二维平面上的一个点 $\mathbf{y}\in\mathbb{R}^2$，可以算出：

$$\mathscr{L}\_g \[f\](\mathbf{y}):=f(g^{-1}\cdot \mathbf{y})=f(\mathbf{R}\_{-\theta}(\mathbf{y}-\mathbf{x}))$$

即现在 $\mathbf{y}$ 处的函数值为先将 $\mathbf{y}$ 平移 $-\mathbf{x}$，再旋转 $-\theta$ 处的 $f$ 值。也就是说，现在的图像是由原先的图像先旋转 $\theta$ 再平移 $\mathbf{x}$ 得到的。在这种情况下 $\mathscr{L}\_g $ 也可以写做 $\mathscr{L}\_g^{SE(2)\rightarrow \mathbb{L}\_2 (\mathbb{R}^2)}$。这就是说，在给定 $f$ 的情况下，$\mathscr{L}$ 会将 $g\in SE(2)$ 转成一个二维图像 $f^{\prime}\in \mathbb{L}\_2 (\mathbb{R}^2)$。

Left-regular representations 满足性质：

$$\mathscr{L}\_{g^{\prime}}\circ\mathscr{L}\_g=\mathscr{L}\_{g^{\prime}\cdot g}$$

![left regular representation](left_regular_rep.png)


### Equivariance

如果一个 operator $\Phi: X\rightarrow Y$ （如一个神经网络）满足以下条件，则说其具有等变性 (equivariance)：

$$\Phi\circ \rho^X(g)=\rho^Y(g)\circ \Phi$$

即，先作用 $\Phi$ 与后作用 $\Phi$ 得到的效果是一样的。比如 $g$ 表示一个平移变换，$\Phi$ 表示用 CNN 从图片中提取特征。

![equivariance](equivariance.png)

## Lecture 1.3 Regular group convolutions | Template matching viewpoint

### Cross-correlations

定义 kernel $k\in \mathbb{L}\_2(\mathbb{R}^2)$ 和二维图像 $f\in \mathbb{L}\_2(\mathbb{R}^2)$ 之间的 cross-correlations 为：

$$(k\star\_{\mathbb{R}^2} f)(\mathbf{x})=\int\_{\mathbb{R}^2} k(\mathbf{x}^{\prime}-\mathbf{x})f(\mathbf{x}^{\prime})\mathrm{ d} \mathbf{x}^{\prime}=(\mathscr{L}\_{\mathbf{x}} k, f)\_{\mathbb{L}\_2 (\mathbb{R}^2)}$$

RHS 的标记是定义出来的。

- **Note:** 这其实就是 CNN 中的 convolution，但并不是严格意义上的 convolution，严格意义上的 convolution 为：

$$\int\_{\mathbb{R}^2} k(\mathbf{x}-\mathbf{x}^{\prime})f(\mathbf{x}^{\prime})\mathrm{ d} \mathbf{x}^{\prime}$$

### Equivariance

Convolutions/cross-correlations 具有平移等变性，即对于 $\forall \mathbf{x}, \mathbf{y}\in \mathbb{R}^2$，有

$$\mathscr{L}\_{\mathbf{y}}^{\mathbb{R}^2\rightarrow \mathbb{L}\_2(\mathbb{R}^2)}[(k\star\_{\mathbb{R}^2}f)(\mathbf{x})]=(k\star\_{\mathbb{R}^2}\mathscr{L}\_{\mathbf{y}}^{\mathbb{R}^2\rightarrow \mathbb{L}\_2(\mathbb{R}^2)}f)(\mathbf{x})$$

![卷积操作具有平移等变性](trans-equi.png)

> Proof: 
> $$LHS=(k\star\_{\mathbb{R}^2}f)(\mathbf{x}-\mathbf{y})=(\mathscr{L}\_{\mathbf{x}-\mathbf{y}}^{\mathbb{R}^2\rightarrow \mathbb{L}\_2(\mathbb{R}^2)} k, f)_{\mathbb{L}_2 (\mathbb{R}^2)}=(\mathscr{L}\_{\mathbf{x}}^{\mathbb{R}^2\rightarrow \mathbb{L}\_2(\mathbb{R}^2)} k, \mathscr{L}\_{\mathbf{y}}^{\mathbb{R}^2\rightarrow \mathbb{L}\_2(\mathbb{R}^2)} f)_{\mathbb{L}_2 (\mathbb{R}^2)}=RHS$$

一般情况下，卷积操作对于旋转操作并不具有等变性。

$$\mathscr{L}\_{\theta}^{SO(2)\rightarrow \mathbb{L}\_2(\mathbb{R}^2)}[(k\star\_{\mathbb{R}^2}f)(\mathbf{x})]\neq (k\star\_{\mathbb{R}^2}\mathscr{L}\_{\theta}^{SO(2)\rightarrow \mathbb{L}\_2(\mathbb{R}^2)}f)(\mathbf{x})$$

![卷积操作不具有旋转等变性](roto-trans-equi.png)

### Regular group CNN

接下来，我们从构造 roto-translation equivariant 的卷积操作入手，一步步搭建 regular group CNN。

#### Lifting correlations

首先，cross-correlation 可以写为：

$$(k\star\_{\mathbb{R}^2} f)(\mathbf{x})=(\mathscr{L}\_{\mathbf{x}}^{\mathbb{R}^2\rightarrow \mathbb{L}\_2 (\mathbb{R}^2)} k, f)\_{\mathbb{L}\_2 (\mathbb{R}^2)}$$

类似地，对于 $\forall k, f\in \mathbb{L}\_2 (\mathbb{R}^2)$，我们定义 lifting correlations 为：

$$(k\tilde{\star} f)(\mathbf{x}, \theta)=(\mathscr{L}\_{(\mathbf{x},\theta)}^{SE(2)\rightarrow \mathbb{L}\_2 (\mathbb{R}^2)} k, f)\_{\mathbb{L}\_2 (\mathbb{R}^2)}=(\mathscr{L}\_{\mathbf{x}}^{\mathbb{R}^2\rightarrow \mathbb{L}\_2 (\mathbb{R}^2)} \mathscr{L}\_{\theta}^{SO(2)\rightarrow \mathbb{L}\_2 (\mathbb{R}^2)} k, f)\_{\mathbb{L}\_2 (\mathbb{R}^2)}$$

这就相当于将 kernel $k$ 的每一种旋转以及每一种平移都与 $f$ 做一次点积操作，最后得到一个 3D feature map。

![Lifting correlations](lifting_correlation.png)

Lifing correlations 具有 roto-translation 等变性，即（省略上标）：

$$\mathscr{L}\_{\mathbf{y}} \mathscr{L}\_{\varphi} (k\tilde{\star} f)(\mathbf{x}, \theta) = (k\tilde{\star} \mathscr{L}\_{\mathbf{y}} \mathscr{L}\_{\varphi} f)(\mathbf{x}, \theta)$$

> Proof: 
> $$LHS=(k\tilde{\star} f)\left((\mathbf{y},\varphi)^{-1} (\mathbf{x},\theta)\right)=(\mathscr{L}\_{\mathbf{y}} \mathscr{L}\_{\varphi} \mathscr{L}\_{\mathbf{x}} \mathscr{L}\_{\theta} k, f)_{\mathbb{L}_2 (\mathbb{R}^2)}=(\mathscr{L}\_{\mathbf{x}} \mathscr{L}\_{\theta} k, \mathscr{L}\_{\mathbf{y}} \mathscr{L}\_{\varphi} f)_{\mathbb{L}_2 (\mathbb{R}^2)}=RHS$$

![lifting correlations are roto-translation equivariant](lift-cor-roto-equi.png)

#### Group correlations

Lifting correlations 可以学到 low-level features，比如要匹配人脸，某个 kernel 可能学到了眼睛，某个 kernel 可能学到了鼻子。如果此时直接对 $\theta$ 轴进行投影，各部分不同的朝向可能会导致相同的投影，如下图所示。

![直接投影可能会导致很多奇怪的图被投影到同样的 feature map](faces.png)

所以需要后续的 layers 用其他 kernels 来匹配全局信息。对于 $\forall k, y \in \mathbb{L}\_2 (SE(2))$，定义 group correlations 为：

$$(k\star f)(\mathbf{x}, \theta)=(\mathscr{L}\_{(\mathbf{x},\theta)}^{SE(2)\rightarrow \mathbb{L}\_2 (SE(2))} k, f)\_{\mathbb{L}\_2 (SE(2))}=(\mathscr{L}\_{\mathbf{x}}^{\mathbb{R}^2\rightarrow \mathbb{L}\_2 (SE(2))} \mathscr{L}\_{\theta}^{SO(2)\rightarrow \mathbb{L}\_2 (SE(2))} k, f)\_{\mathbb{L}\_2 (SE(2))}$$

Group correlations 将两个 3D feature maps 映射为一个 3D feature map。

![Group correlations](group_correlation.png)

类似可以证明，group correlations 也是 roto-translation equivalent 的。

![group correlations are roto-translation equivariant](group-cor-roto-equi.png)