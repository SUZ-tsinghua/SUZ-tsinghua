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

事实上，我们除了可以选取 $\mathbb{E}\_{\tau}\left[R(\tau)\nabla\_{\theta}\sum\_{t=0}^{T-1}\log\pi(a_t|s_t,\theta)\right]$ 来作为 $\nabla_{\theta}J(\theta)$，还有许多别的选择：

![A general form of policy gradient methods. (Schulman et al., 2016, https://arxiv.org/abs/1506.02438)](general_form_policy_gradient.png)

我们接下来考虑形式：

$$\nabla_{\theta}J(\theta)=\mathbb{E}\_{\tau}\left[\sum\_{t=0}^{T-1}(Q^{\pi}(s_t, a_t)-V^{\pi}(s_t))\nabla\_{\theta}\log\pi(a_t|s_t,\theta)\right]$$

其中 $V^{\pi}(s_t)$ 即为上一讲说到的 baseline，可以起到降低 variance 的作用。

由于 $V^{\pi}$ 并不能轻易得到，我们在原先已有神经网络 $\pi_{\theta}$ 的基础上再加一个神经网络 $V^{\pi}\_{\phi}$。用新的神经网络 $V^{\pi}\_{\phi}$ 去拟合 $V^{\pi}$。我们需要同时学习这两个神经网络。直观上来看，由于新的神经网络表征的是 value function，它相当于对原有的 policy 网络 $\pi_{\theta}$ 进行评判，因此我们把这类方法称为 actor-critic，$\pi_{\theta}$ 是 actor，$V^{\pi}_{\phi}$ 是 critic。

### Batch Actor-Critic Algorithm

> **Batch Actor-Critic Algorithm (Without Discount):**
> - 在机器人或别的智能体上跑 $\pi_{\theta}$，得到很多 trajectories 数据。
> - 用收集到的数据更新 $V^{\pi}\_{\phi}$ 网络。
> - 对收集到的每个 transition $(s, a, r, s^{\prime})$， 计算出估计的 advantage function $A^{\pi}(s,a)=r(s,a)+V^{\pi}\_{\phi}(s^{\prime})-V^{\pi}\_{\phi}(s)$。
> - 算出梯度 $\nabla_{\theta}J(\theta)\approx \frac{1}{N} \sum_i A^{\pi}(s_i,a_i)\nabla_{\theta} \log\pi_{\theta}(a_i|s_i)$。
> - 更新参数 $\theta\leftarrow \theta + \alpha \nabla_{\theta}J(\theta)$。

但如果我们不想收集完一整个 episode 的数据再更新，而是想要每个 step 都能更新一次怎么办？我们可以用 TD update 来更新 $V^{\pi}\_{\phi}$，并且再加入 discount。

> **Batch Actor-Critic Algorithm:**
> - 在机器人或别的智能体上跑 $\pi_{\theta}$，得到一个 step 的数据 $(s, a, r, s^{\prime})$。
> - 以 $r+\gamma V^{\pi}\_{\phi}(s^{\prime})$ 为目标更新 $V^{\pi}\_{\phi}(s)$。
> - 计算出估计的 advantage function $A^{\pi}(s,a)=r(s,a)+V^{\pi}\_{\phi}(s^{\prime})-V^{\pi}\_{\phi}(s)$。
> - 算出梯度 $\nabla_{\theta}J(\theta)\approx \frac{1}{N} \sum_i A^{\pi}(s_i,a_i)\nabla_{\theta} \log\pi_{\theta}(a_i|s_i)$。
> - 更新参数 $\theta\leftarrow \theta + \alpha \nabla_{\theta}J(\theta)$。

### DDPG

我们知道，在 DQN 中，$Q$ 的更新需要用到 target: $r+\gamma\max_{a^{\prime}}Q(s^{\prime},a^{\prime})$。由于要对 $a^{\prime}$ 取最大值，DQN 不能直接被应用到 continuous action 的情况。那是否有办法将其应用到 continuous action 的情况呢？提供几种可能的解决方法：

- Solution 1: 取 $N$ 个 $a$ 的samples，取其中的最大值，即认为 $\max_{a^{\prime}}Q(s^{\prime},a^{\prime})\approx \max{Q(s^{\prime},a^{\prime}_1), Q(s^{\prime},a^{\prime}_2), ..., Q(s^{\prime},a^{\prime}_N)}$。但这样取出来的值可能和真正的最大值差的非常大，尤其是 action space 很大的时候。
- Solution 2：参数化 $Q$ 时用 $Q_{\phi}(s,a)=-\frac{1}{2}(a-\mu_{\phi}(s))^{T}P_{\phi}(s)(a-\mu_{\phi}(s))+V_{\phi}(s)$。这样可以直接得出 $\arg\max Q$ 和 $\max Q$，但由于限制了表达形式，会让 $Q$ less expressive。
- Solution 3：每次求 $\max_{a^{\prime}}Q(s^{\prime},a^{\prime})$ 都做 gradient ascent。这样太慢了。
- Solution 4：直接学习一个 $\mu_{\theta}(s)$ s.t. $\mu\_{\theta}(s)\approx\argmax_a Q(s,a)$。

顺着 solution 4，我们得到一个新的算法 Deep Deterministic Policy Gradient (DDPG)。

> **DDPG:**
> - 每一步的 action 从 $\mu_{\theta}(s)$+noise 中 sample，得到很多 transitions $(s_i, a_i, s_i^{\prime}, r_i)$，将他们存到 buffer $\mathcal{B}$ 中。
> - 随机从 $\mathcal{B}$ 中 sample 得到 $N$ 个 transitions $(s_j, a_j, s_j^{\prime}, r_j)$。
> - 用 target network 得到 $y_j=r_j+\gamma Q_{\phi^{\prime}}(s_j^{\prime},\mu_{\theta^{\prime}}(s_j^{\prime}))$。
> - Update $Q_{\phi}$ to minimize the loss funciton $L=\frac{1}{N}\sum_j(y_j-Q_{\phi}(s_j, a_j))^2$。
> - Update the actor network $\mu_{\theta}(s)$ by $\theta\leftarrow \theta+\beta\frac{1}{N}\sum_j\frac{d\mu}{d\theta}(s_j)\frac{dQ}{da}(s_j, \mu_{\theta}(s_j))$。
> - 用不管什么方法，更新 target network。

### TD3

Twin Delayed DDPG (TD3) 通过在 DDPG 的基础上进行改进得来。它多加了 3 个技巧：

#### Clipped Double Q-learning

这是说，采用两套 networks $(Q_{\phi_1}, \mu_{\theta_1}), (Q_{\phi_2}, \mu_{\theta_2})$。在更新两套 networks 时，分别采用：

$$y_1=r+\gamma \min_i Q_{\phi_i}(s^{\prime}, \mu_{\theta_1}(s^{\prime}))$$
$$y_2=r+\gamma \min_i Q_{\phi_i}(s^{\prime}, \mu_{\theta_2}(s^{\prime}))$$

这有助于解决 overestimation of $Q$。

#### Delayed Policy Updates

因为 $Q$ network 和 $\mu$ network 是耦合的，也就是说当 policy 较差的时候， $Q$ 由于 overestimation 也会变差；而 $Q$ 变差了又会得到更差的 policy。因此降低 $\mu$ network 的更新频率，每次先固定 $\mu$，更新多次 $Q$ 直到其较为稳定，这时再去更新 $\mu$。

#### Target Policy Smoothing

计算 $y$ 时，将 $\mu_{\theta^{\prime}}(s^{\prime})$ 替换为 $clip(\mu_{\theta^{\prime}}(s^{\prime})+clip(\epsilon, -c, c), a_{low}, a_{high})$。这样可以让 policy 函数更加 smooth，这是基于相近的 state 本就应该有相近的 action 的假设的。
