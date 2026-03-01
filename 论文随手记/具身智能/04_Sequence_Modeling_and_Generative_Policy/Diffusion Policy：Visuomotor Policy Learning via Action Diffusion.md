  
> Chi et al., 2023/2024  :contentReference[oaicite:1]{index=1}

---

## 概览（What & Why）

> [!summary] 一句话总结  
> **Diffusion Policy（DP）把机器人动作序列视为“连续分布”，  
> 用扩散模型（Diffusion Model）来生成轨迹，成为真实机器人模仿学习的新标准（SOTA）。**  
> 它解决了 Transformer（DT/TT）和 LSTM 类 BC 方法的核心缺陷，使操控任务成功率大幅提升。

DP 的关键思想：

- 不预测下一步动作  
- 而是 **预测未来整个动作序列（H-step trajectory）**  
- 并通过扩散过程建模轨迹的复杂分布  
- 配合视觉编码器（CNN 或 Transformer）

---

# 1. 研究背景：为什么需要 Diffusion Policy？

机器人模仿学习（IL）常见 baseline：

- LSTM-GMM（运动原语）
- IBC（Implicit BC）
- BET（Behavior Transformer）

但它们都有共通问题：

> [!note] 现有模仿学习方法的缺陷  
> - **无法稳定预测长时动作序列**（horizon 长 → 累计误差大）  
> - **多模态动作分布难以表示**（例如抓取时动作分布呈多峰）  
> - **行为存在高度不确定性（multi-modal）**  
> - **真实机器人上下界约束多，难以拟合分布**

扩散模型解决了这一切。

---

# 2. 扩散模型如何用于机器人控制？

核心概念：

> **把机器人未来 H 步动作序列当作一个连续信号，  
> 用去噪扩散模型生成它。**

动作序列：

$$
a_{t:t+H} = [a_t, a_{t+1}, \dots, a_{t+H}]
$$

扩散模型的两个过程：

### ✔ Forward diffusion  
给动作加噪声：

$$
x_k = \sqrt{\alpha_k} x_{k-1} + \sqrt{1 - \alpha_k} \epsilon
$$

### ✔ Reverse denoising（学习部分）  
模型学习逐步还原动作：

$$
\hat{x}_{k-1} = f_\theta(x_k, \text{observation})
$$

最后一步输出的 $x_0$ 就是希望执行的动作轨迹。

---

# 3. 模型结构（论文 Figure 2）

论文结构图（如 page 4–5）显示：

**DP=视觉编码器 + 扩散 U-Net（或 Transformer）+ 预测未来动作序列**

## 3.1 视觉编码器  
两种版本：

- **CNN-based encoder**（较快，适合低资源）  
- **Transformer-based encoder**（更强，继承 TT/ViT 思想）

视觉输入 $I_t$→编码成 latent feature $z_t$。

## 3.2 条件扩散模型（Action Diffusion）
核心模块：**conditional U-Net**

输入：

- noisy action trajectory $x_t$
- visual embedding $z_t$
- 当前机器人状态（joint positions, gripper state）
- 时间步 embedding（diffusion timestep）

输出：

- 去噪后的动作序列 $x_0$

---

# 4. Diffusion Policy 的训练目标

学习 denoising：

$$
\min_\theta \mathbb{E} \left[ \| \epsilon - \epsilon_\theta(x_t, t, z) \|^2 \right]
$$

非常简单，就是标准 diffusion loss。

---

# 5. 推理（Inference）

推理使用 reverse diffusion：

1. 采样初始噪声 $x_T \sim \mathcal{N}(0,I)$  
2. 多步去噪：  
   $$
   x_{t-1} = f_\theta(x_t, z)
   $$
3. 去噪完成 → 得到 $x_0 = a_{t:t+H}$  
4. 执行第一个动作 $a_t$

---

# 6. 为什么 Diffusion Policy 强得离谱？

> [!summary] DP 的三大优势

## ✔ 1. 多模态动作分布建模能力极强  
抓取、推物、插入等任务都有多种成功方式（multi-modal）。  
扩散模型天然可以表达多峰分布。

LSTM/GMM、TT、DT 都做不到这么丰富的分布建模。

---

## ✔ 2. 生成的是“整段”动作序列（Trajectory-level）  
传统方法预测单步，容易误差累计。  
DP 预测整个未来轨迹 → 更稳定。

这是继承 **Trajectory Transformer（TT）** 的思想！

---

## ✔ 3. 可扩展到视觉、real robot  
DP 支持：

- RGB  
- Depth  
- Multi-view（如论文 Figure 1）  
- Real robot Franka Panda 等

它比 TT 更工业化。

---

# 7. 实验结果（论文 Figure 3–7）

论文里多个实验（如 page 7–10）说明：

### ✔ 在所有操控任务中，DP > LSTM-GMM、IBC、BET  
成功率提升高达 **2–3 倍**

### ✔ 真机实验（real robot）表现稳定  
任务包括：

- pick-and-place  
- drawer-opening  
- nut-turning  
- insertion  
- pushing  
- tool use

成功率非常高，且鲁棒性强。

---

# 8. 和 DT、TT 的关系（非常重要）

> [!note] Diffusion Policy ≠ DT / TT，但继承了两者的“轨迹建模”思想

| 模型 | 预测目标 | 核心方法 | 典型场景 |
|------|----------|----------|----------|
| DT | 下一步动作 | Transformer | offline RL |
| TT | 完整轨迹 | Transformer + beam search | planning |
| **DP** | 完整动作轨迹分布 | Diffusion + (CNN/Transformer) | visuomotor control（机器人） |

DP 的优势：

- **不用离散化（TT 的痛点）**  
- **不做 beam search（实时控制不行）**  
- **连续动作更自然**  
- **视觉融合更容易**  

所以：

> [!summary] DP = TT 的轨迹建模思想 + 更强的连续分布生成能力

因此成为 2023–2025 机器人操控的主流。

---

# 9. 重要公式备忘

## 扩散 forward step：
$$
x_t = \sqrt{\alpha_t}x_{t-1} + \sqrt{1-\alpha_t}\epsilon
$$

## reverse denoising：
$$
x_{t-1} = \frac{1}{\sqrt{\alpha_t}} 
(x_t - \frac{1 - \alpha_t}{\sqrt{1-\bar{\alpha_t}}}\epsilon_\theta(x_t,t,z))
$$

## policy：
最终动作：
$$
a_t = (x_0)_1
$$

---

# 10. 复习 Checklist（含答案）

> Q：DP 为什么比 LSTM-GMM、IBC 强？  
A：能建模复杂动作分布（multi-modal），误差累积更少。

> Q：DP 和 TT 有何关系？  
A：继承“轨迹级生成”思想，但用连续空间+扩散替代离散 token + beam search。

> Q：DP 如何融合视觉？  
A：CNN/ViT 编码视觉 → 作为条件输入给扩散模型。

> Q：为什么 DP 能用于真实机器人？  
A：稳定、抗噪、能预测长 horizon action、推理速度快。

> Q：DP 的本质是什么？  
A：**Visuomotor diffusion model for action trajectory generation.**

---

# 一句话总结

> **Diffusion Policy = 现代机器人最强的模仿学习策略。  
> 它用扩散模型生成未来动作分布，把 TT 的轨迹模型思想带入工业级机器人控制。**

