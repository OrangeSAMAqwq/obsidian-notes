> *“RT-1: Robotics Transformer for Real-World Control at Scale”*  
> 贡献者：Google Robotics & Everyday Robots

---

# 概览（What & Why）

> [!summary] 一句话总结  
> **RT-1 是第一个真正意义上的“大规模机器人基础模型”，  
> 使用 Transformer 处理视觉 → 动作序列，  
> 在 130k 真实机器人示教数据上训练，  
> 能执行 700+ 任务，高泛化能力，端到端学习。**

RT-1 是机器人界的重大转折点：

- 不再是 RL  
- 不再是 BC  
- 不再依赖手工特征  
- 而是用 **Transformer 直接从视觉 → 动作** 做跨任务控制（multi-task policy）

这是 Diffusion Policy、3DDA、RoboCat → Vision-Language-Action (VLA) 模型之间的关键连接点。

---

# 1. RT-1 想解决什么问题？（动机）

### 传统机器人控制的问题：

1. **单任务训练，无法扩展**  
   每个任务必须训练一个专用模型。

2. **数据量小、泛化弱**  
   标准 BC / RL 在 unseen objects 上表现差。

3. **模型无法共享知识**  
   新任务无法迁移旧任务学到的经验。

4. **难以工业部署**  
   控制不稳定，真实世界易出错。

---

> [!summary] RT-1 的目标  
> 构建一个 **能学习数百任务、操作数千物体、跨场景泛化** 的机器人 Transformer。  
> 训练在“一个巨大的混合数据集”上，不再为每个任务训练独立模型。

---

# 2. 数据集：130k 真实机器人示例（规模革命）

RT-1 使用 Google Everyday Robots 收集的数据：

- **130,000+ episodes**  
- **700+ tasks**  
- 数千物体  
- 多机器人、多环境、多相机视角  
- 无需仿真，全部真实世界数据  

这让 RT-1 的泛化能力达到：

> **模型越大 → 任务越多 → 越能成功执行新任务。**

类似 NLP 界的 Scaling Law。

---

# 3. 模型架构（Architecture）

RT-1 本质是：

> **Vision Encoder（EfficientNet） → Tokenizer → Transformer → Multi-step Actions**

整体 pipeline：

```
Multiview RGB → Vision Tokens
Robot state → State Tokens
-------------------------------------
Concat → Perceiver-style Transformer
-------------------------------------
Outputs → Action tokens (discretized)
```

---

## 3.1 视觉编码（Vision Encoder）

使用 EfficientNet-V2（轻量但强大）。  
将图像压缩成 patch-like embedding：

- 捕捉物体形状  
- 颜色  
- 任务上下文  

---

## 3.2 动作 Tokenization（核心创新之一）

RT-1 不直接回归动作，而是将动作 **离散化（tokenization）**：

例如：

- gripper open/close  
- Δx / Δy / Δz  
- rotation  
- wrist angle  

每个动作维度变成 **一组 token**。

为什么要离散化？

> [!note]  
> Transformer 在离散空间训练更稳定、更可扩展，  
> 类似 NLP 的 token → word distribution。

---

## 3.3 Transformer（Perceiver-style）

结构灵感来自 Perceiver AR：

- 支持长序列  
- 高效多模态融合  
- 内存成本低  
- 可扩展到巨大数据集  

Transformer 输出 **未来多帧动作**（multi-step action prediction），比 BC 的单帧预测更稳定。

---

# 4. 训练（Training）

训练任务是：

$$
\max_{\theta} \prod_t P_\theta(a_t \mid o_t)
$$

即可视为：

- 一个多任务监督学习问题  
- 类似 GPT 的 next-token prediction

也不需要 RL，不需要 reward signal。

---

# 5. 泛化能力（Generalization）

RT-1 的突破在于：

> [!summary] **它不仅执行训练过的任务，也能 zero-shot 新任务。**

例如：

- 新物体  
- 新组合  
- 新视角  
- 新目标位置  
- 多任务混合要求（组合技能）

论文实验显示：

- 在训练任务上：**高成功率（70%–90%）**  
- 在未见过的任务组合上：显著优于所有 baseline  
- 比 BC 强 3×、比 IBC 强 2×  

---

# 6. 多任务能力（Multi-task Learning）

RT-1 同时学习 700+ 任务。

Transformer 自然利用跨任务的知识共享：

- 识别相同物体  
- 复用抓取技能  
- 复用移动技能  
- 学到泛化语义（如“手靠近目标再抓取”）  

这就是“Foundation Robot Model”的雏形。

---

# 7. RT-1 为什么强？（核心思想）

> [!tip] RT-1 的强大来自三个本质因素：

---

## ✔ 1. 大规模真实数据（Real-World Large Dataset）  
机器人界首次出现超 100k trajectories 的 dataset。

---

## ✔ 2. Transformer 架构的序列建模能力  
能自动学习：

- 什么是抓取  
- 什么是移动  
- 什么是避障  
- 什么是任务目标  

无需 explicit model。

---

## ✔ 3. Tokenized action space  
让 Transformer 可以：

- 轻松预测动作  
- 稳定收敛  
- 支持长 horizon  
- 更容易 scaling up

RT-1 是 Diffusion Policy + TT 思想在工业级别上的成功验证。

---

# 8. RT-1 与 RoboCat 的关系（非常重要！）

| 项目 | RoboCat | RT-1 |
|------|---------|-------|
| 多机器人 | ✔ | ❌（主要一个机器人） |
| 多任务 | ✔ | ✔（700+ 任务） |
| 自我提升 | ✔ | ❌ |
| 模型结构 | Transformer | Transformer |
| 训练方式 | Demonstration + self-improve | Pure supervision on large dataset |

RT-1 吸收了 RoboCat 的基础理念：

- 多模态输入  
- 轨迹序列预测  
- Transformer backbone  

但 RT-1 的创新在于：

- **规模更大**  
- **tokenization 更成熟**  
- **推理更高效**  
- **操控成功率更高**  

这为 RT-2（Vision-Language-Action 基础模型）奠定基础。

---

# 9. RT-1 的意义（Why It Matters）

> [!summary] RT-1 是现代机器人基础模型时代的真正开端。

它向世界证明：

### ✔ 1. Transformer's scaling law 同样适用于机器人  
数据量越大 → 泛化越强。

### ✔ 2. 多任务学习是通向通用机器人智能的必经之路  
不再为每个任务训练新模型。

### ✔ 3. 机器人可以端到端从 **视觉 → 动作**  

### ✔ 4. 任务间的知识可以共享  
（如移动技能、抓取技能）

### ✔ 5. 改变整个领域的范式  
RT-1 → RT-2 → RT-X → OpenVLA → Octo → 2026的机器人GPT

RT-1 是 Foundation Robot Models 的基石。

---

# 10. 复习 Checklist（含答案）

> Q：RT-1 预测什么？  
A：动作 token sequence（未来多步动作），不是连续控制量。

> Q：RT-1 和 Diffusion Policy 最大区别是什么？  
A：DP 用扩散生成动作分布；RT-1 用 Transformer 离散 tokens 预测动作序列。

> Q：为什么 RT-1 需要 tokenization？  
A：Transformer 在离散空间表现更稳定、更易 scaling。

> Q：RT-1 为什么比 BC 强？  
A：多任务训练 + 更强的序列建模 + action tokenization。

> Q：RT-1 如何泛化？  
A：学习 130k 跨任务数据，Transformer 自动捕捉通用操作技能。

> Q：RT-1 的定位是什么？  
A：**第一个真正可扩展的机器人基础模型。**

---

# 一句话总结

> **RT-1 = Transformer + 大规模真实数据 → 机器人界的 GPT-1。  
> 它开启了 Vision-Language-Action 基础模型（RT-2）的新时代。**

