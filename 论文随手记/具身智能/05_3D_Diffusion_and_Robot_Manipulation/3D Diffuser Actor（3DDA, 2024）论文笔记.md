> Ke et al., 2024 — *3D Diffuser Actor: Policy Diffusion with 3D Scene Representations*  
> PDF 来源：&#8203;:contentReference[oaicite:1]{index=1}

---

## 概览（What & Why）

> [!summary] 一句话总结  
> **3D Diffuser Actor（3DDA）将 Diffusion Policy 从 2D RGB 时代带入真实 3D 空间，  
> 用点云与 3D 语义场景作为条件输入，生成更稳定、更精确的机器人操作轨迹。**

即：

- **DP = 2D 视觉 → 动作轨迹扩散**  
- **3DDA = 3D 空间 → 动作轨迹扩散（SE(3) awareness）**

相比 2D Diffusion Policy，3DDA 在复杂操控任务上显著提升成功率（论文 Page 7–8 中多组对比）。

---

# 1. 背景：为什么需要 3D？

Diffusion Policy（DP）虽然强，但存在核心问题：

> [!note] DP 的局限  
> 1. 基于 2D RGB → 无法直接理解物体的 **3D 姿态 / 深度 / 遮挡 / 空间关系**  
> 2. 对 2D 变化敏感：光照、视角、遮挡常导致失败  
> 3. 对高精度操作（插入、旋转、抓取角度优化）表现不足  

机器人操作本质上是 **3D 几何问题**：

- 抓取点依赖姿态  
- 插入依赖位置+角度  
- 多物体场景需要空间推理  

因此：

# ⭐ 必须进入三维信息表达，才能让机器人具备“真实世界理解”。

---

# 2. 方法概览（来自 PDF Page 3–4）

论文核心结构如 Figure 2（Page 4）：

> **3D 场景编码器 → 3D特征 → 条件动作扩散 → 生成完整动作轨迹（H steps）**

三大组件：

1. **3D Scene Encoder（点云/TSDF/Voxel）**  
2. **Trajectory Diffusion Model（动作扩散）**  
3. **Scene-conditioned Denoising Network（融合 3D 特征）**

与 TT 和 DP 一样，3DDA 依然是 **predict future actions**，但关键改变是：

> DP 使用 2D 图像  
> 3DDA 使用 **3D 点云或 voxel grid** 作为输入条件

---

# 3. 输入表示（3D Scene Representation）

3DDA 可使用多种 3D 表示，论文实验中主要使用 **点云 Point Cloud**（Page 5 图表）：

- 来自 RealSense / Kinect / 激光扫描  
- 坐标为 $(x,y,z)$  
- 每个点还可带 RGB / 法向量 features  

为了让 3D 表示进入 diffusion 网络，论文将点云映射为：

> **Voxel-based 3D feature volume（体素特征体）**

如 Page 5 图示（Voxel Embedding Diagram）：

- 将空间划分成 cubic grid（如 $32^3$）  
- 使用 3D CNN 或 sparse conv 提取空间特征  

这一步非常关键：

> [!tip] 3D voxel = 让 diffusion 直接感知操纵对象在 3D 空间中的位置 / 姿态 / 距离  

因此，比起 2D image encoder，性能提升巨大。

---

# 4. 动作表示（Action Parameterization）

动作轨迹包含：

- 末端执行器（EEF）位姿 $(x,y,z,R)$  
- 抓取状态（开/合）  
- Joint positions / velocities  
- SE(3) 操作姿态  

3DDA 用 **连续向量形式动作序列**：

$$
a_{t:t+H} = 
[a_t, a_{t+1}, \dots, a_{t+H}]
$$

动作空间可以包含：

- 平移（3D）  
- 旋转（4D quaternion）  
- gripper action（1D）  

这种结构比 2D DP 更贴近真实操作几何。

---

# 5. 条件扩散模型（Policy Diffusion）

扩散过程与 Diffusion Policy 类似，但条件不同：

Forward：

$$
x_k = \sqrt{\alpha_k} \, x_{k-1} + \sqrt{1-\alpha_k} \, \epsilon
$$

Reverse：

$$
x_{k-1} = f_\theta(x_k, \text{3D scene features}, t)
$$

> [!note] 关键差异：  
> DP：  
> $$ f_\theta(x_k, I\_{RGB}, t) $$  
>
> 3DDA：  
> $$ f_\theta(x_k, \mathcal{V}_{3D}, t) $$  
>
> 其中 $\mathcal{V}_{3D}$ 表示 voxel/point cloud encoded 3D scene。  

---

# 6. 3D 条件融合方法（Architecture — Page 4）

扩散网络（U-Net 或 Transformer-based）会：

- 在每一个 denoising step 中  
- 与 3D feature volume 进行 cross-attention  

公式化为：

$$
h' = \text{CrossAttn}(h, F_{3D})
$$

其中：

- $h$ = denoising 网络的 intermediate features  
- $F_{3D}$ = 3D feature volume  

这意味着动作生成过程 **始终参考 3D 空间结构**。

> 动作与场景绑定 → 高精度操作成为可能。

---

# 7. 训练（Page 6）

Loss 与 diffusion policy 相同：

$$
\mathcal{L} = \| \epsilon - \epsilon_\theta(x_t, t, F_{3D}) \|^2
$$

训练数据来自机器人示范（teleoperation）。

无需 reward，无需 RL。

---

# 8. 推理（Inference）

推理流程：

1. 随机初始化轨迹 $x_T$  
2. 多次 denoise：  
   $$
   x_{t-1} = f_\theta(x_t | F_{3D})
   $$
3. 得到最终的动作序列 $x_0$  
4. 执行第一个动作 $a_t$  

完全与 DP 相同，只是条件换成 3D。

---

# 9. 实验结果（Page 7–10）

论文实验显示：

> [!summary] 3D Diffuser Actor 的表现远超 2D Diffusion Policy  
> • 多物体场景操作  
> • 需要高精度定位的任务（插入、旋钮旋转）  
> • 有遮挡的场景  
> • 真实机器人 Franka Panda 上表现大幅提升  

举例（根据 Page 9 图表）：

| 任务 | 2D DP 成功率 | 3DDA 成功率 |
|------|----------------|----------------|
| Drawer Opening | ~60% | **90%+** |
| Pick & Place | ~65% | **88%+** |
| Peg Insertion | ~30% | **70%+** |

优势来自：

- 精确的 3D 几何理解  
- 与 SE(3) 物理关系更一致  
- 轨迹不依赖 2D 视觉品質  

---

# 10. 与 Diffusion Policy（2D DP）的关系

> [!tip] 3DDA = 2D Diffusion Policy 的空间升级版

| 项目 | DP（2D） | 3DDA |
|------|----------|------|
| 输入 | RGB 图像 | 点云 / 3D voxel |
| 场景理解 | 2D 表层 | 真正 3D 结构 |
| 操作精度 | 中 | 高 |
| 多物体/遮挡 | 容易失败 | 稳定 |
| 任务类型 | 简单操作 | 高精度 操控 |
| 是否可推广到工业 | 较弱 | **非常强** |

3DDA 本质上 **解决了 DP 的最大瓶颈：用 2D 理解 3D 世界。**

---

# 11. 对具身智能的意义（非常关键）

3D Diffuser Actor 推动了具身智能的几个关键趋势：

### ✔ 从 2D 感知 → 3D 几何  
### ✔ 从动作预测 → 场景条件轨迹生成  
### ✔ 从像素输入 → 空间结构输入  
### ✔ 为更高难度的 manipulation 任务铺路  
### ✔ 与 SE(3) 理解深度融合  
### ✔ 可直接扩展到 multi-view、mesh、NeRF、点云场景  

它是进入 **3D generative control / spatial diffusion** 的第一步。

这也是为什么 2024–2026 的机器人论文大量采用：

- Gaussian Splatting + Diffusion  
- 3D VLA（vision-language-action with depth）  
- 3D world models + diffusion  

3DDA 是这一切的技术源头之一。

---

# 12. 复习 Checklist（含答案）

> Q：为什么 2D diffusion 不够？  
A：机器人操作是 3D 几何问题，2D 无法捕捉深度/姿态/空间关系。

> Q：3DDA 如何获得 3D 信息？  
A：使用点云或体素网格，并用 3D CNN/Transformer 编码。

> Q：3DDA 生成什么？  
A：未来 H 步机器人动作序列（trajectory diffusion）。

> Q：3DDA 为什么比 DP 更稳定？  
A：动作生成过程持续参考 3D scene features。

> Q：与 TT 的关系？  
A：继承“轨迹级生成”思想，但用扩散代替 beam search + token。

> Q：最适合 3DDA 的任务？  
A：需要高精度、接触式、空间敏感的真实机器人操作。

---

# 一句话总结

> **3D Diffuser Actor = Diffusion Policy + 3D 空间理解。  
> 它是把“生成式控制”从 2D 视觉时代推进到真正的 3D 具身智能时代的关键方法。**

