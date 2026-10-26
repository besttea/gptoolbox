# 70 四元数 / Quaternions

> 本章覆盖 `quat/` 目录下的四元数运算模块。该目录只有 **1 个 `.m` 文件**：`quatmultiply` 及其可选参数。
> 它是全库唯一的四元数算术原语，被 `mesh/` 中的前向运动学（`forward_kinematics`）、骨骼重定向（`retarget`）
> 与切片（`slice_triangles`、`slice_tets`）等函数调用。
>
> This chapter covers the `quat/` module. The directory contains exactly **one `.m` file**: `quatmultiply`.
> It is the toolbox's only quaternion arithmetic primitive, and it is consumed by `mesh/` functions such as
> `forward_kinematics`, `retarget`, `slice_triangles` and `slice_tets`.

- 语言 / Language：中英对照（中文为主）
- 相关章节 / See also：[70 四元数模块索引](README.md)、[10-mesh/16 刚体变换与四元数](../10-mesh/16-rigid-transforms-and-quaternion.md)、
  [80-python](../80-python/01-python-bindings.md)、[appendices/03-doc-template.md](../appendices/03-doc-template.md)
- 命名约定 / Notation：`q`、`Q`、`r` 均为 `M by 4` 四元数矩阵，**标量在前** `[w x y z]`（等价 `[1 i j k]`）
- 验证方式 / Verification：本章所有签名、行号、公式均**逐字取自源码**（读源核对），不做超出源码的演绎

---

## 本章概览 / Chapter overview

### (a) 关键约定：分量顺序 / Key convention: component order

> **结论：`quat/quatmultiply.m` 使用"标量在前"顺序 `[w x y z]`（Hamilton 约定），与全库一致。**
> 依据（均取自源码）：
>
> - 头注 L7-L8 明确写 "Q and R have their **scalar number as the first column**"
>   （[quat/quatmultiply.m L7-L8](file:///c:/VSWork/gptoolbox-1/quat/quatmultiply.m#L7-L8)）。
> - 向量部分代码把 `q(:,1)`、`r(:,1)` 当作**标量**分量使用（L62-L66），标量部分
>   `scalar = q(:,1).*r(:,1) − q(:,2).*r(:,2) − q(:,3).*r(:,3) − q(:,4).*r(:,4)`（L70-L71）；
>   若首列是 x 分量则该式的含义不成立，故首列必为 `w`。
> - 错误标识符为 `aero:quatnorm:*`（L37/L41/L49/L52/L55），表明该文件源自 **MATLAB Aerospace** 系列同名函数。
> - 与 [10-mesh/16](file:///c:/VSWork/gptoolbox-1/docs/10-mesh/16-rigid-transforms-and-quaternion.md#L18-L34)、
>   `mesh/axisangle2quat`、`mesh/quatslerp`、`external/quaternionrotate.m` 的 `[w xi yj zk]` 约定一致。
>
> All quaternions in this chapter follow the scalar-first Hamilton order `[w x y z]`.

### (b) Hamilton vs JPL 约定 / Hamilton vs JPL

| 维度 | 本文件（Hamilton） | JPL 约定（对照） |
|---|---|---|
| 分量顺序 | `[w x y z]`（标量在前） | 常见为 `[x y z w]`（标量在后，如 MATLAB Robotics/ROS） |
| 标量定义 | `w = cos(θ/2)` | `w = cos(θ/2)`（同），但**列位置不同** |
| 矢量部分符号 | 旋转向量用 `q v q*` | 视"旋转坐标系/旋转向量"选择 `q v q*` 或 `q* v q` |
| 乘法实现 | 哈密顿积（本文件 L62-L71） | 同构积，但列序不同致逐元素公式不同 |

- 本文件实现的是标准**哈密顿积** `q ⊗ r`：若 `q=(s1,v1)`、`r=(s2,v2)`，则
  `q⊗r = ( s1·s2 − v1·v2 , s1·v2 + s2·v1 + v1×v2 )`，与源码 L60-L71 逐项对应。
- 头注 L32 特别提示 "Quaternion multiplication is **not commutative**"（乘法不可交换）。
- 该文件**不强制单位长度**：末尾的单位化代码（L75-L87）被整段注释掉，因此连续相乘会产生数值漂移，
  调用方（如 `forward_kinematics`、`retarget`）需自行按需归一化（仓库中常配合 `normalizerow` 使用）。

### (c) 与 MATLAB 同名函数的关系 / Relation to MATLAB's same-named functions

- 本文件是 **MATLAB 系列 `quatmultiply` 的同名副本**（头注版权 `Copyright 2000-2006 The MathWorks, Inc.`，
  错误 ID 前缀 `aero:`）。MATLAB 中 `quatmultiply` 存在于 **Aerospace Toolbox / Aerospace Blockset** 与
  **Robotics System Toolbox**：
  - **Aerospace** 版与本文件**一致**：`M-by-4`、**标量在前** `[w x y z]`。
  - **Robotics System Toolbox** 版（机器人领域常用）为 `[x y z w]`（标量在后），**与本文件的列序不兼容**；
    两者**不可直接互换**（Phased Array System Toolbox 不提供 `quatmultiply`）。
- 本文件把该函数 **vendored（内联复制）到 `quat/`**，使 gptoolbox 在**没有安装上述工具箱**时也能使用，
  并保证全库列序统一为 `[w x y z]`。
- 头注 See also 列出的 `QUATCONJ`、`QUATDIVIDE`、`QUATINV`、`QUATMOD`、`QUATNORM`、`QUATNORMALIZE`、
  `QUATROTATE` **在 gptoolbox 仓库内均无对应文件**（见下"依赖"），它们是 Aerospace/Robotics 工具箱的自带函数。

### (d) 模块依赖图 / Module dependency diagram

```mermaid
flowchart TD
  Q1["mesh/forward_kinematics"] -->|Q(p,:) ⊗ dQ(b,:)| QM["quat/quatmultiply"]
  Q2["mesh/retarget"] -->|Qd ⊗ Qp| QM
  Q3["mesh/slice_triangles"] -->|Q ⊗ [0 M] ⊗ Q*| QM
  Q4["mesh/slice_tets (注释示例)"] -->|同 slice_triangles| QM
  Q5["mesh/pseudoedge_dof"] -->|Q(PE,:) ⊗ normalizerow(Qc(PE,:))| QM
  QM -->|"qout = [scalar vec]"| OUT["M by 4 四元数积 (w x y z)"]
```

> 说明：`quat/` **不依赖任何同库函数**，是依赖图中的叶子节点（纯 MATLAB 实现）。

---

## 逐函数条目 / Per-function entries

### `quatmultiply` — 四元数乘积 / Quaternion product
- **文件 / File**: [quat/quatmultiply.m](file:///c:/VSWork/gptoolbox-1/quat/quatmultiply.m#L1-L87)
- **签名 / Signature**: `qout = quatmultiply( q, varargin )`
- **功能 / Purpose**: `EN: Calculate the product of two quaternions (M-by-4 or 1-by-4, scalar number in the first column).` / 中文：计算两个四元数（`M by 4` 或 `1 by 4`，标量在第 1 列）的哈密顿乘积；未给出第二个操作数时对 `q` 自乘。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `q` | `M by 4` 实数 | 第一个四元数，标量在前 `[w x y z]` / first quaternion (scalar-first) |
  | `r`（`varargin{1}`，位置参数） | `M by 4` 或 `1 by 4` 实数 | 第二个四元数；缺省取 `r = q`（自乘）/ second quaternion, defaults to `q` |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `qout` | `M by 4` | 乘积 `[scalar vec]`，标量在前 / quaternion product (scalar-first) |

- **可选参数 / Optional**（本函数**无** `'Name',Value` 名值对；第二操作数是普通**位置**参数）:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `r`（第 2 个位置参数） | `M by 4` 或 `1 by 4` | `q`（未给时自乘） | 右乘操作数 / right-hand operand |

- **变量流动 / Variable flow**:
  `q,(r 或 q) →(校验 real / 4 列 / 行数相等或其一为 1)→ r →(L60-L66 向量部分: s1·v2 + s2·v1 + v1×v2)→ vec →`
  `(L68-L71 标量部分: s1·s2 − v1·v2)→ scalar →(L73 拼接)→ qout = [scalar vec]`
- **算法步骤 / Algorithm**:
  1. `error(nargchk(1, 2, nargin,'struct'))`：限制参数个数为 1~2（L34）。
  2. 校验 `q`：元素必须全为实数（否则 `error('aero:quatnorm:isnotreal1',...)`，L36-L38）；列数必须为 4（否则 `...wrongdim1`，L40-L42）。
  3. 若 `nargin == 1`，则 `r = q`（自乘 / 平方）；否则 `r = varargin{1}`，再校验 `r` 为实数（`...isnotreal2`）、列数为 4（`...wrongdim2`），并要求 `size(r,1)==size(q,1)` **或**二者之一为 1（允许单行**广播**，否则 `...wrongdim3`）（L44-L58）。
  4. 计算**向量部分** `vec`（L60-L66）：三项相加
     `vec = [q1·r2, q1·r3, q1·r4] + [r1·q2, r1·q3, r1·q4] + [q3·r4−q4·r3, q4·r2−q2·r4, q2·r3−q3·r2]`
     （即 `s1·v2 + s2·v1 + v1×v2`）。
  5. 计算**标量部分** `scalar = q1·r1 − q2·r2 − q3·r3 − q4·r4`（即 `s1·s2 − v1·v2`）（L68-L71）。
  6. `qout = [scalar vec]`（L73）。
  7. **不做单位化**：单位长度强制代码被整段注释（L75-L87），故输出不保证 `‖qout‖ = 1`。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 q (M by 4), 可选 r"] --> B["nargchk(1,2,nargin)"]
    B --> C{"isreal(q) && size(q,2)==4 ?"}
    C -- 否 --> E1["error aero:quatnorm:*"]
    C -- 是 --> D{"nargin == 1 ?"}
    D -- 是 --> D1["r = q (自乘)"]
    D -- 否 --> D2["r = varargin{1}"]
    D2 --> D3{"r 实数 && 4 列 && 行数相等或其一为 1 ?"}
    D3 -- 否 --> E2["error aero:quatnorm:*"]
    D3 -- 是 --> F
    D1 --> F["vec = s1·v2 + s2·v1 + v1×v2"]
    F --> G["scalar = s1·s2 − v1·v2"]
    G --> H["qout = [scalar vec]"]
    H --> Z["输出 qout (M by 4, w x y z)"]
  ```

- **依赖 / Dependencies**:
  - 本库依赖：**无**（纯 MATLAB，不调用任何同库函数）。
  - 非本库（MATLAB 内置/工具箱）：`error`、`nargchk`（**已过时**的内置函数，新版 MATLAB 仍可用但会告警）。
  - ⚠️ **头注 See also 中的同名函数在仓库内不存在**：`quatconj`、`quatdivide`、`quatinv`、`quatmod`、
    `quatnorm`、`quatnormalize`、`quatrotate` 均无对应 `.m` 文件（属 Aerospace/Robotics 工具箱内置），
    调用方若直接使用需自行确保环境提供。
- **See also**: `forward_kinematics`（[mesh/forward_kinematics.m L39](file:///c:/VSWork/gptoolbox-1/mesh/forward_kinematics.m#L39)）、
  `retarget`（[mesh/retarget.m L45/L53](file:///c:/VSWork/gptoolbox-1/mesh/retarget.m#L45)）、
  `pseudoedge_dof`（[mesh/pseudoedge_dof.m L72](file:///c:/VSWork/gptoolbox-1/mesh/pseudoedge_dof.m#L72)）、
  `slice_triangles`（[mesh/slice_triangles.m L33](file:///c:/VSWork/gptoolbox-1/mesh/slice_triangles.m#L33)）、
  `quat2mat`、`quatbetween`、`quatslerp`（均见 [10-mesh/16](../10-mesh/16-rigid-transforms-and-quaternion.md)）
- **备注 / Note**:
  - 本文件是 **MathWorks Aerospace 同名函数的 vendored 副本**（`Copyright 2000-2006 The MathWorks, Inc.`，
    `$Revision: 1.1.6.2 $`，L29-L30）；文件名与函数名**一致**（符合 `style.md` 命名规范）。
  - 与 **Robotics System Toolbox** 的同名 `quatmultiply`（`[x y z w]`，标量在后）**列序不兼容**，
    移植代码时勿直接互替。
  - 输出**未归一化**（L75-L87 被注释），长链相乘需调用方自行 `normalizerow`。

---

## 依赖汇总 / Dependencies summary

- **同库依赖**：无。`quat/` 是依赖图的叶子（无出边），仅被 `mesh/` 侧调用（入边见 `docs/00-overview/05-dependency-map.md` 的 E10）。
- **调用方（入边，节选）**：`mesh/forward_kinematics`（L39）、`mesh/retarget`（L45/L53）、
  `mesh/slice_triangles`（L33）、`mesh/pseudoedge_dof`（L72）；`mesh/slice_tets`（L59）与
  `mesh/dirac_eigs`（L115/L126）仅在**注释**中出现，非实际调用。
- **外部/内置**：`error`、`nargchk`（MATLAB 内置）；头注 See also 的 `quat*` 系列为工具箱函数，**不在本仓库**。

## 交叉引用 / Cross references

- 四元数与旋转表示总览（轴角 / 四元数 / 旋转矩阵 / 对偶四元数互转）：见 [10-mesh/16 刚体变换与四元数](../10-mesh/16-rigid-transforms-and-quaternion.md)
- 模块依赖图（`mesh → quat` 边）：见 [00-overview/05-dependency-map.md](../00-overview/05-dependency-map.md)
- 文档写作规范：见 [appendices/03-doc-template.md](../appendices/03-doc-template.md)

---

## 函数清单核对表 / Coverage Checklist

| 函数 (anchor) | 一句话用途 |
|---|---|
| [`quatmultiply`](file:///c:/VSWork/gptoolbox-1/quat/quatmultiply.m#L1-L87) | 两个 `[w x y z]` 四元数的哈密顿乘积（单参时自乘），不归一化 |
