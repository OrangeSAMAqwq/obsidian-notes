
> 作者：Michael Janner, Sergey Levine  
> 来源：&#8203;:contentReference[oaicite:0]{index=0}

---

## # 概览（What & Why）

> [!summary] **一句话总结**
> **Trajectory Transformer (TT)** 把 **强化学习过程视为“序列建模问题”**：  
> **把状态、动作、奖励都当成 tokens，用 Transformer 模型整个轨迹。**  
> 然后使用 **beam search 规划** 来输出动作序列。

> TT = **DT 的“轨迹级版本”**  
> 不预测“下一步动作”，而是预测 **完整的未来 trajectory**。

---

## # 1. TT 解决的问题  
传统 RL 有两种做法：

1. **model-free**（SAC/PPO 等）  
   - 只能学一个 step 的策略  
   - 不能建模长时依赖  
   - 对 offline dataset 非常敏感  

2. **model-based**（PlaNet / Dreamer）  
   - 有 world model，但通常是 step-wise dynamics  
   - compounding error（累计误差）很严重  

> [!summary] **TT 的核心想法**
> 直接用 Transformer 去“模拟整个轨迹”，  
> **避免单步模型的误差累积**，  
> 并且能自然建模 long-horizon dependencies。

---

## # 2. 基本思想：RL = 序列建模

论文提出一个极重要的观点：

> **RL 可以被理解为一个序列生成问题，而不是优化问题。**

轨迹定义为：

$$
\tau = (s_1, a_1, r_1, s_2, a_2, r_2, ..., s_T, a_T, r_T)
$$

TT 把所有维度（状态维度 N、动作维度 M）都离散化为 tokens：

```
[s1^1, s1^2, ..., s1^N, a1^1, ..., a1^M, r1, Rt1,
 s2^1, ..., a2^M, r2, Rt2, ...]
```

> [!note]  
> 这是 TT 的核心：  
> **把连续轨迹 → token 序列 → 用语言模型方式训练。**

---

## # 3. 模型结构

### ✔ 输入 token 序列：  
- 状态：每个维度一个 token  
- 动作：每个维度一个 token  
- 奖励：一个 token  
- reward-to-go：一个 token

### ✔ 模型：GPT 风格 Transformer  
- decoder-only  
- causal mask  
- teacher forcing 训练  

### ✔ 输出：下一个 token 的概率分布  
可以是下一维度的 state, action, reward…

---

## # 4. 为什么 Transformer 适合轨迹？  

> [!tip] Transformer 的优势  
> - 能建模 **长时间依赖**（长 horizon）  
> - Autoregressive 训练适合轨迹预测  
> - discrete token 方式比 Gaussian dynamics 稳定  
> - 天然支持 sequence likelihood，用 beam search 直接规划  

在论文中（图 3）清楚显示：

> **TT 的长序列预测性能远超 Gaussian dynamics model（例如 PETS）。**

这就是为什么 TT 能做 long horizon planning。

---

## # 5. TT 的三大用途（规划方式）

TT 只需要一个模型，但可以做三种不同任务：

---

### **A. Imitation Learning（模仿学习）**

直接用 beam search 找 **最可能的轨迹**：

$$
\tau^* = \arg\max_\tau P_\theta(\tau \mid s_t)
$$

取第一步动作作为输出。

效果：  
- 在 Hopper / Walker2d 上可达 **≈ 110% expert**  
- 比行为克隆更稳健，因为生成的是“整段动作”

---

### **B. Goal-conditioned RL（目标条件控制）**

直接在序列中加入目标状态：

```
[goal_state, s1, a1, r1, s2, ...]
```

Transformer 自动学习：

> 如果想达到目标，该生成怎样的轨迹？

无需奖励、无需 shaping，效果惊艳（论文 Figure 6）。

---

### **C. Offline RL（离线强化学习）**

核心难点：beam search 默认生成“高似然”轨迹，而不是“高奖励”轨迹。

解决方案：  
在 tokens 中加入 **reward-to-go**：

$$
R_t = \sum_{k=t}^T \gamma^{k-t} r_k
$$

规划时用：

$$
\text{score} = \sum r_t + R_{t+1}
$$

结果：  
- 在 D4RL locomotion 上几乎 state-of-the-art  
- 在 AntMaze 上结合 Q-function → 全部 SOTA

---

## # 6. TT 的关键技术设计

### ✔ 1. 离散化 continuous 状态/动作  
两种离散化方案：

- uniform  
- quantile（表现更好）

### ✔ 2. beam search（语言模型的解码器 → RL 的 planner）  
这是论文最大的创新之一：

> 把一切 RL 规划问题都转换为“序列生成 + beam search”。

### ✔ 3. Q-guided Beam Search（解决 sparse reward）  
对困难任务（AntMaze）：

> TT + IQL 的 Q-function  
> → 暴打所有 offline RL 方法（包括 CQL、MBOP、DT）。

---

## # 7. TT 的意义：DT → TT → DP 的核心桥梁

> [!summary]  
> DT：一步预测一个动作（token-level generation）  
>  
> **TT：预测整段轨迹（trajectory-level generation）**  
>  
> Diffusion Policy：生成连续动作分布（distribution-level generation）

TT 展示了“轨迹也可以像句子一样建模”，是 2023–2024 所有 diffusion-based policy 的直接前身。

---

## # 8. TT 的优点与局限

### ✔ 优点  
- 统一 offline RL / imitation / goal-reaching  
- 长序列预测 SOTA  
- 规划自然且稳定  
- 解决 model error compounding  
- 强大的 sequence modeling 能力  

### ✘ 局限  
- 速度慢：beam search + Transformer → 不适合实时控制  
- 需要离散化（损失精度）  
- 不能直接理解世界，只能模仿轨迹  
- 仍依赖 offline dataset（不具备探索能力）  

---

## # 9. 对具身智能的意义（非常重要）

TT 展示了一个关键趋势：

# ➤ **机器人控制不再依赖 RL 优化，而是依赖生成模型（Generative Policy）。**

TT 推动的时代：

- 行为建模 > 动态规划  
- 大模型 > 手工设计 RL 算法  
- 轨迹生成 > 单步控制  
- 生成式规划 > value iteration  

TT 是 **Diffusion Policy 的直接理论基础**。

---

## # 10. 复习 Checklist（带答案）

### ✔ TT 如何看待 RL？
> RL = 轨迹序列生成任务  
> 目标是生成高奖励的 token 序列。

### ✔ 为什么 TT 预测更稳定？
> 因为它预测整段轨迹，而不是单步 dynamics，  
> 避免了 model compounding error。

### ✔ 为什么需要离散化？
> 为了让连续状态/动作能被 Transformer token 化。

### ✔ TT 的 planning 如何工作？
> 使用 beam search 生成高概率/高奖励的未来轨迹。

### ✔ TT 与 DT 最大区别？
> DT 生成动作，TT 生成整个 trajectory。  
> TT 更像 world-level planning。

### ✔ TT 如何做 offline RL？
> 插入 reward-to-go token，用 beam search 最大化回报。

### ✔ TT 为何能结合 Q-function？
> 因为 beam search 本质上可以用任意 score function 引导。

### ✔ TT 的局限？
> 实时性太差；需要离散化；不能做真实世界推理。

---

