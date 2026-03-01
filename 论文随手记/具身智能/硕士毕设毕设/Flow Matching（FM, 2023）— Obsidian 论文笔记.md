
> 论文：Flow Matching for Generative Modeling  
> 作者：Yaron Lipman 等  
> 机构：Meta AI (FAIR), Weizmann Institute  
> 年份：2023  
> 关键词：Continuous Normalizing Flow、CNF、Diffusion、Optimal Transport、Simulation-Free

---

# 1️⃣ 论文解决了什么问题？

> [!danger] 生成模型的两大痛点
> 1. Diffusion 采样慢（需要大量 steps）
> 2. CNF 可表达性强，但训练极其昂贵（需要反复 ODE 模拟）

---

## 🎯 核心目标

构造一种：

- ✅ 不需要 ODE 仿真训练（simulation-free）
- ✅ 比 diffusion 更 general
- ✅ 比 score matching 更稳定
- ✅ 可以直接训练 CNF
- ✅ 支持更优的概率路径（如 OT path）

---

# 2️⃣ Continuous Normalizing Flow 回顾

CNF 定义流：

$$
\frac{d}{dt}\phi_t(x)=v_t(\phi_t(x))
$$

概率通过 pushforward：

$$
p_t = [\phi_t]_* p_0
$$

连续性方程（关键约束）：

$$
\frac{d}{dt}p_t(x) + \text{div}(p_t(x)v_t(x))=0
$$

> [!note]
> 只要向量场满足 continuity equation，它就生成该概率路径。

---

# 3️⃣ Flow Matching 核心思想

我们不从 diffusion SDE 出发。

我们直接：

> **指定一条概率路径 p_t(x)**  
> 然后学习一个向量场 v_t(x) 去匹配它

---

## 3.1 原始 Flow Matching Objective

$$
L_{FM}(\theta)=
\mathbb{E}_{t,x\sim p_t}
\|v_\theta(x)-u_t(x)\|^2
$$

其中：

- $u_t(x)$ 是生成 $p_t$ 的真实向量场
- $v_\theta$ 是我们学的 CNF

> 本质：直接回归 vector field。

---

### ❗问题

- $p_t$ 是 marginal density
- $u_t$ 是 marginal vector field
- 都不可计算

怎么办？

---

# 4️⃣ Conditional Flow Matching（核心突破）

作者提出：

> 不直接构造 marginal 路径  
> 而是构造 per-sample conditional 路径

---

## 4.1 条件概率路径

对每个样本 $x_1$：

$$
p_t(x|x_1)=\mathcal{N}(x|\mu_t(x_1),\sigma_t^2(x_1)I)
$$

满足：

- t=0 → 标准高斯
- t=1 → 收敛到 x₁

然后：

$$
p_t(x)=\int p_t(x|x_1) q(x_1)dx_1
$$

---

## 4.2 Conditional Flow Matching Loss

$$
L_{CFM}=
\mathbb{E}_{t,x_1,x\sim p_t(x|x_1)}
\|v_\theta(x)-u_t(x|x_1)\|^2
$$

神奇结论：

> [!success]
> CFM 与 FM 梯度完全等价！

因此：

- 不需要知道 marginal path
- 不需要知道 marginal vector field
- 只需 per-sample Gaussian path

这就是论文的核心理论贡献。

---

# 5️⃣ Gaussian Conditional Path 推导

设：

$$
\psi_t(x)=\sigma_t x + \mu_t
$$

则对应向量场：

$$
u_t(x|x_1)=
\frac{\dot\sigma_t}{\sigma_t}(x-\mu_t)+\dot\mu_t
$$

> [!important]
> 这是 Gaussian path 的唯一 canonical 向量场。

---

# 6️⃣ 两种关键概率路径

---

# 🔵 6.1 Diffusion Path

Variance Preserving (VP):

$$
p_t(x|x_1)=\mathcal{N}(x|\alpha_{1-t}x_1,
(1-\alpha_{1-t}^2)I)
$$

对应向量场复杂、弯曲。

特点：

- 轨迹是弯曲的
- 噪声直到最后阶段才消失
- 训练稳定，但采样慢

---

# 🟢 6.2 Optimal Transport Path（论文最大亮点）

设：

$$
\mu_t = t x_1
$$

$$
\sigma_t = 1-(1-\sigma_{min})t
$$

对应向量场：

$$
u_t(x|x_1)=
\frac{x_1-(1-\sigma_{min})x}{1-(1-\sigma_{min})t}
$$

---

## ✨ 关键性质

- 轨迹是直线
- 速度恒定
- 条件 flow 是 OT displacement map
- 训练更快
- 采样更少步数
- 数值误差更低

---

### 对比直觉

Diffusion：

> 曲线 → overshoot → 回拉

OT：

> 直线 → 直接到达

论文 Figure 3 明确展示：

- diffusion trajectory 弯曲
- OT trajectory 直线

---

# 7️⃣ 为什么 FM 比 Score Matching 更稳定？

Diffusion 训练的是：

$$
\nabla \log p_t(x|x_1)
$$

FM 训练的是：

$$
u_t(x|x_1)
$$

区别：

| 方法 | 回归目标 |
|------|----------|
| Score Matching | score（梯度） |
| Flow Matching | vector field |

向量场更“物理”，更接近真实 flow dynamics。

论文实验显示：

- FM-Diffusion > Score Matching
- FM-OT 最优

---

# 8️⃣ 采样机制

采样过程：

1. $x_0 \sim \mathcal{N}(0,I)$
2. 解 ODE：

$$
\frac{dx}{dt}=v_\theta(x,t)
$$

优势：

- 直接 ODE
- 不需要 SDE
- 可用任意 ODE solver
- 支持自适应步长

---

## 🔥 采样效率实验

ImageNet 32×32：

FM-OT 只需约 60% NFE  
即可达到 diffusion 相同误差。

低 NFE 下 FID 仍优。

---

# 9️⃣ 实验结论（ImageNet）

FM-OT 在：

- NLL
- FID
- 采样速度

均优于：

- DDPM
- Score Matching
- Score Flow

尤其：

> FM-OT = 更低 NFE + 更好 FID

---

# 🔟 理论贡献总结

> [!summary] 核心贡献

1. 提出 Flow Matching objective
2. 提出 Conditional Flow Matching
3. 证明 CFM 与 FM 梯度等价
4. 将 Diffusion 视为 Gaussian path 特例
5. 引入 OT path（更优路径）
6. 打开“自定义概率路径”的大门

---

# 11️⃣ 与 Diffusion 的关系

Diffusion = 特殊 Gaussian path

FM 是更 general 框架：

```
Diffusion ⊂ Gaussian Paths ⊂ Flow Matching
```

---

# 12️⃣ 更高层理解

Flow Matching 本质是：

> 从“建模噪声过程”  
> → 转为“建模概率路径”

不再思考：

- forward SDE
- reverse SDE
- score trick

而是：

> 直接指定 path  
> 直接匹配 vector field

这是范式级转变。

---

# 13️⃣ Flow Matching 的哲学意义

Diffusion：

> 学 score

Flow Matching：

> 学 flow

OT Flow Matching：

> 学“最直”的 flow

---

# 14️⃣ 复习 Checklist

- [ ] 什么是 Flow Matching？
- [ ] 为什么 FM 不需要仿真？
- [ ] CFM 为什么梯度等价？
- [ ] Gaussian path 的 canonical vector field？
- [ ] Diffusion path 和 OT path 区别？
- [ ] 为什么 OT 采样更快？
- [ ] 为什么 FM 比 score matching 稳定？
- [ ] FM 与 diffusion 的关系？

---

# 🎯 一句话总结

> Flow Matching 是把生成模型从“扩散过程建模”提升到“概率路径建模”的统一框架，而 OT path 让这条路径变得最直、最快、最优雅。
