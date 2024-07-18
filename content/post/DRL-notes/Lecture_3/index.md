---
title: Deep Reinforcement Learning Lecture 3
description: Q-Learning and Deep Q-Learning
slug: DRL-notes-3
date: 2024-07-18 9:00:00
math: true
hidden: false
categories:
    - 课程笔记
tags:
    - DRL
---

# Q-Learning and Deep Q-Learning

## From Estimation to Policy

### Policy Evaluation for $Q(s,a)$

在上一节中，我们学习了在 model-free 的情况下，如何进行 Policy Evaluation。我们知道，Policy Iteration 除了 Policy Evaluation，还有一步 Policy Improvement。这里先来看下如何在 model-free 的情况下进行 Policy Improvement。最初的 Policy Iteration 中 Policy Improvement 的方法是：

$$\pi(s)=\argmax_a \sum_{s^{\prime}}T(s,a,s^{\prime})[R(s,a,s^{\prime})+\gamma V(s^{\prime})]$$

但这种方法要求知道 $T(s,a,s^{\prime})$ 和 $R(s,a,s^{\prime})$。另一种方法是，在 Policy Evaluation 的时候，我们并不去计算 $V(s)$，而是计算 $Q(s,a)$。这样就可以直接通过：

$$\pi(s)=\argmax_a Q(s,a)$$

来进行 Policy Improvement。$Q(s,a)$ 也满足 Bellman Equation：

$$Q(s,a)=\sum_{s^{\prime}}T(s,a,s^{\prime})(R(s,a,s^{\prime})+\gamma\max_{a^{\prime}}Q(s^{\prime}, a^{\prime}))$$

因此用 MC、TD 对 $Q$ 进行 Policy Evaluation 的方法与对 $V$ 进行 Policy Evaluation 的方法类似，这里不再详细阐述。

### Exploration v.s. Exploitation

当然，完全根据以上得到的 $Q$ 进行 Policy Iteration 是不可取的。我们来看这样一个简单的例子：

假设一个 MDP 只有一个 state $s$，在这个 state 上有两个 actions $a_1, a_2$。$R(s,a_1)$ 有 100 个样本，均取值为 1；而 $R(s,a_2)$ 仅有一个样本，取值为 0。那么根据以上的 Policy Iteration，得出的 policy 会永远选择 $a_1$。但实际上由于选择 $a_2$ 的样本数较少，我们不能确定 $\mathbb{E}(R(s,a_2))<\mathbb{E}(R(s,a_1))$。所以只用以上的方法进行 RL，很可能无法学到 optimal policy。有时候也需要选择当前看来并不是最优的 action 来拓展眼界。

永远选择当前最优策略的行动方法称为 exploitation，而在上面所说的例子中，我们可以在某些时候选择当前看来并不是最好的 solution $a_2$，这种行为叫做 exploration。

- Exploration: Get more information about the world. 可能有些更好地选择 agent 之前并不知道，它需要勇于尝试未知的事物才能得到更好的 reward。
- Exploitation: To try to get reward. RL 的目的毕竟还是得到更高的 reward，在大多数情况下，我们更希望是一个“保守派”，选择目前最优的策略以期得到较高的 reward。

剩下的问题就是，该在什么时候选择 exploration，什么时候选择 exploitation，这种决定何时选择 exploration 何时选择 exploitation 的策略被称为 exploration/exploitation policy。 一个数学上十分简单的 exploration/exploitaton policy 是 $\epsilon$-greedy exploration，即有 $1-\epsilon$ 的概率选择 greedy action (exploitation)，有 $\epsilon$ 的概率 act randomly (exploration)。由于 exploration 的时候也有可能选到 greedy action，policy 可以被写成：

$$\pi(a|s)=\begin{cases} \epsilon/|A|+1-\epsilon &\text{if } a=\argmax_{a^{\prime}\in A}Q(s,a^{\prime})\\\\
                         \epsilon/|A| & \text{otherwise}\end{cases}$$



有些 exploration/exploitation policy 具有一种特殊的性质，这种性质被称为 **Greedy in the Limit with Infinite Exploration (GLIE)**。顾名思义，这类 exploration/exploitation policy 当 explore 得足够多的时候会收敛到 fully exploitation policy。更严谨地说，当所有的 state-action pair 都被 explore 了无数遍时，

$$\lim_{k\to\infty} N_k(s,a)=\infty$$

要求 exploration/exploitation policy 收敛到 greedy policy，

$$\lim_{k\to\infty}\pi_k(a|s)=\mathbf{1}\left(a=\argmax_{a^{\prime}\in A}Q_k(s,a^{\prime})\right)$$

这里，$k$ 代表走了多少个 step。For example, $\epsilon$-greedy policy is **GLIE** if $\epsilon_k=\frac{1}{k}$. 另一个 **GLIE** 的例子是 Boltzman Exploration：

$$\pi(a|s)=\frac{\exp(Q(s,a)/T)}{\sum_{a^{\prime}\in A}\exp(Q(s,a^{\prime})/T)}$$

要求 $T\to 0$。

### $Q$ Learning

总结一下之前所学的内容：

- 我们先学了 Value Iteration, Policy Iteration 等。
- 但是由于大部分情况下我们并不知道 transition/reward functions，我们需要 MC, TD 等方法来 estimate value function。
- 用 value function 来得到 policy 还是需要 transition/reward functions，所以我们用 TD 来 estimate $Q$ 而非 value function。
- 即使采用了 $Q$ function，我们的学习过程仍然是 passive 的，有些未曾涉足的区域会永远被忽视。因此需要 explore，我们采用某种 exploration/exploitation policy。

通过以上的过程，我们几乎已经创造出了 $Q$ Learning。Formally:

> **$Q$-Learning**:
> - Start with initial $Q$-function (e.g. all zeros).
> - Every time take an action from an exploration/exploitation **GLIE** policy. This action gives a transition $s_t,a_t,r_t,s_{t+1}$. Perform TD update for the $Q$ function:
> $$Q(s_t,a_t)\leftarrow Q(s_t, a_t)+\alpha\left(r_t+\gamma\max_{a^{\prime}}Q(s_{t+1}, a^{\prime})-Q(s_t, a_t)\right)$$

$Q$-learning converges to the optimal $Q$-value function in the limit with probability 1, if every state-action pair is visited infinitely often. 即 $Q$-learning 在某条件下一定能收敛到最优解。

由于每一步的 update 并不依赖真实的 $a_{t+1}$，我们称 $Q$ Learning 是 **off-policy** 的。$Q$ Learning 的一种 **on-policy** 的变种是 State-Action-Reward-State-Action (SARSA)，即把每步 update 替换为：

$$Q(s_t,a_t)\leftarrow Q(s_t, a_t)+\alpha\left(r_t+\gamma Q(s_{t+1}, a_{t+1})-Q(s_t, a_t)\right)$$

## Function Approximation

尽管 $Q$ Learning 非常强大，但它也有其局限性。$Q$ Learning 中，我们需要存储 $Q$ function，其定义域大小正比于 $|S|\times|A|$，因此 $Q$ Learning 只能适用于 $|S|$ 和 $|A|$ 很小的情况，不能适用于围棋 ($\sim 10^{170}$ states), 机器人 (continuous state space) 等领域。 

一个解决方法是，我们并不去计算 $V$ 或是 $Q$ 在定义域上每个点的值，而是用一个参数化的函数去 approximate $V, Q$。这样我们就只需要储存函数的参数，而不用存 $V, Q$ 的每一个函数值。

有许多办法可以 approximate functions，比如 Decision Tree, Linear combinations of features 等。我们着重考虑 differentiable 的方法，比如用 neural networks。

我们先以 $V$ 为例子，我们的目标是找到一个参数 vector $w$, 使得以下 loss 尽可能小：

$$J(w)=\mathbb{E}_{\pi} \left[(v\_{\pi}(S)-\hat{v}(S,w))^2\right]$$

其中，$v_{\pi}$ 是要近似的函数，$\hat{v}$ 是我们的参数化函数。

Gradient Descent:

$$\Delta w = -\frac{1}{2}\alpha\nabla_{w}J(w) = \alpha \mathbb{E}_{\pi}\left[(v\_{\pi}(S)-\hat{v}(S,w)) \nabla\_{w} \hat{v}(S,w)\right]$$

添加的系数 $\frac{1}{2}$ 只是为了把求导出来的 2 给约掉。如果我们只用一个 sample，即 SGD，则式子变为：

$$\Delta w = \alpha (v\_{\pi}(S)-\hat{v}(S,w)) \nabla\_{w} \hat{v}(S,w)$$

其中，$v_{\pi}(S)$ 可以用上一节中讲的 MC、TD 表示，比如：

$$v_{\pi}(S)=G_t\quad MC$$

$$v_{\pi}(S)=R_{t}+\gamma \hat{v}(S_{t+1},w)\quad TD(0)$$

$$v_{\pi}(S)=G_t^{\lambda}\quad TD(\lambda)$$

## Deep $Q$ Learning

与 $V$ 一样，$Q$ 也可以被 neural nets approximate。Deep $Q$ Learning 就是 $Q$-learning with non-linear approximators：

> **Deep $Q$-Learning**:
> - Start with initial parameter values.
> - Every time take an action from an exploration/exploitation **GLIE** policy. This action gives a transition $s_t,a_t,r_t,s_{t+1}$. Perform TD update for the parameters. Do gradient descent on the following loss function:
> $$\mathcal{L}(w)=\left(r_t+\gamma\max_{a^{\prime}}Q(s_{t+1}, a^{\prime}; w)-Q(s_t, a_t; w)\right)^2$$

与 $Q$-learning 不同，deep $Q$-learning 并没有收敛保证，但 empirically 它是能用的。为了 DQN (Deep Q Network) 能训得更好，还需要加一些其他的 tricks。

### Trick 1. Experience Replay

由于每几个 step 的 states 是类似的，比如 $s_t, s_{t+1}, ..., s_{t+n}$，DQN 在用这些 transition 进行更新时很可能会 overfit 到 state space 的这个区域。我们可以用 Experience Replay 来解决这个问题，即每次得到一个 transition $(s_t, a_t, r_t, s_{t+1})$，都将其存到 replay memory $D$ 中，每次要更新 network 时就从 $D$ 中取一个 mini-batch 出来进行更新。

### Trick 2. Target $Q$ Network

在 DQN 的 loss function 中，我们可以发现每次更新 $w$ 都会同时更新 target $(r_t+\gamma\max_{a^{\prime}}Q(s_{t+1}, a^{\prime}; w))$ 和 $Q$-network $(Q(s_t, a_t; w))$。但是同时更新二者容易造成训练不稳定，我们可以让 target 更新得慢一点，即每次更新时用 loss：

$$\mathcal{L}(w)=\left(r_t+\gamma\max_{a^{\prime}}Q(s_{t+1}, a^{\prime}; w^-)-Q(s_t, a_t; w)\right)^2$$

其中，$w^-$ 是较早几个 steps 的参数，等到 $w$ 更新了几个 steps 之后再去更新 $w^-$，而不是每个 step 都去更新 $w^-$。

### Trick 3. Reward Clipping

有些 step 的 reward 非常大，使得 gradient 也很大，容易造成训练不稳定。我们实际计算时，将 reward clip 到某个合适的范围，比如 [-1, 1]。
