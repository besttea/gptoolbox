# 70 四元数模块 / Quaternion Module

> `quat/` 是 gptoolbox 中**最小的功能模块**：仅 **1 个 `.m` 文件** `quatmultiply.m`，
> 提供全库唯一的四元数算术原语（哈密顿积）。它是依赖图中的**叶子节点**（无同库出边），
> 仅被 `mesh/` 侧若干函数调用。本目录下的 1 篇章节对该函数给出：签名、约定论证、
> 输入/输出/可选参数、变量流动链、算法步骤、Mermaid 流程图、依赖与调用方清单。
>
> `quat/` is the **smallest functional module** of gptoolbox: exactly **one `.m` file**,
> `quatmultiply.m`, providing the toolbox's only quaternion arithmetic primitive (Hamilton
> product). It is a **leaf** in the dependency graph (no same-library out-edges) and is called
> only by a handful of `mesh/` functions. The single chapter in this folder documents that
> function with signature, convention argument, I/O table, variable-flow chain, algorithm
> steps, Mermaid flowchart, dependencies and caller list.

- 语言 / Language：中英对照（中文为主）
- 覆盖 / Coverage：`quat/*.m` **1 / 1**（100%）
- 粒度 / Granularity：逐函数 API + 约定论证 + 算法 + 流程图 + 变量流动

---

## 1. 章节结构 / Chapter layout

| 章节 | 文件 | 函数数 | 主题 |
|---|---|---|---|
| 70.1 | [01-quaternion.md](01-quaternion.md) | 1 | 四元数乘积与分量顺序约定 / Quaternion product & component-order convention |
| — | **合计 / Total** | **1** | — |

---

## 2. 模块职责 / Module responsibility

- 提供**四元数乘法**（哈密顿积 `q ⊗ r`），供 `mesh/` 中的刚体运动链与切片运算使用：
  - **前向运动学**：`mesh/forward_kinematics` 逐骨累乘局部四元数增量 `Q(p,:) ⊗ dQ(b,:)`。
  - **骨骼重定向**：`mesh/retarget` 用 `Qd ⊗ Qp` 组合源/目标姿态，并用 `quatconj(Q) ⊗ (dQ ⊗ Q)`
    求取骨架相对变换。
  - **切片**：`mesh/slice_triangles`（及 `slice_tets` 的注释示例）用 `Q ⊗ [0 M] ⊗ Q*` 旋转切片平面坐标。
  - **伪边自由度**：`mesh/pseudoedge_dof` 用 `Q(PE,:) ⊗ normalizerow(Qc(PE,:))` 更新边的四元数状态。
- 该模块**不含**四元数构造（轴角→四元数在 `mesh/axisangle2quat`）、球面插值（`mesh/quatslerp`）、
  旋转矩阵互转（`mesh/quat2mat`、`mesh/quatbetween`）等——这些位于 `mesh/`（见 [10-mesh/16](../10-mesh/16-rigid-transforms-and-quaternion.md)）。

---

## 3. 全量函数索引 / Complete function index

| 函数 (anchor) | 文件 | 一句话功能 |
|---|---|---|
| [`quatmultiply`](file:///c:/VSWork/gptoolbox-1/quat/quatmultiply.m#L1-L87) | [quat/quatmultiply.m](file:///c:/VSWork/gptoolbox-1/quat/quatmultiply.m#L1-L87) | 两个 `[w x y z]` 四元数（`M by 4` 或 `1 by 4`）的哈密顿积；缺省第二操作数时**自乘**；**不归一化** |

---

## 4. 关键约定结论 / Key convention findings

> **结论：本模块使用"标量在前"顺序 `[w x y z]`（Hamilton 约定），与全库一致。**

| 维度 | 本模块（Hamilton） | 对照（JPL / Robotics） |
|---|---|---|
| 分量顺序 | `[w x y z]`（标量在第 1 列） | `[x y z w]`（标量在末列，如 MATLAB Robotics / ROS） |
| 乘法 | `q⊗r = (s1·s2 − v1·v2, s1·v2 + s2·v1 + v1×v2)` | 同构积，但列序不同致逐元素公式不同 |
| 是否归一化 | **否**（单位化代码 L75-L87 被整段注释） | Robotics 版按约定归一 |

依据（取自源码）：

- 头注 L7-L8 明写 "scalar number as the first column"（[quat/quatmultiply.m L7-L8](file:///c:/VSWork/gptoolbox-1/quat/quatmultiply.m#L7-L8)）。
- 标量部分 `scalar = q1·r1 − q2·r2 − q3·r3 − q4·r4`（L70-L71）仅在首列为 `w` 时成立。
- 错误 ID 前缀 `aero:`（L37/L41/L49/L52/L55）表明该文件源自 **MATLAB Aerospace** 同名函数。
- 与 `mesh/axisangle2quat`、`mesh/quatslerp`、`external/quaternionrotate.m` 的 `[w xi yj zk]` 约定一致
  （见 [90-external/02b](../90-external/02b-geometry-and-format-helpers.md)）。
- 与 **Robotics System Toolbox** 的同名 `quatmultiply`（`[x y z w]`）**列序不兼容**，移植时勿直接互替。

---

## 5. 依赖与构建 / Dependencies & build

- **同库依赖 / Same-library deps**：**无**。`quat/` 无出边，是依赖图叶子（见
  [00-overview/05-dependency-map.md](../00-overview/05-dependency-map.md#L100) 的 `E10` 边）。
- **调用方 / Callers（入边）**：

  | 调用点 | 用法 |
  |---|---|
  | [mesh/forward_kinematics.m#L39](file:///c:/VSWork/gptoolbox-1/mesh/forward_kinematics.m#L39) | `Q(b,:) = quatmultiply(Q(p,:),dQ(b,:))` |
  | [mesh/retarget.m#L45](file:///c:/VSWork/gptoolbox-1/mesh/retarget.m#L45) | `Q(q,:) = quatmultiply(Qd,Qp)` |
  | [mesh/retarget.m#L53](file:///c:/VSWork/gptoolbox-1/mesh/retarget.m#L53) | `dS = quatmultiply(quatconj(Q),quatmultiply(dQ,Q))` |
  | [mesh/slice_triangles.m#L33](file:///c:/VSWork/gptoolbox-1/mesh/slice_triangles.m#L33) | `W = quatmultiply(quatmultiply(Q,[zeros(...) M]),Q.*[1 -1 -1 -1])` |
  | [mesh/pseudoedge_dof.m#L72](file:///c:/VSWork/gptoolbox-1/mesh/pseudoedge_dof.m#L72) | `Q(PE(:),:) = quatmultiply(Q(PE(:),:),normalizerow(Qc(PE(:),:)))` |
  | [mesh/slice_tets.m#L59](file:///c:/VSWork/gptoolbox-1/mesh/slice_tets.m#L59) | 仅出现在注释示例中（`% ...`），非实际调用 |

  > 另：[mesh/dirac_eigs.m#L115-L126](file:///c:/VSWork/gptoolbox-1/mesh/dirac_eigs.m#L115-L126) 的注释中提及
  > "Vectorized quatmultiply"，但**不含实际调用**。

- **外部/内置 / Built-ins**：`error`、`nargchk`（**已过时**，新版 MATLAB 仍可用但会告警）。
- **头注 See also 缺失项 / Missing See-also targets**：`quatconj`、`quatdivide`、`quatinv`、
  `quatmod`、`quatnorm`、`quatnormalize`、`quatrotate` **在仓库内均无对应 `.m` 文件**
  （属 Aerospace/Robotics 工具箱内置），调用方若直接使用需自行确保环境提供。
- **构建 / Build**：**纯 MATLAB，无需编译**（不同于 [60-mex](../60-mex/README.md)）。

---

## 6. 交叉引用 / Cross references

- 本章章节：[70.1 四元数](01-quaternion.md)
- 四元数与旋转表示总览（轴角 / 四元数 / 旋转矩阵 / 对偶四元数互转）：[10-mesh/16](../10-mesh/16-rigid-transforms-and-quaternion.md)
- 模块依赖图（`mesh → quat` 边，`E10`）：[00-overview/05-dependency-map.md](../00-overview/05-dependency-map.md)
- 第三方四元数工具（`newquaternion`/`quaternionrotate`）：[90-external/02b](../90-external/02b-geometry-and-format-helpers.md)
- 文档写作规范：[appendices/03-doc-template.md](../appendices/03-doc-template.md)
