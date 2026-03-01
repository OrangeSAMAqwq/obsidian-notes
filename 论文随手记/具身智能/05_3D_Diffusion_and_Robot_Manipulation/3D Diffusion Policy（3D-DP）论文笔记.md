
> 一个通用的 3D visuomotor diffusion framework  
> 基于 2023–2024 多篇 3D Diffusion Manipulation 工作总结  
> （包括 DP3D、SE(3) Diffusion、3D Diffusion Fields、VoxelDiffusion 等）

---

# 概览（What & Why）

> [!summary] 一句话总结  
> **3D Diffusion Policy = 在 3D 空间中进行策略生成（action diffusion），  
> 用点云 / voxel / 3D 特征场表示场景，让扩散模型基于真实空间结构生成动作轨迹。**

为什么要 3D？

因为机器人操作（manipulation）本质是：

- 3D 几何任务  
- 物体姿态（orientation）  
- 位置关系（spatial relations）  
- 约束（contacts, collisions）  
- 插入 / 抓取 / 开关 / 组装等高精度任务  

**2D Diffusion Policy 在复杂操控任务明显不够用。**  
因此诞生了 3D-DP。

---

# 1. 动机（Why 3D?）

2D RGB 存在不可避免的问题：

> [!note] 2D 的致命缺陷  
> - 无法表达深度（depth）  
> - 同一像素点对应多个可能的 3D 解释  
> - 遮挡（occlusion）使操作不稳定  
> - 相机视角变化会导致行为崩溃  
> - 难以处理多物体复杂操作  
> - 对接/插入任务 2D 无法提供正确的接触几何  

因此 2023–2024 的共识是：

# ⭐ “机器人策略必须使用 3D 表示，而不是 2D 图像。”

---

# 2. 输入表示（3D Observations）

主流有三种 3D 输入形式：

---

## (1) 点云 Point Cloud  
最常用，来自深度相机。

优点：

- 轻量  
- 直接表示 $(x,y,z)$  
- 易于处理  

常用编码方式：

- PointNet++  
- 3D sparse convolution  
- voxelization → 3D CNN  

---

## (2) 体素（Voxel Grid）  
将空间离散成 cubic grid  
Each voxel holds occupancy / feature

优点：

- 扩散模型更容易使用 3D CNN  
- 支持空间卷积（spatial conv）  

---

## (3) 3D 特征场（Feature Fields）  
如：

- TSDF（truncated signed distance field）  
- Gaussian Splatting  
- NeRF feature volume  
- 遮挡感知 feature maps  

这些提供比 RGB 更丰富的结构信息。

---

> [!tip] 3D-DP 的关键思想：  
> **动作生成过程必须“对齐到”真实 3D 空间。**

---

# 3. 动作表示（SE(3) Action Parameterization）

机器人末端执行器动作通常包括：

- 3D 位置（xyz）
- 3D 旋转（quaternion 或 axis-angle）
- gripper 状态（open/closed）
- 有时包含关节角（joint positions）

形成：

$$
a_t = [x, y, z, q_x, q_y, q_z, q_w, g]
$$

未来 H-step 轨迹：

$$
a_{t:t+H}
$$

扩散模型生成的就是这个 **动作轨迹向量**。

---

# 4. Diffusion Policy 的 3D 版（核心结构）

3D-DP 的整体流程：

> 3D encoder → scene feature volume → action diffusion → trajectory denoising

---

## (1) 3D Encoder  
输入点云/voxel，输出 3D 特征：

$$
F_{3D} = \text{Encoder}_{3D}(P)
$$

可以是：

- Sparse 3D CNN  
- PointNet++  
- Voxel Transformer  
- 3D Gaussian Splatting encoder  

---

## (2) Conditional Diffusion Model  
扩散过程与标准 diffusion 完全一致：

### Forward：

$$
x_t = \sqrt{\alpha_t} x_{t-1} + \sqrt{1-\alpha_t}\epsilon
$$

### Reverse：

$$
x_{t-1} = f_\theta(x_t, F_{3D}, t)
$$

---

## (3) 融合 3D 场景  
关键操作是 **cross-attention 或 3D feature injection**：

$$
h' = \text{CrossAttn}(h, F_{3D})
$$

其中：

- $h$：denoising 网络的隐藏状态  
- $F_{3D}$：3D 特征体  

这样模型在生成动作轨迹时 **持续参考 3D 空间结构**。

---

# 5. 训练目标

标准扩散 loss：

$$
\mathcal{L} = \mathbb{E}[\| \epsilon - \epsilon_\theta(x_t, t, F_{3D}) \|^2]
$$

训练数据来自：

- teleoperation（远程示教）  
- kinesthetic teaching（手把手示教）  
- human demonstration  

无需 reward，无需 RL。

---

# 6. 推理（Inference）

与 2D Diffusion Policy 类似：

1. 从 Gaussian 采样噪声轨迹 $x_T$  
2. 多次去噪  
3. 得到动作轨迹 $x_0$  
4. 执行第一步动作  

关键区别：

> [!summary] **去噪过程始终参考 3D 场景结构**  
→ 因此 trajectory 与 3D 几何强绑定  
→ 操作更稳定、更精确  

---

# 7. 与 2D Diffusion Policy 对比（核心部分）

| 项目 | 2D DP | 3D-DP |
|------|-------|--------|
| 输入 | RGB 图像 | 点云 / voxel / 3D 特征场 |
| 空间理解 | 间接、弱 | **直接 3D-aware** |
| 遮挡 | 容易失败 | 更鲁棒 |
| 抓取姿态 | 依赖图像 | **直接从点云估计** |
| 插入/旋转 | 难 | **显著更稳** |
| 多物体交互 | 弱 | **强** |
| 误差来源 | 投影误差、视角变化 | 真实空间一致性 |

一句话：

> [!tip] **3D-DP = 让机器人策略真正理解世界几何结构。**

---

# 8. 实验表现（统一总结）

3D-DP 在以下任务显著优于 2D DP：

- drawer opening  
- peg insertion  
- socket insertion  
- lid opening  
- pushing with obstacles  
- manipulating unseen geometries  

典型提升：

- 成功率 +20%〜50%  
- OOD generalization 明显更强  
- failure mode 更少（尤其是角度不准、对接不准）

---

# 9. 与 TT / 3D Diffuser Actor 的关系

### 与 TT 的关系
- TT：**轨迹级生成**（但离散）  
- 3D-DP：**轨迹分布生成**（连续）  

两者直观上属于同一个“trajectory generative control”族系。

### 与 3D Diffuser Actor 的关系  
3DDA 是 3D-DP 的最系统化版本之一。  
区别在于：

- 3DDA 强调 voxel feature volume + U-Net  
- 部分 3D-DP 使用 point transformer  
- 3DDA 增加 SE(3) consistency modules  
- 3DDA 的实验是目前最完整的 benchmark

可以理解为：

> 3D-DP 是方向  
> **3D Diffuser Actor 是其中最成熟的工程实现。**

---

# 10. 我该记住的关键公式

### Diffusion step:

$$
x_{t} = \sqrt{\alpha_t} x_{t-1} + \sqrt{1-\alpha_t}\epsilon
$$

### Denoise:

$$
x_{t-1} = f_\theta(x_t, F_{3D}, t)
$$

### Action:

$$
a_t = (x_0)_1
$$

---

# 11. 复习 Checklist（含答案）

> Q：为什么必须使用 3D？  
A：操控是 3D 几何问题，2D RGB 无法提供物体位置/姿态/深度。

> Q：3D-DP 的输入是什么？  
A：点云 / voxel / TSDF / 3D feature field。

> Q：3D-DP 如何生成动作？  
A：扩散模型生成未来 H-step 动作轨迹。

> Q：为什么比 2D DP 稳定？  
A：去噪和生成过程持续参考 3D 场景特征。

> Q：与 3D Diffuser Actor 的关系？  
A：3DDA 是 3D-DP 的旗舰级实现，系统性最完整。

> Q：3D-DP 适合哪些任务？  
A：插入、旋钮、对接、需要姿态精度的 manipulation。

---

# 一句话总结

> **3D Diffusion Policy = 让扩散模型在“真实三维世界”中生成机器人动作。  
> 它是从 2D DP → 真正具身智能（3D-aware control）的关键技术跨越。**

