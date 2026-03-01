Chen et al., 2021  :contentReference[oaicite:0]{index=0}

---

> [!summary] **本文一言以蔽之**
> **Decision Transformer（DT）把强化学习变成了“序列建模”问题。  
> 不再需要 Q-learning、不再做 Bellman backup，只需要像 GPT 一样学习轨迹序列，并通过 RTG（Return-To-Go）来生成动作。**

---

## 一、DT 想解决什么问题？
传统强化学习痛点：

- Bellman backup 不稳定（Deadly Triad）
- TD-learning 难以处理长期依赖
- Sparse reward 导致 credit assignment 极难
- 离线 RL 需要 value pessimism（如 CQL）
- 难以大规模扩展到 GPT 级别模型

DT 提出的核心思想：

> **用 Transformer 直接建模完整的轨迹序列，  
> 然后让模型像 GPT 生成句子一样生成动作。**

---

## 二、RTG（Return-To-Go）是什么？

定义：

$$
R_t = \sum_{k=t}^{T} r_k
$$

它代表：  
**从当前时刻开始未来还能拿到多少奖励。**

在 DT 中，RTG 的角色非常关键：

> [!summary] **RTG 的作用**
> - 相当于动作生成的“Prompt”（性能目标）  
> - 训练时帮助模型理解轨迹好坏  
> - 测试时输入更大的 RTG → 生成更优策略  
> - 避免 sparse reward 的不稳定性

DT 的序列格式为：

```
[RTG1, s1, a1, RTG2, s2, a2, ...]
```

---

## 三、模型结构（核心机制）

DT 使用 **GPT-like causal Transformer**，做一件事：

> **给定 (RTG, state, action) 序列 → 自回归预测下一个动作。**

训练目标：

$$
\max_\theta \log \pi_\theta(a_t \mid R_{\le t}, s_{\le t}, a_{<t})
$$

没有：

- Q function  
- value function  
- actor-critic  
- Bellman equations  

它是**纯监督学习**。

---

## 四、训练方式（Supervised Learning）

从离线数据集中采样一个长度为 K 的序列：

```
(R1, s1, a1, ..., RK, sK, aK)
```

每一步只做：

> **预测动作 aₜ（MSE 或 CE loss）。**

不学习 reward，不学习 dynamics，不估计 Q。

---

## 五、推理方式（Action Generation）

流程如下：

1. 输入目标 RTG（如 expert-level return）
2. 输入当前 state
3. Transformer 输出动作分布
4. **从动作分布中采样**（而非 argmax）
5. 与环境交互，获得 reward
6. 更新 RTG：  
   $$
   R_{t+1} = R_t - r_t
   $$
7. 滚动窗口向前继续预测

> [!note] 为什么一定要 sampling？
> - 防止策略坍塌  
> - 保留轨迹多样性  
> - 离线 RL 本质是分布建模  
> - 与 GPT 一样，sampling 泛化更强

---

## 六、实验结论（论文主结果）

### 1. Atari（高维视觉任务）
- DT 的表现与 CQL 接近或更好  
- 在 Pong、Breakout 上特别强  

### 2. D4RL（MuJoCo）
- 全面领先 BEAR/BRAC/AWR  
- 绝大部分环境中超过 CQL  
- Medium-Replay 低质量数据下依然优秀

### 3. Sparse Reward / Delayed Reward
- DT 几乎不受延迟奖励影响  
- CQL 崩溃（因为 TD-learning 需要密集 reward）

### 4. Long-horizon credit assignment（Key-to-Door）
- DT 能识别关键事件（如拿钥匙）  
- 比 CQL 稳定得多  

---

## 七、DT 与 Behavior Cloning（BC）的区别

BC：  
> 只复制训练集中出现过的行为。

DT：  
> 根据 RTG 来选择“想模仿的性能水平”，并可以组合轨迹片段，生成训练集没有出现过的策略。

> [!summary] 关键差异  
> - BC：无目标 → 被动模仿  
> - DT：有目标 → 主动生成满足回报的动作  

---

## 八、为什么 DT 有效？直觉解释

### 1. Transformer 擅长长程依赖  
Credit assignment 通过 self-attention 自然实现。

### 2. 避免 Bellman backup 不稳定性  
彻底绕开“Deadly Triad”。

### 3. 更适合 sparse reward  
RTG 是平滑的、稳定的信号。

### 4. 可以扩展到 GPT 级别模型  
具备“Scaling laws”潜力。

### 5. 天然适合多任务决策  
只需改变 RTG 就能切换策略能力。

---

## 九、具身智能中的意义（非常关键）

DT 的成功直接影响了：

- Trajectory Transformer（TT）
- Diffusion Policy（DP）
- 3D Diffuser Actor
- 谷歌 RT-1 / RT-2
- DeepMind Gato
- OpenAI Sora 的动作建模思想

趋势正在发生：

> **从“RL 优化策略” → “大模型生成策略”**  
> DT 是这个范式转移的起点。

---

## 十、关键公式备忘

**Return-to-go：**

$$
R_t = \sum_{k=t}^{T} r_k
$$

**训练目标：**

$$
\max_\theta \log \pi_\theta(a_t \mid R_{\le t}, s_{\le t}, a_{<t})
$$

**推理动作：**

$$
a_t \sim \pi_\theta(\cdot)
$$

---

## 十一、复习 Checklist（含答案）

> **RTG 是什么？**  
未来剩余奖励，是决策 prompt。

> **为什么 DT 不需要 Bellman backup？**  
因为它是 supervised learning，不需要值函数。

> **为什么 DT 能做长期 credit assignment？**  
Transformer 的 attention 能跨长序列关联关键状态。

> **为什么 sparse reward 对 DT 友好？**  
因为 RTG 是平滑信号，而不是稀疏奖励。

> **DT 和 BC 的区别？**  
DT 是“目标条件化的 BC”。

> **推理时为什么用 sampling？**  
保持策略分布的多样性，避免过拟合。

> **DT 最主要的局限？**  
依赖数据质量；无法进行在线探索。

---

## 十二、一句话总结

> **Decision Transformer = 用 GPT 思维做强化学习，  
> 它把 RL 变成一个“条件生成动作”的序列建模问题，是具身智能时代的基础算法之一。**

