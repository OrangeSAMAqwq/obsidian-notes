 *Learning Latent Dynamics for Planning from Pixels*  
Danijar Hafner et al., 2019  

---

## 1. 论文基本信息

> [!summary] **基本信息**
> - **标题：** Learning Latent Dynamics for Planning from Pixels  
> - **作者：** Hafner, Lillicrap, Ha 等  
> - **年份：** 2019 (ICML)  
> - **关键词：** world model、latent dynamics、planning、RSSM、model-based RL  
> - **核心地位：** Dreamer 系列的起点，现代世界模型路线奠基论文  

---

# 2. PlaNet 解决了什么问题？

> [!danger] 强化学习的三大痛点（像素控制场景）

1. **model-free 太费数据**（机器人无法承受）
2. 无法利用“想象未来”的能力
3. 像素输入维度高，动力学难以建模

传统做法：

- DDPG / SAC：直接学 Q 或 policy
- 不建模环境
- 完全依赖 trial & error

PlaNet 的问题意识是：

> 如果已知动力学，planning 非常强（MPC、控制理论）  
> 那为什么不先学一个动力学？

---

# 3. PlaNet 的核心思想（一句话）

> **学习一个 latent 世界模型，然后在这个模型中做 planning。**

结构分三步：

1. 图像 → latent state  
2. 学 latent dynamics  
3. 在 latent 中 rollout + CEM 搜索动作序列  

---

# 4. 世界模型（World Model）是什么？

环境真实动力学：

$$
s_{t+1} \sim p(s_{t+1} \mid s_t, a_t)
$$

PlaNet 学的是：

$$
\hat{s}_{t+1} \sim p_\theta(s_{t+1} \mid s_t, a_t)
$$

区别在于：

- 不知道真实动力学
- 用神经网络去拟合

> [!tip] 直觉理解  
> 它不是学物理公式，而是学“统计时间序列预测”。

---

# 5. 为什么不能直接在像素空间预测？

像素空间：

- 维度极高
- 误差累积严重
- rollout 很快崩溃

因此 PlaNet 采用：

> 在 latent space 里建模动力学

流程：

```
Image → Encoder → s_t
```

然后在 latent 中预测：

$$
s_{t+1} = f(s_t, a_t)
$$

---

# 6. 核心结构：RSSM（Recurrent State-Space Model）

> [!important] 这是 PlaNet 真正的技术核心

RSSM = deterministic + stochastic 两条路径

---

## 6.1 结构形式

Deterministic 部分（记忆长期信息）：

$$
h_t = f(h_{t-1}, s_{t-1}, a_{t-1})
$$

Stochastic 部分（表达不确定性）：

$$
s_t \sim p(s_t \mid h_t)
$$

Observation / reward：

$$
o_t \sim p(o_t \mid h_t, s_t)
$$

$$
r_t \sim p(r_t \mid h_t, s_t)
$$

---

## 6.2 为什么要两条路径？

> [!note] 论文 Figure 2 的核心思想

如果只有 deterministic RNN：

- 无法表达多种未来
- planner 会 exploit 误差

如果只有 stochastic SSM：

- 长期记忆能力弱
- 信息容易丢失

RSSM 结合两者：

- deterministic 负责记忆
- stochastic 负责多模态表达

这是 PlaNet 稳定规划的根本。

---

# 7. 训练目标（变分推断）

PlaNet 使用 VAE 风格的目标。

---

## 7.1 Reconstruction

$$
\mathcal{L}_{rec}
=
\sum_t
\mathbb{E}_{q(s_t)}[\log p(o_t \mid s_t)]
$$

---

## 7.2 KL 正则

$$
\mathcal{L}_{KL}
=
\sum_t
KL(
q(s_t \mid o_{\le t})
\Vert
p(s_t \mid s_{t-1}, a_{t-1})
)
$$

---

## 7.3 总损失

$$
\mathcal{L}
=
\mathcal{L}_{rec}
-
\mathcal{L}_{KL}
$$

> [!tip] 直觉  
> posterior 要接近 transition prior，  
> 这样 rollout 时才能可信。

---

# 8. Latent Overshooting（解决多步预测误差）

问题：

> 训练只约束一步预测  
> 但 planning 需要多步 rollout

PlaNet 提出：

> 在 latent space 约束多步预测

形式：

$$
KL(q(s_t) \Vert p(s_t \mid s_{t-d}))
$$

d = 1 … D

作用：

- 减少 rollout 漂移
- 提升 planning 稳定性
- 不需要生成额外像素（高效）

---

# 9. Planning（CEM）

PlaNet 不训练 policy 网络。

它直接：

1. 在 latent 中 rollout H 步
2. 计算累计 reward
3. 用 CEM 搜索动作序列

流程：

初始化动作分布：

$$
a_{t:t+H} \sim \mathcal{N}(0, I)
$$

重复：

- 采样 J 个序列
- 选 top-K
- 更新均值和方差

最终：

> 执行第一个动作（MPC 思想）

---

# 10. 为什么 PlaNet 数据效率高？

Model-free：

- 只学 reward 相关信息
- Bellman backup 信息稀疏

Model-based：

- 每一步都学 dynamics
- 每一帧都有监督信号
- 信息密度高很多

> [!success] 论文结果：  
> 1000 episodes ≈ D4PG 100000 episodes

---

# 11. PlaNet 的局限

1. 每一步都要 CEM → 计算开销大  
2. 不能 end-to-end 优化 policy  
3. rollout horizon 有限制  

这直接导致下一篇：

> Dreamer（把 planning 变成 actor-critic）

---

# 12. PlaNet 在具身智能路线中的位置

```
Model-free RL → PlaNet → Dreamer → DreamerV3 → World Models × LLM
```

它的历史意义：

- 第一次在像素环境中成功 planning
- 第一次证明 latent dynamics 足够稳定
- 世界模型正式成为主流方向

---

# 13. 复习 Checklist（带答案）

---

### 最大贡献是什么？

RSSM + latent planning。

---

### 为什么不用像素 planning？

维度高、误差爆炸、计算昂贵。

---

### RSSM 为什么要 deterministic + stochastic？

- deterministic：长期记忆
- stochastic：多模态未来
- 组合才能稳定 rollout

---

### latent overshooting 解决什么？

多步预测误差累积。

---

### PlaNet 与 SAC 本质区别？

SAC 不学环境，只学 Q。  
PlaNet 先学环境，再规划。

---

# 🎯 一句话总结

> PlaNet 证明了：  
> 只要学到稳定的 latent dynamics，就可以在脑海中想象未来，并用 planning 控制智能体。

它开启了世界模型时代。

