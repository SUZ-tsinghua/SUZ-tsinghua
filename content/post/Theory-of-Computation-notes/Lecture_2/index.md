---
title: Theory of Computation Lecture 2
description: Mathematical Logic (2)
slug: Theory-of-Computation-Lecture-2
date: 2024-04-02 17:00:00+0800
lastmod: 2024-04-02 17:00:00
math: true
hidden: false
image: cover.png
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

在引入量词的基础上，我们希望跟第一讲中一样，构造一形式系统。我们称之为 *一阶语言 (first order language)*。按照第一讲中定义，形式系统由符号库、合式公式、公理、演绎规则组成。先考察前二者。

The alphabet of symbols:

|Names|Symbols|
|-----|-------|
|变元|$x\_1, x\_2, \cdots$|
|某些（可能没有）个体常元|$a\_1, a\_2, \cdots$|
|某些（可能没有）谓词字母|$A\_i^n$|
|某些（可能没有）函数字母|$f\_i^n$|
|标点符号|( ) ,|
|联结词|$\sim$ $\to$|
|量词|$\forall$|

其中，个体常元即为一个特殊的个体，如“苏格拉底”这样一个指定的人。谓词类似一种关系 $R$，可以被视为一个函数，获取 $n$ 个输入，并返回 $T$ 或 $F$。由于 $\exists$ 可以转化为 $\forall$，我们在符号库中仅使用 $\forall$。

例如：$(\forall x_1)(\forall x_2) A_1^2(f_1^2(x_1,x_2), f_1^2(x_2,x_1))$，若 $A_1^2$ 表示 $=$，$f_1^2$ 表示 $+$，那整个命题就可以写成 $(\forall x_1)(\forall x_2) (x_1+x_2=x_2+x_1)$。

接下来定义合式公式：

> **Definition:**
>
> 令 $\mathscr{L}$ 是一阶语言，$\mathscr{L}$ 中的一个 *项 (term)* 定义如下：
> - 变元和个体常元是项。
> - 如果 $t_1, \cdots, t_n$ 是项，那么 $f_i^n(t_1,\cdots,t_n)$ 是项。
> - 所有项都由上两条规则生成。
> 
> 若 $t_1, \cdots, t_k$ 是 $\mathscr{L}$ 中的项，那么 $A_i^k(t_1, \cdots, t_k)$ 是 $\mathscr{L}$ 中的一个 *原子公式 (atomic fomula)*。
>
> $\mathscr{L}$ 中的 *合式公式 (well-formed formula)* 定义为：
> - 每个原子公式是一个合式公式。
> - 若 $\mathscr{A}, \mathscr{B}$ 是合式公式，那么 $(\sim \mathscr{A}), (\mathscr{A}\to\mathscr{B})$ 和 $(\forall x_i)\mathscr{A}$ （其中 $x_i$ 是任何变元）也是 $\mathscr{L}$ 中的合式公式。
> - 所有合式公式都由上两条规则生成。

我们应注意到 $(\forall x_1)(\mathscr{A}\to\mathscr{B})$ 与 $((\forall x_1)\mathscr{A}\to\mathscr{B})$ 表达的是不同的东西，故而需要引入下面的定义。

> **Definition:**
>
> 在公式 $(\forall x_i)\mathscr{A}$ 中，我们称 $\mathscr{A}$ 是量词的 *辖域 (scope)*。
>
> 变元 $x_i$ 如果出现在 $(\forall x_i)$ 的辖域中，则称它是 *约束的 (bound)*，反之称它是 *自由的 (free)*。

例如 $(\forall \textcolor{red}{x_1})(A_1^2(\textcolor{red}{x_1},x_2)\to (\forall \textcolor{red}{x_2})A_1^1(\textcolor{red}{x_2}))$ 中，标红的即为约束的变元。

现考察变元的替换，在公式 $(\exists x_2)(x_2=x_1(x_1+1))$ 中，我们可以将 $x_1$ 换成 $x_3, f(x_1, x_3)$ 等不包含 $x_2$ 的项，但显然不能换成 $f(x_1, x_2)$ 这种包含 $x_2$ 的项。引入以下定义：

> **Definition:**
>
> 令 $\mathscr{A}$ 是 $\mathscr{L}$ 中的任何公式，我们称项 $t$ 对 $\mathscr{A}$ 中的 $x_i$ 是自由的，如果 $x_i$ 并不自由地出现在 $\mathscr{A}$ 的一个 $(\forall x_j)$ 的辖域中，这里 $x_j$ 是出现在 $t$ 中的任何变元。

在上面的例子中，$t=f(x_1,x_2), x_i=x_1, x_j=x_2$，$x_1$ 自由地出现在 $\mathscr{A}$ 中 $x_2$ 的辖域中，故 $f(x_1,x_2)$ 对 $(\exists x_2)(x_2=x_1(x_1+1))$ 中的 $x_1$ 不是自由的。

显然，对任何 $x_1$ 和 $\mathscr{A}$ 来说，$x_1$ 对 $\mathscr{A}$ 中的 $x_1$ 都是自由的。

### 解释

我们现在希望考察 $\mathscr{L}$ 中的公式什么时候能被称为是 “真” 的。事实上，只有当公式中内容的 “解释” 被给出的时候，我们才能讨论公式的真假。

例如， $(\forall x_1)(\forall x_2)A_1^2(f_1^2(x_1,x_2), f_1^2(x_2,x_1))$。如果我们在自然数的范围内讨论，且认为 $A_1^2$ 代表 $=$，$f_1^2$ 代表 $+$，那么公式为真。但倘若 $f_1^2$ 代表 $-$，那么公式显然是假的。

> **Definition:**
>
> $\mathscr{L}$ 中的一个 *解释 (interpretation)* $I$ 由以下四部分组成：
> - 一个非空集合 $D_I$，即 $I$ 的 *论域 (domain)*。
> - 一个 *特异元素集 (a collection of distinguished elements)* $\bar{a}_i\in D_I$。
> - 一个在 $D_I$ 上的函数集 $\bar{f}_i^n: D_I^n\to D_I$。
> - 一个在 $D_I$ 上的关系集 $\bar{A}_i^n$。

四者分别是对符号表中变元，个体常元，函数字母，谓词字母的具体解释。

例如可以取 $D_I=\\{0,1,2,\cdots\\}, a_1=0, A_1^2$ 表示 $=$, $f_1^2$ 表示 $+$。那么在 $I$ 中，$(\forall x_1) A_1^2(f_1^2(x_1, a_1), x_1)$ 为真。

### 满足，真

> **Definition:**
>
> $I$ 上的一个 *赋值 (valuation)* 是一从 $\mathscr{L}$ 的项集到集合 $D_I$ 的具有下列性质的一个函数 $v$:
> - $v(a_i)=\bar{a}_i$，对 $\mathscr{L}$ 中的每个个体常元 $a_i$。
> - $v(f_i^n(t_1,t_2,\cdots, t_n)) = \bar{f}_i^n(v(t_1), v(t_2), \cdots, v(t_n))$，其中 $f_i^n$ 是 $\mathscr{L}$ 中的任意函数字母，$t_1, \cdots, t_n$ 是 $\mathscr{L}$ 中的任意项。

接下来我们要讨论一个赋值能够使得公式为真。

> **Definition:**
>
> 两个赋值 $v, v^{\prime}$，如果对每个 $j\neq i$，都有 $v(x_j)=v^{\prime}(x_j)$，则称二者是 $i$-等值的。

> **Definition:**
>
> 令 $\mathscr{A}$ 是 $\mathscr{L}$ 的一个公式，$I$ 是 $\mathscr{L}$ 的一个解释，我们称 $I$ 中的一个赋值 $v$ *满足 (satisfies)* $\mathscr{A}$，如果能按如下四个条件归纳地表明 $v$ 满足 $\mathscr{A}$:
> - 如果 $\bar{A}_j^n(v(t_1), \cdots, v(t_n))$ 在 $D_I$ 中为真，那么称 $v$ 满足原子公式 $A_j^n(t_1, \cdots, t_n)$。
> - 如果 $v$ 不满足 $\mathscr{B}$，那么 $v$ 满足 $(\sim\mathscr{B})$。
> - 如果 $v$ 满足 $(\sim \mathscr{B})$ 或 $\mathscr{C}$，那么 $v$ 满足 $(\mathscr{B}\to\mathscr{C})$。
> - 如果每个 $i$-等值于 $v$ 的赋值 $v^{\prime}$ 都满足 $\mathscr{B}$，那么 $v$ 满足 $(\forall x_i)\mathscr{B}$。

> **Definition:**
>
> - 一公式 $\mathscr{A}$ 在解释 $I$ 中称为 *真的 (true)*，如果在 $I$ 中的每个赋值都满足 $\mathscr{A}$。
> - 一公式 $\mathscr{A}$ 在解释 $I$ 中称为 *假的 (false)*，如果 $I$ 中不存在任何满足 $\mathscr{A}$ 的赋值。

在 *真* 的基础上更近一步：

> **Definition:**
>
> - $\mathscr{L}$ 的一个合式公式 $\mathscr{A}$ 称为 *逻辑有效的 (logically valid)*，如果 $\mathscr{A}$ 在 $\mathscr{L}$ 中的每个解释都为真。
> - $\mathscr{L}$ 的一个合式公式 $\mathscr{A}$ 称为 *矛盾的 (contradictory)*，如果 $\mathscr{A}$ 在 $\mathscr{L}$ 中的每个解释都为假。

> **Proposition:**
>
> 如果在一个解释中，公式 $\mathscr{A}$ 和 $(\mathscr{A}\to\mathscr{B})$ 都为真，那么 $\mathscr{B}$ 也为真。
>
> **Proof:**
> 由真和赋值的定义即可证明。

> **Proposition:**
> 
> 在一个解释中，公式 $\mathscr{A}$ 为真，当且仅当 $(\forall x_i) \mathscr{A}$ 为真，其中 $x_i$ 是任意变元。
>
> **Proof:**
> 由定义。

例如可以取 $D_I=\\{0,1,2,\cdots\\}, A_1^2$ 表示 $=$, $f_1^2$ 表示 $+$。那么 
$$(\forall x_1)(\forall x_2) A_1^2(f_1^2(x_1,x_2),f_1^2(x_2,x_1))$$
$$(\forall x_2) A_1^2(f_1^2(x_1,x_2),f_1^2(x_2,x_1))$$
$$A_1^2(f_1^2(x_1,x_2),f_1^2(x_2,x_1))$$
都为真。

> **Definition:**
> 
> $\mathscr{L}$ 中的一个公式 $\mathscr{A}$ 称为 *闭的 (closed)*，如果没有变元在 $\mathscr{A}$ 中自由出现。

例如，$(\forall x_1)(\forall x_2) A_1^2(f_1^2(x_1,x_2),f_1^2(x_2,x_1))$ 是闭的，$(\forall x_1) A_1^1(x_1)\to A_1^1(x_1)$ 不是闭的。

> **Proposition:**
>
> 如果 $\mathscr{A}$ 是 $\mathscr{L}$ 中的闭公式，且 $I$ 是 $\mathscr{L}$ 的一个解释，那么 $\mathscr{A}$ 在 $I$ 中非真即假。
>
> **Proof:**
>
> 由于所有变量都是约束的，所以可以根据 *满足* 的定义的第四条得知，一个赋值满足 $\mathscr{A}$ 当且仅当所有赋值满足 $\mathscr{A}$。

## Formal Predicate Calculus (形式的谓词演算)

### 形式系统 $K_{\mathscr{L}}$

在 [# 一阶语言](#一阶语言) 中我们说到，形式系统需由符号库、合式公式、公理、演绎规则组成，并定义了前二者。接下来我们定义后二者，并由此得到一个形式系统 $K_{\mathscr{L}}$。

> **Definition:**
>
> 令 $\mathscr{A},\mathscr{B},\mathscr{C}$ 是 $\mathscr{L}$ 中的任意公式，以下是 $K_{\mathscr{L}}$ 的公理：
> - (K1) $(\mathscr{A}\to(\mathscr{B}\to\mathscr{A}))$.
> - (K2) $((\mathscr{A}\to(\mathscr{B}\to\mathscr{C}))\to ((\mathscr{A}\to\mathscr{B})\to(\mathscr{A}\to\mathscr{C})))$.
> - (K3) $(((\sim \mathscr{A})\to(\sim \mathscr{B}))\to (\mathscr{B}\to\mathscr{A}))$.
> - (K4) $((\forall x_i) \mathscr{A}\to \mathscr{A})$，如果 $x_i$ 不在 $\mathscr{A}$ 中自由出现。
> - (K5) $((\forall x_i) \mathscr{A}(x_i)\to \mathscr{A}(t))$，如果 $t$ 是 $\mathscr{L}$ 中的一个项，并且在 $\mathscr{A}(x_i)$ 中对 $x_i$ 自由。
> - (K6) $(\forall x_i)(\mathscr{A}\to\mathscr{B})\to(\mathscr{A}\to(\forall x_i)\mathscr{B})$，如果 $x_i$ 不在 $\mathscr{A}$ 中自由出现。
>
> 演绎规则：
> - MP：从 $\mathscr{A}$ 和 $(\mathscr{A}\to\mathscr{B})$ 可以演绎出 $\mathscr{B}$。
> - Generalization：从 $\mathscr{A}$ 可以演绎出 $(\forall x_i)\mathscr{A}$，其中 $x_i$ 是任意变元。

事实上，如果 $x_i$ 在 $\mathscr{A}$ 中自由出现，因为 $x_i$ 在 $\mathscr{A}$ 中对 $x_i$ 自由，可以由(K5) 知道，(K4) 仍然成立。所以，无论 $x_i$ 是否在 $\mathscr{A}$ 中自由出现，都有 $((\forall x_i) \mathscr{A}\to \mathscr{A})$。

> **Definition:**
>
> $K_{\mathscr{L}}$ 中的一个 *证明 (proof)* 是指 $\mathscr{L}$ 的公式的这样一个序列 $\mathscr{A}_1, \cdots, \mathscr{A}_n$，使得对于每个 $i (1\leq i\leq n)$，$\mathscr{A}_i$ 或者是 $K\_{\mathscr{L}}$ 的一个公理，或者是由位于 $\mathscr{A}_i$ 之前的公式通过 MP 或 Generalization 规则得到的。最后的公式 $\mathscr{A}_n$ 称为 $K\_{\mathscr{L}}$ 中的一条 *定理 (theorem)*，记作 $\vdash\_{K\_{\mathscr{L}}} \mathscr{A}_n$。
>
> 如果 $\Gamma$ 是 $\mathscr{L}$ 中的一组合式公式，在 $K\_{\mathscr{L}}$ 中从 $\Gamma$ 的一个 *演绎 (deduction)* 是指公式的这样一个序列 $\mathscr{A}_1, \cdots, \mathscr{A}_n$，使得对于每个 $i (1\leq i\leq n)$，$\mathscr{A}_i$ 或者是 $K\_{\mathscr{L}}$ 的一个公理，或者是由位于 $\mathscr{A}_i$ 之前的公式通过 MP 或 Generalization 规则得到，或者是 $\Gamma$ 中的一个成员。记作 $\Gamma \vdash\_{K\_{\mathscr{L}}} \mathscr{A}_n$。

以下用 $K$ 代替 $K_{\mathscr{L}}$。

> **Proposition (Soundness):**
>
> (K1)-(K6) 都是逻辑有效的。这可以用定义证明。
>
> 由此可归纳出，$K$ 中的所有定理都是逻辑有效的。

同样，$K$ 中也有 *演绎定理*，但不同于 $L$ 中的演绎定理，我们先来看一个特殊情况：

> **Proposition:**
> 
> 我们知道对于 $K$ 中的任意公式 $\mathscr{A}$，都有 $\\{\mathscr{A}\\}\vdash_{K} (\forall x_i)\mathscr{A}$，但是 $\vdash_{K} (\mathscr{A} \to (\forall x_i)\mathscr{A})$ 并不是必然的。
>
> **Proof:**
>
> 例如 $\\{(x_1 = 0)\\}\vdash_{K} (\forall x_1) (x_1 = 0)$，但是并没有 $\vdash_{K} ((x_1 = 0) \to (\forall x_1)(x_1 = 0))$。因为由 *满足* 定义的第四条可知，不存在赋值满足 $(\forall x_1)(x_1 = 0)$，所以 $((x_1 = 0) \to (\forall x_1)(x_1 = 0))$ 不是逻辑有效的，因此不是 $K$ 中的定理。

> **Proposition ($K$ 的演绎定理，the deduction theorem)**
>
> 如果 $\Gamma\cup \\{\mathscr{A}\\} \vdash_{K}\mathscr{B}$，并且演绎过程中不涉及 $\mathscr{A}$ 中自由的变元，那么 $\Gamma\vdash_{K}(\mathscr{A}\to\mathscr{B})$。
>
> **Proof:**
> 类似于 $L$ 中演绎定理的证明，对 $\mathscr{B}$ 进行分类讨论。

由演绎定理可得推论：

> **Proposition:**
>
> 如果 $\Gamma\cup \\{\mathscr{A}\\} \vdash_{K}\mathscr{B}$，并且 $\mathscr{A}$ 是闭公式，那么 $\Gamma\vdash_{K}(\mathscr{A}\to\mathscr{B})$。

演绎定理的逆定理始终成立，无需加条件：

> **Proposition:**
>
> 如果 $\Gamma\vdash_{K}(\mathscr{A}\to\mathscr{B})$，那么 $\Gamma\cup \\{\mathscr{A}\\} \vdash_{K}\mathscr{B}$。

$K$ 中同样有三段论：

> **Proposition (HS):**
>
> 对任意公式 $\mathscr{A}, \mathscr{B}, \mathscr{C}$，如果 $\\{(\mathscr{A}\to\mathscr{B}), (\mathscr{B}\to\mathscr{C})\\}\vdash_{K}(\mathscr{A}\to\mathscr{C})$。

同样有替换定理：

> **Proposition:**
>
> $\mathscr{A}$ 和 $\mathscr{B}$ 是 $\mathscr{L}$ 中的闭公式，如果 $\mathscr{B}_0$ 是由 $\mathscr{A}_0$ 通过将其中的 $\mathscr{A}$ 替换成 $\mathscr{B}$ 得到的，那么有：
> $$\vdash\_{K}(\mathscr{A}\leftrightarrow \mathscr{B})\to (\mathscr{A}_0\leftrightarrow \mathscr{B}_0)$$

> **Proposition:**
>
> 如果 $x_i$ 不在 $\mathscr{A}$ 中自由出现，那么
> - $\vdash_{K}(\forall x_i)(\mathscr{A}\to \mathscr{B})\leftrightarrow (\mathscr{A}\to(\forall x_i)\mathscr{B})\$
> - $\vdash_{K}(\exists x_i)(\mathscr{A}\to \mathscr{B})\leftrightarrow (\mathscr{A}\to(\exists x_i)\mathscr{B})\$
>
> 如果 $x_i$ 不在 $\mathscr{B}$ 中自由出现，那么
> - $\vdash_{K}(\forall x_i)(\mathscr{A}\to \mathscr{B})\leftrightarrow ((\exists x_i)\mathscr{A}\to\mathscr{B})\$
> - $\vdash_{K}(\exists x_i)(\mathscr{A}\to \mathscr{B})\leftrightarrow ((\forall x_i)\mathscr{A}\to\mathscr{B})\$

### 前束范式

> **Definition:**
>
> $\mathscr{L}$ 中的一个公式 $\mathscr{A}$ 称为 *前束范式 (prenex form)*，如果它形如:
> $$(Q_1 x_{i1})(Q_2 x_{i2})(Q_k x_{ik})\mathscr{D}$$
> 其中 $Q_j$ 是 $\forall$ 或 $\exists$，$\mathscr{D}$ 是不带量词的公式。

> **Proposition:**
>
> $\mathscr{L}$ 中的任意合式公式 $\mathscr{A}$，总存在前束范式 $\mathscr{B}$ 等价于 $\mathscr{A}$。
>
> **Proof:**
> 由上一条 proposition 可证。

> **Definition:**
>
> 一个前束范式是一个 $\Pi_n$ 式，如果它以 $\forall$ 开头，并有 $n-1$ 次量词交叉。
>
> 一个前束范式是一个 $\Sigma_n$ 式，如果它以 $\exists$ 开头，并有 $n-1$ 次量词交叉。

比如 $(\exists x_3)(\forall x_1)(\forall x_4)(\forall x_5) A(x_1,x_2,x_3,x_4,x_5)$ 就是 $\Sigma_2$ 式。

### $K_{\mathscr{L}}$ 的完备性定理

与 $L$ 一样， $K$ 也满足 soundness, consistency, adequacy，因此 $K$ 中的定理 $\iff$ 逻辑有效。

> **Proposition (Adequacy, 完备性):**
>
> 所有逻辑有效的公式都是 $K$ 中的定理。也被称为 *一阶逻辑的哥德尔完备性定理*。
>
> **Proof:**
> 过于复杂不便展示。

### 模型

> **Definition:**
>
> 如果 $S$ 是一个一阶逻辑系统，$S$ 的一个 *模型 (model)* 指的是一个解释，使得 $S$ 中的所有定理都为真。

> **Theorem:**
>
> 如果 $I$ 是一个解释，使得 $S$ 的所有公理都为真，那么 $I$ 是 $S$ 的一个模型。

## 数学系统

现在要在 $K$ 的基础上加些数学。

### 一阶算术

> **Definition:**
>
> *一阶算术 $\mathscr{N}$ (first order arithmetic)* 是 $K$ 的一个一致扩充，额外引入了以下公理：
> - (E7) $x_1=x_1$
> - (E8) $t_k=u\to f_i^n(t_1,\cdots,t_k,\cdots,t_n)=f^n_i(t_1,\cdots,u,\cdots,t_n)$
> - (E9) $t_k=u\to (A_i^n(t_1,\cdots,t_k,\cdots,t_n)\to A^n_i(t_1,\cdots,u,\cdots,t_n))$
> - (N1) $(\forall x_1) \sim (x_1^{\prime}=0)$
> - (N2) $(\forall x_1)(\forall x_2)(x_1^{\prime}=x_2^{\prime} \to x_1=x_2)$
> - (N3) $(\forall x_1) (x_1+0=x_1)$
> - (N4) $(\forall x_1)(\forall x_2)((x_1+x_2^{\prime})=(x_1+x_2)^{\prime})$
> - (N5) $(\forall x_1)(x_1\times 0=0)$
> - (N6) $(\forall x_1)(\forall x_2)((x_1\times x_2^{\prime})=(x_1\times x_2)+x_1)$
> - (N7) $\mathscr{A}(0) \to ((\forall x_1)(\mathscr{A}(x_1)\to\mathscr{A}(x_1^{\prime}))\to (\forall x_1)\mathscr{A}(x_1))$，其中 $x_1$ 在 $\mathscr{A}(x_1)$ 中自由出现。

只有 0 是 $\mathscr{N}$ 中的符号。而 $1, 2, 3, \cdots$ 被表示为 $0^{\prime}, 0^{\prime\prime}, 0^{\prime\prime\prime}, \cdots$

## Gödel incompleteness theorem (哥德尔不完全性定理)

这部分先放着吧，有兴趣有必要了再写。