---
title: Theory of Computation Lecture 1
description: Mathematical Logic (1)
slug: Theory-of-Computation-Lecture-1
date: 2024-03-25 17:00:00
lastmod: 2024-03-25 17:00:00
math: true
hidden: false
categories:
    - 课程笔记
tags:
    - Theory of Computation
---

# Lecture 1 Mathematical Logic (1)

## Informal Statement Calculus (非形式的命题演算)

自然语言中有许多命题 (statement)，比如“拿破仑死了”。而命题之间又可以通过联结词 (connective) 组成更复杂的命题，比如“拿破仑死了*并且*世界正在欢腾”。在这里，我们假设所有的命题都是非真即假的。常见的联结词及其对应的含义可见下表：

|Meaning|Connectives|
|-------|-----------|
|not $A$|$\sim A$|
|$A$ and $B$|$A \land B$|
|$A$ or $B$|$A \lor B$|
|if $A$ then $B$|$A \to B$|
|$A$ if and only if $B$|$A \leftrightarrow B$|

“拿破仑死了*并且*世界正在欢腾”就可以用$A \land B$表示。

各联结词的真值表如下所示：

Negation:

|$p$|$\sim p$|
|-|-|
|$T$|$F$|
|$F$|$T$|

Conjunction:

|$p$|$q$|$p \land q$|
|-----|-----|-----|
|$T$|$T$|$T$|
|$T$|$F$|$F$|
|$F$|$T$|$F$|
|$F$|$F$|$F$|

Disjunction:

|$p$|$q$|$p \lor q$|
|-----|-----|-----|
|$T$|$T$|$T$|
|$T$|$F$|$T$|
|$F$|$T$|$T$|
|$F$|$F$|$F$|

Conditional:

|$p$|$q$|$p \to q$|
|-----|-----|-----|
|$T$|$T$|$T$|
|$T$|$F$|$F$|
|$F$|$T$|$T$|
|$F$|$F$|$T$|

Conditional 的真值表的前两行并不难理解，后两行则可以认为是一种定义上的方便。因为采取这样的定义后，在判断 $p\to q$ 是否恒为真时只需要判断 $p$ 为真是否始终能推出 $q$ 为真即可，而不需要考察 $p$ 为假的情况，这与先前的认知是一致的。

Biconditional:

|$p$|$q$|$p \leftrightarrow q$|
|-----|-----|-----|
|$T$|$T$|$T$|
|$T$|$F$|$F$|
|$F$|$T$|$F$|
|$F$|$F$|$T$|

类似表中的字母 $p, q, r, \cdots$ 称为命题变元 (statement variable)，它们表示任意的非特定的单个命题。而由命题变元和联结词组成的表达式称为命题形式 (statement form)，定义如下：

> **Definition:**
>
> 一个*命题形式*是一个含有命题变元和联结词的表达式，并且能用以下规则构成：
>
> (1) 任何命题变元是一个命题形式。
>
> (2) 如果 $\mathscr{A}$ 和 $\mathscr{B}$ 是命题形式，那么 $(\sim \mathscr{A}), (\mathscr{A} \land \mathscr{B}), (\mathscr{A}\lor \mathscr{B}), (\mathscr{A}\to\mathscr{B}), (\mathscr{A} \leftrightarrow \mathscr{B})$ 是命题形式。

每个命题形式都有其真值表。

> **Definition:**
>
> (1) 一命题形式称为*重言式 (tautology)*，如果对于其中出现的命题变元的各种可能的真值指派，它总取真值为 *T*。
>
> (2) 一命题形式称为*矛盾式 (contradiction)*，如果对于其中出现的命题变元的各种可能的真值指派，它总取真值为 *F*。

$(p\lor \sim p)$ 是一个重言式，而 $(q\land \sim q)$ 是一个矛盾式。

> **Definition:**
>
> 设 $\mathscr{A}$ 和 $\mathscr{B}$ 是命题形式，我们说 $\mathscr{A}$ *逻辑蕴含 (logically implies)* $\mathscr{B}$，如果 $(\mathscr{A}\to\mathscr{B})$ 是一重言式，我们说 $\mathscr{A}$ *逻辑等值 (logically equivalent)* $\mathscr{B}$，如果 $(\mathscr{A}\leftrightarrow\mathscr{B})$ 是一重言式。

$(p\land q)$ 逻辑蕴含 $p$，$(\sim(p\land q))$ 逻辑等值 $((\sim p)\lor(\sim q))$。

> **Proposition:**
>
> 如果 $\mathscr{A}$ 和 $(\mathscr{A}\to\mathscr{B})$ 都是重言式，那么 $\mathscr{B}$ 也是重言式。
>
> **Proof:**
> omitted.

> **Proposition (Rules for Substitution):**
>
> 令 $\mathscr{A}$ 是一个命题形式，其中有命题变元 $p\_1, p\_2, \cdots, p\_n$，并且令 $\mathscr{A}\_1, \mathscr{A}\_2, \cdots, \mathscr{A}\_n$ 是任意命题形式。如果 $\mathscr{A}$ 是一个重言式，那么由 $\mathscr{A}$ 通过用 $\mathscr{A}\_i$ 到处去替换每个 $p\_i$ 而得到的 $\mathscr{B}$ 也是一重言式。
>
> **Proof:**
> omitted.

比如，$(p\land q)$ 逻辑蕴含 $p$，所以对任意 $\mathscr{A}, \mathscr{B}$，都有 $(\mathscr{A}\land \mathscr{B})$ 逻辑蕴含 $\mathscr{A}$。

> **Proposition (De Morgan's Law):**
>
> 令 $\mathscr{A}\_1, \mathscr{A}\_2, \cdots, \mathscr{A}\_n$ 是任意的命题形式，那么：
>
> (1) $(\mathop{\lor}\limits\_{i=1}^n (\sim\mathscr{A}\_i))$ 逻辑等值于 $(\sim(\mathop{\land}\limits\_{i=1}^n \mathscr{A}\_i))$
>
> (2) $(\mathop{\land}\limits\_{i=1}^n (\sim\mathscr{A}\_i))$ 逻辑等值于 $(\sim(\mathop{\lor}\limits\_{i=1}^n \mathscr{A}\_i))$
>
> **Proof:**
>
> 用数学归纳法，先用 Rules for Substitution 证明 $n=2$ 的情形，再推广至任意正整数 $n$。

