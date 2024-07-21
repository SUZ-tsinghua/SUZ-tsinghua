---
title: Deep Reinforcement Learning Lecture 5
description: Actor-Critic
slug: DRL-notes-5
date: 2024-07-21 7:00:00
math: true
hidden: false
categories:
    - 课程笔记
tags:
    - DRL
---

# Actor-Critic

## Off-Policy Policy Gradient

上节中我们提到，REINFORCE 中：

$$\nabla_{\theta}J(\theta)=\mathbb{E}_{\tau}\left[R(\tau)\nabla\_{\theta}\sum\_{t=0}^{T-1}\log\pi(a_t|s_t,\theta)\right]$$

等式右边的期望 $\mathbb{E}_{\tau}$ 是对当前 policy $\pi\_{\theta}$ 而言的。因此在更新 policy 时只能采用当前 policy 跑出来的 trajectory 数据，不能用过往 policy 或者任意别的 policy 跑出来的 trajectory。而这使得 REINFORCE 的训练样本量较小，训练速度较慢。我们现在希望用某种方法使得 REINFORCE 能够使用别的 trajectory 数据。

### Importance Sampling

假设现在有一个随机变量 $x$，其概率密度函数 $p(x)$，我们希望估计函数 $f(x)$ 的均值 $\mathbb{E}_{x\sim p(x)}[f(x)]$。如果我们拥有一些符合概率密度函数 $p(x)$ 的 samples $\\{x_i\\}$ 则可以通过 Monte-Carlo 的办法进行直接估计：

$$\mathbb{E}\_{x\sim p(x)}[f(x)]\approx \frac{1}{|\{x_i\}|} \sum_{i} f(x_i)$$

但实际情况是，我们拥有的 samples $\\{x_i\\}$ 并不符合 $p(x)$，而是符合另一概率密度函数 $q(x)$，则可以用以下 Monte-Carlo 的办法估计：

$$\mathbb{E}\_{x\sim p(x)}[f(x)]=\int f(x) p(x) dx =\int f(x) \frac{p(x)}{q(x)} q(x) dx \approx \frac{1}{|\{x_i\}|} \sum_{i} f(x_i) \frac{p(x_i)}{q(x_i)}$$

这种方法称为 Importance Sampling。

### Off-Policy PG

利用 Importance Sampling，在更新策略时我们可以使用别的 policy 跑出来的 trajectory，假设我们利用 $\pi_{\theta^{\prime}}$ 跑出来的数据来更新 $\pi_{\theta}$。

$$\begin{align*}
\nabla\_{\theta}J(\theta)&=\mathbb{E}\_{\tau\sim p\_{\theta}}\left[R(\tau)\nabla\_{\theta}\sum\_{t=0}^{T-1}\log\pi\_{\theta}(a_t|s_t)\right]\\\\
                        &=\mathbb{E}\_{\tau\sim p^{\prime}\_{\theta}}\left[\frac{p_{\theta}(\tau)}{p_{\theta}^{\prime}(\tau)}R(\tau)\nabla\_{\theta}\sum\_{t=0}^{T-1}\log\pi\_{\theta}(a_t|s_t)\right]\\\\
\end{align*}$$

其中：

$$\frac{p_{\theta}(\tau)}{p_{\theta}^{\prime}(\tau)}=\frac{\mu(s_0)\prod_{t=0}^{T-1}\left[\pi_{\theta}(a_t|s_t)\mathbb{P}(s_{t+1},r_t|s_t, a_t)\right]}{\mu(s_0)\prod_{t=0}^{T-1}\left[\pi_{\theta^{\prime}}(a_t|s_t)\mathbb{P}(s_{t+1},r_t|s_t, a_t)\right]}=\frac{\prod_{t=0}^{T-1}\left[\pi_{\theta}(a_t|s_t)\right]}{\prod_{t=0}^{T-1}\left[\pi_{\theta^{\prime}}(a_t|s_t)\right]}$$

## Actor-Critic