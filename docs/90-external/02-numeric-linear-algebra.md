# 90.2 数值线性代数辅助（第三方） / Numeric Linear-Algebra Helpers (Vendored)

> 本章覆盖 `external/` 下一组**数值/线性代数**第三方 vendored 代码：仿射平面拟合（`affine_fit`）、
> 批量 2×2 / 3×3 解析求逆（`inv2`/`inv3`）、带列置换的 LU 分解（`luq`）、稀疏零空间与四基本子空间
> （`spnull`/`spspaces`）、组合枚举（`combn`）、截断正态采样（`trandn`），以及结构相似度指标
> （`ssim`/`ssim_index`）。其中大部分**并非几何算法本身**，而是服务几何/矩阵计算的底层数值工具。
>
> This chapter covers a set of **numeric / linear-algebra** vendored third-party files in `external/`:
> affine plane fitting (`affine_fit`), batched closed-form 2×2 / 3×3 inversion (`inv2`/`inv3`),
> LU with column permutation (`luq`), sparse null and four fundamental subspaces (`spnull`/`spspaces`),
> combinatorial enumeration (`combn`), truncated-normal sampling (`trandn`) and the structural
> similarity index (`ssim`/`ssim_index`). Most of these are **not geometry algorithms per se**, but
> low-level numeric tools used by geometry/matrix code.

- 语言 / Language：中英对照（中文为主）
- 相关章节 / See also：[90-external/README.md](README.md)、[90.1 色图与颜色](01-colormaps-and-color.md)、
  [90.3 MATLAB 环境与绘图](03-matlab-environment-and-plotting.md)、
  [appendices/03-doc-template.md](../appendices/03-doc-template.md)
- 重要声明 / Disclaimer：`external/` 为**第三方 vendored 代码**，函数名/作者/许可由**文件头**决定；本节忠实
  记录文件头所述署名，并在发现**与本库其它模块功能重叠**或**依赖外部 MATLAB 工具箱**时明确标注。

---

## 本章概览 / Chapter Overview

- 本章 10 个文件分为四类：(1) **拟合/求逆** `affine_fit`、`inv2`、`inv3`；(2) **分解/子空间**
  `luq`、`spspaces`、`spnull`；(3) **枚举/采样** `combn`、`trandn`；(4) **图像质量指标** `ssim`、`ssim_index`。
- 三者构成一条依赖链：`spnull → spspaces → luq`；其余文件彼此独立。
- `ssim` 与 `ssim_index` 是**同一 SSIM 的两份近似重复实现**（分别对应"带/不带自动降采样"），二者**互不调用**。
- 多位作者：`inv2`/`inv3` 为 Bruno Luong；`luq`/`spspaces` 为 Pawel Kowal（LREM_SOLVE，**仅限非商业学术**）；
  `ssim`/`ssim_index` 源自 Zhou Wang 并重写；`trandn` 为 Zdravko I. Botev；`combn` 为 Jos van der Geest；
  `affine_fit` 为 Adrien Leygue；`spnull` 为 Jan Schellenberger。

### 函数总表 / Function overview

| 函数 / Function | 来源 / 署名 (Attribution) | 一句话功能 / One-line purpose |
|---|---|---|
| `affine_fit` | Adrien Leygue（2013，BSD 式） | 点云最小二乘拟合仿射平面（法向 + 平面内正交基） |
| `inv2` | Bruno Luong（2010，头无许可文本） | 批量 2×2 矩阵解析求逆 / 求解 `A\B` |
| `inv3` | Bruno Luong（2010，头无许可文本） | 批量 3×3 矩阵解析求逆 / 求解 `A\B` |
| `luq` | Pawel Kowal（2006，LREM_SOLVE，非商业学术） | 带列置换的 LU 分解 `A=L[Ub 0;0 0]Q` |
| `spnull` | Jan Schellenberger（2009，头无许可文本） | 稀疏矩阵快速零空间基（类 `null(S,'r')`） |
| `spspaces` | Pawel Kowal（2006，LREM_SOLVE，非商业学术） | 稀疏矩阵四基本子空间（经 `luq`） |
| `combn` | Jos van der Geest（2013，BSD） | 向量元素的 N 元（可重复）组合枚举 |
| `trandn` | Zdravko I. Botev（2016，BSD 式） | 截断正态分布高效批量采样 |
| `ssim` | Zhou Wang（2009，仅教育/研究）；Turcotte & Robidoux 重写 | 带自动降采样的 SSIM 指标 |
| `ssim_index` | Zhou Wang（2003，仅教育/研究）；Turcotte & Robidoux 重写 | 无自动降采样的 SSIM 指标 |

---

## 逐函数详解 / Per-function reference

### `affine_fit` — 点云最小二乘仿射平面拟合

- **文件 / File**: [external/affine_fit.m](file:///c:/VSWork/gptoolbox-1/external/affine_fit.m#L1-L60)
- **署名 / Attribution**: 作者 **Adrien Leygue**（文件头 `Author: Adrien Leygue`，`Date: August 30 2013`）；许可：**BSD 式**条款（`Copyright (c) 2013, Adrien Leygue`，保留声明即可再分发）。
- **签名 / Signature**: `[n,V,p] = affine_fit(X,W)`
- **功能 / Purpose**: `EN: Weighted least-squares (total least squares) affine subspace fit to a point cloud — unit normal n, orthonormal in-plane basis V and a point p on the plane; dimension-agnostic.` / 中文：对点云做（加权）最小二乘/总体最小二乘仿射子空间拟合，返回单位法向 `n`、平面内正交基 `V` 与平面上一代表点 `p`，维度不限（2/3/4/…）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `X` | `#X by dim` | 每行为一个样本点 / each row is a sample point |
  | `W` | `#X by 1`（可选） | 样本权重 / sample weights |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `n` | `dim by 1` | 单位法向 / unit normal |
  | `V` | `dim by dim-1` | 平面内正交基（列为特征向量）/ orthonormal in-plane basis |
  | `p` | `1 by dim` | 平面上一代表点（加权质心）/ a point on the plane |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `W` | `#X by 1` 数值 | `1/size(X,1)*ones(size(X,1),1)` | 均匀权重 / uniform weights |

- **变量流动 / Variable flow**: `X,W →(1 归一化权重)→ W →(2 加权质心)→ p →(3 去心加权)→ R →(4 eig(R'*R))→ V,D →(5 取列)→ n,V`
- **算法步骤 / Algorithm**:
  1. 未给 `W` 时取均匀权重；归一化 `W = W(:)./sum(W)`（L19–L22）。
  2. 加权质心 `p = sum(bsxfun(@times,W,X))` 必落在拟合仿射子空间上（L25）。
  3. 去心并加权得到散布矩阵输入 `R`（L28）。
  4. 对 `R'*R` 做**特征分解** `eig`：最小特征值对应单位法向 `n=V(:,1)`，其余列构成平面正交基 `V=V(:,2:end)`（L30–L33）。
  5. 注：文件头称其为**最小化到平面的法向距离平方**（即最小二乘仿射映射 `y=A x+b` 的几何等价）；维度通过 `bsxfun` 广播隐式检查（要求 `W` 长度为 `#X`），实际采用的是散布矩阵的 `eig` 而非显式 `svd`。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 X,W"] --> B{"给了 W ?"}
    B -- 否 --> B1["W = 1/N (均匀)"]
    B -- 是 --> C
    B1 --> C["W = W/sum(W)"]
    C --> D["p = sum(W.*X)"]
    D --> E["R = (X-p).*W"]
    E --> F["eig(R'*R) → V,D"]
    F --> Z["n = V(:,1); V = V(:,2:end)"]
  ```

- **依赖 / Dependencies**: 无同库依赖（MATLAB 内置 `eig`、`bsxfun`）。
- **See also**: [matrix/affine_null_space.m](file:///c:/VSWork/gptoolbox-1/matrix/affine_null_space.m#L1)（仿射零空间）、[external/inv3.m](file:///c:/VSWork/gptoolbox-1/external/inv3.m#L1-L59)。

### `inv2` — 批量 2×2 矩阵解析求逆 / 求解

- **文件 / File**: [external/inv2.m](file:///c:/VSWork/gptoolbox-1/external/inv2.m#L1-L46)
- **署名 / Attribution**: 作者 **Bruno Luong**（`brunoluong@yahoo.com`，`Original 27-May-2010`）；许可：文件头**未附许可证文本** / no licence text in header。
- **签名 / Signature**: `X = inv2(A, B, varargin)`
- **功能 / Purpose**: `EN: One-shot closed-form inverse of a stack of 2×2 matrices, or solves the stacked systems A(:,:,k)\B(:,:,k) when B is given.` / 中文：一次性对一批 2×2 矩阵做解析求逆；若给定 `B`，则批量求解 `A(:,:,k)\B(:,:,k)`。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `A` | `2 by 2 by n` | 待求逆矩阵堆栈 / stack of 2×2 matrices |
  | `B` | `2 by p by n`（可选） | 右侧多项 / right-hand sides |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `X` | `2 by 2 by n` 或 `2 by p by n` | `inv(A(:,:,k))` 或 `A(:,:,k)\B(:,:,k)` |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `B` | `2 by p by n` | 不提供 | 提供时从"求逆"切换为"求解" / solve mode |
  | `varargin` | 任意 | 未使用 | 仅为占位，函数体未读取 / unused placeholder |

- **变量流动 / Variable flow**: `A →(1 reshape 4×n)→ A' →(2 伴随矩阵 [d,-b;-c,a])→ X →(3 行列式 D)→ D →(4 1/D 缩放 + reshape)→ X →(5 有 B: mtimesx(X,B,'speed'))→ X`
- **算法步骤 / Algorithm**:
  1. 校验 `size(A,1)==size(A,2)==2`，否则 `error('A must be [2x2xn] array')`（L27–L29）。
  2. `A = reshape(A,4,[]).'` 把堆栈展平为 `n by 4`（L31）。
  3. 直接构造 **伴随矩阵**（互换对角、取反副对角）`[d -b -c a]`（L33）。
  4. 行列式 `D = A(:,1).*A(:,4) - A(:,2).*A(:,3)`，`bsxfun(@times,X,1./D)` 归一化（L36–L38）。
  5. `reshape(X.',2,2,[])` 还原堆栈（L40）。
  6. 若 `nargin>=2`，`X = mtimesx(X,B,'speed')` 批量右乘实现 `A\B`（L42–L44）；头注提醒**病态矩阵下不如反斜杠精确**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 A,B?"] --> B{"size(A)==2x2xn ?"}
    B -- 否 --> B1["error 抛出"]
    B -- 是 --> C["reshape → n by 4"]
    C --> D["伴随矩阵 + 行列式 D"]
    D --> E["X = adj/D"]
    E --> F["reshape → 2x2xn"]
    F --> G{"给了 B ?"}
    G -- 是 --> G1["X = mtimesx(X,B)"]
    G -- 否 --> Z["输出 X"]
    G1 --> Z
  ```

- **依赖 / Dependencies**: **`mtimesx`**（James Tursa 的第三方 MEX，File Exchange 25977，**仓库内不存在**，仅在提供 `B` 时需要）。
- **See also**: [external/inv3.m](file:///c:/VSWork/gptoolbox-1/external/inv3.m#L1-L59)（3×3 版）、`eig3`（头注提及，若存在）。

### `inv3` — 批量 3×3 矩阵解析求逆 / 求解

- **文件 / File**: [external/inv3.m](file:///c:/VSWork/gptoolbox-1/external/inv3.m#L1-L59)
- **署名 / Attribution**: 作者 **Bruno Luong**（`brunoluong@yahoo.com`，`Original 27-May-2010`）；许可：文件头**未附许可证文本** / no licence text in header。
- **签名 / Signature**: `X = inv3(A, B, varargin)`
- **功能 / Purpose**: `EN: One-shot closed-form inverse of a stack of 3×3 matrices via adjugate/determinant, or stacked solve when B is given.` / 中文：用伴随矩阵/行列式一次性对一批 3×3 矩阵求逆；给定 `B` 时批量求解 `A(:,:,k)\B(:,:,k)`。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `A` | `3 by 3 by n` | 待求逆矩阵堆栈 / stack of 3×3 matrices |
  | `B` | `3 by p by n`（可选） | 右侧多项 / right-hand sides |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `X` | `3 by 3 by n` 或 `3 by p by n` | `inv(A(:,:,k))` 或 `A(:,:,k)\B(:,:,k)` |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `B` | `3 by p by n` | 不提供 | 提供时从"求逆"切换为"求解" / solve mode |
  | `varargin` | 任意 | 未使用 | 仅为占位 / unused placeholder |

- **变量流动 / Variable flow**: `A →(1 reshape 9×n)→ A' →(2 三个 2×2 子式 m1,m2,m3)→ m1,m2,m3 →(3 伴随矩阵 9 列)→ X →(4 行列式 D)→ D →(5 1/D 缩放 + reshape)→ X →(6 有 B: mtimesx(X,B,'speed'))→ X`
- **算法步骤 / Algorithm**:
  1. 校验 `A` 为 `3x3xn`，否则 `error('A must be [3x3xn] array')`（L27–L29）。
  2. `A = reshape(A,9,[]).'` 展平（L31）。
  3. 计算三个 2×2 子式 `m1,m2,m3`（L33–L36），再展开得到完整 **伴随矩阵** 的 9 列 `X`（L38–L46）。
  4. 行列式 `D = A(:,1).*m1 - A(:,4).*m2 + A(:,7).*m3`，`1./D` 缩放（L48–L51）。
  5. `reshape(X.',3,3,[])` 还原（L53）。
  6. 若 `nargin>=2`，`X = mtimesx(X,B,'speed')` 实现批量 `A\B`（L55–L57）；头注同样提醒病态时不如反斜杠精确。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 A,B?"] --> B{"size(A)==3x3xn ?"}
    B -- 否 --> B1["error 抛出"]
    B -- 是 --> C["reshape → n by 9"]
    C --> D["子式 m1,m2,m3"]
    D --> E["伴随矩阵 9 列 + 行列式 D"]
    E --> F["X = adj/D → reshape 3x3xn"]
    F --> G{"给了 B ?"}
    G -- 是 --> G1["X = mtimesx(X,B)"]
    G -- 否 --> Z["输出 X"]
    G1 --> Z
  ```

- **依赖 / Dependencies**: **`mtimesx`**（James Tursa 第三方 MEX，仓库内不存在，仅在提供 `B` 时需要）。
- **See also**: [external/inv2.m](file:///c:/VSWork/gptoolbox-1/external/inv2.m#L1-L46)（2×2 版）、`eig3`（头注提及，若存在）、[matrix/diag_blk_inv.m](file:///c:/VSWork/gptoolbox-1/matrix/diag_blk_inv.m#L1)。

### `luq` — 带列置换的 LU 分解

- **文件 / File**: [external/luq.m](file:///c:/VSWork/gptoolbox-1/external/luq.m#L1-L147)
- **署名 / Attribution**: 作者 **Pawel Kowal**（`pkowal3@sgh.waw.pl`，`Copyright (c) Pawel Kowal (2006)`）；许可：**LREM_SOLVE toolbox，仅限非商业学术使用**（`free for noncommercial academic use only`）。
- **签名 / Signature**: `[L,U,Q] = luq(A,do_pivot,tol)`
- **功能 / Purpose**: `EN: LU decomposition with column permutation yielding A = L*[Ubar 0; 0 0]*Q, where Ubar is square and invertible.` / 中文：带列置换的 LU 分解，得到 `A = L [Ubar 0; 0 0] Q`，其中 `Ubar` 为可逆方阵，`L`、`Q` 可逆。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `A` | `n by m`（稀疏或稠密） | 待分解矩阵 / matrix to factor |
  | `do_pivot` | `scalar`（0/1） | 是否做列置换 / column pivoting flag |
  | `tol` | `scalar` | 判定零/非零的容差 / zero-nonzero tolerance |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `L` | `n by n` | 左可逆因子 / invertible factor |
  | `U` | `n by m` | 分块上三角（含 `Ubar`）/ block upper factor |
  | `Q` | `m by m` | 右（列置换）可逆因子 / right factor |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `do_pivot` | `0` / `1` | **必填**（无默认） | `1` 带列置换，`0` 不带 / with or without pivoting |
  | `tol` | `scalar` | **必填**（无默认） | 分离零/非零的容差 / tolerance |

- **变量流动 / Variable flow**: `A,do_pivot,tol →(1 sparse(A))→ A →(2 lu 分解)→ L,U,P,Q →(3 补齐方阵)→ L,U,Q →(4 找对角可逆块 I)→ I,Jl,Jq →(5 消去右下/左下非零块)→ Ubar2,Lbar1,Qbar1 →(6 递归 luq 处理零对角块)→ L,U,Q`
- **算法步骤 / Algorithm**:
  1. 强制稀疏 `sparse(A)`；处理 `0` 行/列特例（返回 `speye`）（L30–L50）。
  2. `do_pivot` 时 `[L,U,P,Q]=lu(A)` 并取 `Q=Q'`；否则 `[L,U,P]=lu(A)`、`Q=speye(m)`（L55–L61）。
  3. 用 `P'*L`、`P(n-p+1:n,:)'`、`[U;sparse(p,m)]` 把矩形的 LU 补齐成方阵 `L,U,Q`（L62–L65）。
  4. 取 `S=diag(U)`，以 `abs(S)>tol` 找可用主对角块索引 `I`，补集为 `Jl`（行）、`Jq`（列）（L70–L79）。
  5. 用 `Ubar1'\U(Jl,I)'` 与 `Ubar1\U(I,Jq)` 消去 `U(I,Jq)`、`U(Jl,I)` 块，更新 `Ubar2`、`Lbar1`、`Qbar1`（L92–L102）。
  6. 在 `Ubar2` 上找零对角但含非零元的块 `A=Ubar2(I2,I5)`，**递归**调用 `luq(...,do_pivot,tol)`，再拼装最终 `L,U,Q`（L114–L147）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 A,do_pivot,tol"] --> B["A = sparse(A)"]
    B --> C{"空矩阵特例 ?"}
    C -- 是 --> C1["L=Q=speye; U=A; return"]
    C -- 否 --> D{"do_pivot ?"}
    D -- 是 --> D1["lu(A) → L,U,P,Q"]
    D -- 否 --> D2["lu(A) → L,U,P; Q=speye"]
    D1 --> E["补齐方阵 L,U,Q"]
    D2 --> E
    E --> F["I = abs(diag(U))>tol"]
    F --> G["消去右下/左下非零块 → Ubar2,Lbar1,Qbar1"]
    G --> H{"Ubar2 含零对角非零块 ?"}
    H -- 是 --> H1["递归 luq(A子块)"]
    H1 --> Z["拼装输出 L,U,Q"]
    H -- 否 --> Z
  ```

- **依赖 / Dependencies**: 自身（递归调用）；MATLAB 内置 `lu`、`sparse`、`speye`、`diag`。
- **See also**: [external/spspaces.m](file:///c:/VSWork/gptoolbox-1/external/spspaces.m#L1-L106)、[matrix/affine_null_space.m](file:///c:/VSWork/gptoolbox-1/matrix/affine_null_space.m#L1)（`'luq'` 方法直接使用本函数）、[matrix/lexmin.m](file:///c:/VSWork/gptoolbox-1/matrix/lexmin.m#L1)。

### `spnull` — 稀疏矩阵快速零空间基

- **文件 / File**: [external/spnull.m](file:///c:/VSWork/gptoolbox-1/external/spnull.m#L1-L20)
- **署名 / Attribution**: 作者 **Jan Schellenberger**（`10/20/2009`），基于 MathWorks File Exchange 11120 "null-space of a sparse matrix"；许可：文件头**未附许可证文本** / no licence text in header。
- **签名 / Signature**: `N = spnull(S, tol)`
- **功能 / Purpose**: `EN: Computes a sparse (rref-like, non-orthonormal) null-space basis of a sparse matrix; much faster than null(S), with lower numerical accuracy.` / 中文：计算稀疏矩阵的稀疏零空间基（类似 `null(S,'r')`，基非正交、速度快、精度略低）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `S` | `m by n`（稀疏） | 目标稀疏矩阵 / sparse matrix |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `N` | `n by k`（稀疏） | 零空间基（非正交）/ sparse null basis |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `tol` | `scalar` | `1e-9` | 零化阈值 / zeroing tolerance |

- **变量流动 / Variable flow**: `S,tol →(1 spspaces(S,2,tol))→ SpRight →(2 取 SpRight{1}(:,SpRight{3}))→ N →(3 abs(N)<tol 置零)→ N`
- **算法步骤 / Algorithm**:
  1. 默认 `tol=1e-9`（L14–L16）。
  2. 调用 `spspaces(S,2,tol)`（`opt=2` 只求**右侧**零/值域空间，内部经 `luq` 的**最小次数**列置换实现稀疏基）（L17）。
  3. `N = SpRight{1}(:,SpRight{3})` 取出右侧零空间对应列，再将 `abs(N)<tol` 的元素精确置零，得到稀疏基（L18–L19）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 S,tol?"] --> B{"给了 tol ?"}
    B -- 否 --> B1["tol = 1e-9"]
    B -- 是 --> C
    B1 --> C["spspaces(S,2,tol) → SpRight"]
    C --> D["N = SpRight{1}(:,SpRight{3})"]
    D --> E["abs(N)<tol 置零"]
    E --> Z["输出稀疏基 N"]
  ```

- **依赖 / Dependencies**: `spspaces`（进而 `luq`）。
- **See also**: [external/spspaces.m](file:///c:/VSWork/gptoolbox-1/external/spspaces.m#L1-L106)、[matrix/pagenull.m](file:///c:/VSWork/gptoolbox-1/matrix/pagenull.m#L1-L16)、[matrix/affine_null_space.m](file:///c:/VSWork/gptoolbox-1/matrix/affine_null_space.m#L1)、[mesh/betti.m](file:///c:/VSWork/gptoolbox-1/mesh/betti.m#L1)（调用 `spnull`）。
- **重叠 / Overlap**: [matrix/pagenull.m](file:///c:/VSWork/gptoolbox-1/matrix/pagenull.m#L1-L16)（堆叠矩阵零空间，基于 `pagesvd`，**精确**）与 [matrix/affine_null_space.m](file:///c:/VSWork/gptoolbox-1/matrix/affine_null_space.m#L1) 为**本库自有**同类功能；`spnull` 的优势是**稀疏 + 快**，代价是精度与非正交基。

### `spspaces` — 稀疏矩阵四基本子空间

- **文件 / File**: [external/spspaces.m](file:///c:/VSWork/gptoolbox-1/external/spspaces.m#L1-L106)
- **署名 / Attribution**: 作者 **Pawel Kowal**（`pkowal3@sgh.waw.pl`，`Copyright (c) Pawel Kowal (2006)`）；许可：**LREM_SOLVE toolbox，仅限非商业学术使用**。
- **签名 / Signature**: `[SpLeft, SpRight] = spspaces(A, opt, tol)`
- **功能 / Purpose**: `EN: Finds the left and right null and range spaces of a sparse matrix, built on luq.` / 中文：基于 `luq` 求稀疏矩阵的**四基本子空间**（左/右零空间与左/右值域空间）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `A` | `n by m`（稀疏） | 目标稀疏矩阵 / sparse matrix |
  | `opt` | `1`/`2`/`3` | `1` 左空间、`2` 右空间、`3` 双侧 / which spaces |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `SpLeft` | `1 by 4` cell（`{}` 若 `opt=2`） | `{Q, I(值域行), J(左零空间行), Q}` / left range & null |
  | `SpRight` | `1 by 4` cell（`{}` 若 `opt=1`） | `{Q^-1, I(值域列), J(右零空间列), Q}` / right range & null |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `opt` | `1` / `2` / `3` | **必填**（无默认） | 计算哪些空间 / spaces to compute |
  | `tol` | `scalar` | `max(max(size(A))*norm(A,1)*eps, 100*eps)` | 判定零/非零的容差 |

- **变量流动 / Variable flow**: `A,opt,tol →(1 switch opt)→ calc_left,calc_right →(2 luq(A,0,tol))→ L,U,Q →(3 左: LL=L^-1, S=max(abs(U),[],2))→ I,J →SpLeft →(4 右: QQ=Q^-1, S=max(abs(U),[],1))→ I,J →SpRight`
- **算法步骤 / Algorithm**:
  1. 未给 `tol` 时取 `max(max(size(A))*norm(A,1)*eps, 100*eps)`（L55–L57）。
  2. 由 `opt` 置 `calc_left`/`calc_right`（L59–L69）。
  3. `[L,U,Q] = luq(A,0,tol)`（不带列置换）（L71）。
  4. **左空间**：`LL = L^-1`；按行最大值 `S=max(abs(U),[],2)` 与 `tol` 切分出行指标 `I`（左值域）与 `J`（左零空间，满足 `Q(J,:)A=0`），返回 `SpLeft={LL,I,J,L}`（L73–L89）。
  5. **右空间**：`QQ = Q^-1`；按列最大值 `S=max(abs(U),[],1)` 切分，返回 `SpRight={QQ,I,J,Q}`（L90–L106）。
  6. 头注**警告**：当矩阵零/非零奇异值分离不佳时，右零空间与值域可能**很不精确**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 A,opt,tol?"] --> B{"给了 tol ?"}
    B -- 否 --> B1["tol = max(size)*norm(A,1)*eps"]
    B -- 是 --> C
    B1 --> C["switch opt → calc_left,calc_right"]
    C --> D["luq(A,0,tol) → L,U,Q"]
    D --> E{"calc_left ?"}
    E -- 是 --> E1["LL=L^-1; 按行切分 I,J → SpLeft"]
    E -- 否 --> F
    E1 --> F{"calc_right ?"}
    F -- 是 --> F1["QQ=Q^-1; 按列切分 I,J → SpRight"]
    F -- 否 --> Z["输出 SpLeft,SpRight"]
    F1 --> Z
  ```

- **依赖 / Dependencies**: `luq`。
- **See also**: [external/spnull.m](file:///c:/VSWork/gptoolbox-1/external/spnull.m#L1-L20)、[external/luq.m](file:///c:/VSWork/gptoolbox-1/external/luq.m#L1-L147)。

### `combn` — 组合枚举

- **文件 / File**: [external/combn.m](file:///c:/VSWork/gptoolbox-1/external/combn.m#L1-L159)
- **署名 / Attribution**: 作者 **Jos van der Geest**（`jos@jasen.nl`，`version 4.3 (apr 2013)`）；许可：**BSD 式**条款（`Copyright (c) 2009, Jos van der Geest`）。
- **签名 / Signature**: `[M,IND] = combn(V,N)`
- **功能 / Purpose**: `EN: Enumerates all combinations of N elements (with repetition) drawn from V — the Cartesian power V^N, size numel(V)^N by N; V may be numeric, cell or char.` / 中文：枚举从 `V` 中（可重复）取 `N` 个元素的所有组合（即笛卡尔幂 `V^N`），`V` 可为数值/元胞/字符串。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `V` | 向量/数组/元胞/字符串 | 候选元素集合 / candidate elements |
  | `N` | 正整数标量 | 每次取的个数 / number of elements per combination |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `M` | `numel(V)^N by N` | 全部组合 / all combinations |
  | `IND` | `numel(V)^N by N` | 索引矩阵，满足 `M = V(IND)` / indices |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | 第二输出 `IND` | 请求与否 | 不请求 | 请求时额外返回索引 / optional second output |

- **变量流动 / Variable flow**: `V,N →(1 特例:空或N=0或N=1)→ M,IND →(2 local_allcomb(V,N) 或 local_allcomb(1:numel(V),N))→ Y/IND →(3 [Y{N:-1:1}]=ndgrid(X); cat(N+1,Y{:}); reshape([],N))→ M →(4 有 IND: M=V(IND))→ M,IND`
- **算法步骤 / Algorithm**:
  1. `error(nargchk(2,2,nargin))` 校验参数个数（L55）。
  2. `V` 为空或 `N==0` → 返回空 `M=[]`,`IND=[]`（L57–L59）。
  3. `N` 必须为正整数标量，否则报错 `combn:negativeN`（L60–L61）。
  4. `N==1` 直接返回列向量 `M=V(:)` 与索引（L62–L65）。
  5. 否则：仅要 `M` 时 `M=local_allcomb(V,N)`；同时要索引时对 `1:numel(V)` 求组合得 `IND` 再 `M=V(IND)`（L66–L74）。
  6. `local_allcomb`：`[Y{N:-1:1}]=ndgrid(X)` 生成 N 维网格，`cat(N+1,Y{:})` 拼接后 `reshape([],N)` 展成二维矩阵（L79–L89）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 V,N"] --> B{"V 空 或 N==0 ?"}
    B -- 是 --> B1["M=[]; IND=[]"]
    B -- 否 --> C{"N 正整数标量 ?"}
    C -- 否 --> C1["error combn:negativeN"]
    C -- 是 --> D{"N==1 ?"}
    D -- 是 --> D1["M=V(:); IND=(1:numel(V)).'"]
    D -- 否 --> E{"nargout<2 ?"}
    E -- 是 --> E1["M = local_allcomb(V,N)"]
    E -- 否 --> E2["IND=local_allcomb(1:numel(V),N); M=V(IND)"]
    E1 --> Z["输出 M(,IND)"]
    E2 --> Z
  ```

- **依赖 / Dependencies**: 无同库依赖；MATLAB 内置 `ndgrid`、`cat`、`reshape`（注意：`nargchk` 已被新版 MATLAB 弃用/移除，可能导致运行错误）。
- **See also**: MATLAB 的 `perms`、`nchoosek`、File Exchange 的 `combn_sub`、`allcomb`；[matrix/nmultichoosek.m](file:///c:/VSWork/gptoolbox-1/matrix/nmultichoosek.m#L1)。
- **重叠 / Overlap**: [matrix/nmultichoosek.m](file:///c:/VSWork/gptoolbox-1/matrix/nmultichoosek.m#L1) 是本库的"可重复组合"实现；注意 `combn` 实为**带重复的笛卡尔幂**（与 `nchoosek` 的"不重复组合"语义不同）。

### `trandn` — 截断正态分布采样

- **文件 / File**: [external/trandn.m](file:///c:/VSWork/gptoolbox-1/external/trandn.m#L1-L126)
- **署名 / Attribution**: 作者 **Zdravko I. Botev**（`Copyright (c) 2016, Zdravko I. Botev`，UNSW）；许可：**BSD 式**条款（禁止以 UNSW 名义背书）。注：任务上下文曾标注为 "Nicolas Chopin"，但**文件头与参考文献均为 Botev (2016)**，此处以文件头为准。
- **签名 / Signature**: `x = trandn(l, u)`
- **功能 / Purpose**: `EN: Efficient generator of a vector of samples from the standard normal truncated to [l,u]; infinite limits accepted.` / 中文：对标准正态分布在区间 `[l,u]` 上截断，高效批量生成样本向量（允许 `l`/`u` 为 ±Inf）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `l` | `k by 1` | 下截断限（可为 `-Inf`）/ lower limits |
  | `u` | `k by 1` | 上截断限（可为 `+Inf`）/ upper limits |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `x` | `k by 1` | 截断正态样本 / truncated-normal samples |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | 无 | — | — | 无 Name/Value 选项；仅两个位置参数 / positional only |

- **变量流动 / Variable flow**: `l,u →(1 l=l(:),u=u(:) + 长度校验)→ l,u →(2 阈值 a=0.66 分三类)→ I,J →(3 case1 ntail(l,u)；case2 -ntail(-u,-l)；case3 tn(l,u))→ x`
- **算法步骤 / Algorithm**:
  1. 列向量化并校验 `length(l)==length(u)`，否则报错（L17–L20）。
  2. 取阈值 `a=0.66` 用于方法切换（L22）。
  3. 情况 1：`l>a`（远离原点的右尾）→ `ntail`，用 **Rayleigh 分布接受-拒绝**（Marsaglia 1964 式）（L26–L29、L42–L62）。
  4. 情况 2：`u<-a` → 对称变换后同样走 `ntail`（`x=-ntail(-u,-l)`）（L31–L34）。
  5. 情况 3：其余（区间靠近原点）→ `tn`：跨度 `abs(u-l)>tol(=2)` 时用 `randn` 接受-拒绝 `trnd`，否则用 `erfcinv` **逆变换采样**（L35–L39、L64–L83、L85–L99）。
  6. 参考文献：Botev, Z. I. (2016) "The normal law under linear restrictions: simulation and estimation via minimax tilting", JRSS-B。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 l,u"] --> B["l=l(:); u=u(:)"]
    B --> C{"length(l)==length(u) ?"}
    C -- 否 --> C1["error 抛出"]
    C -- 是 --> D{"l > a (=0.66) ?"}
    D -- 是 --> D1["x = ntail(l,u) (Rayleigh 接受-拒绝)"]
    D -- 否 --> E{"u < -a ?"}
    E -- 是 --> E1["x = -ntail(-u,-l)"]
    E -- 否 --> F{"abs(u-l) > 2 ?"}
    F -- 是 --> F1["trnd: randn 接受-拒绝"]
    F -- 否 --> F2["erfcinv 逆变换"]
    D1 --> Z["输出 x"]
    E1 --> Z
    F1 --> Z
    F2 --> Z
  ```

- **依赖 / Dependencies**: 无同库依赖；MATLAB 内置 `rand`、`randn`、`erfc`、`erfcinv`、`reallog`、`expm1`。
- **See also**: MATLAB Statistics Toolbox 的 `truncated`/`normrnd`（无直接替代）；[matrix/sample_discrete.m](file:///c:/VSWork/gptoolbox-1/matrix/sample_discrete.m#L1)（离散采样，用途相邻）。

### `ssim` — 带自动降采样的结构相似度

- **文件 / File**: [external/ssim.m](file:///c:/VSWork/gptoolbox-1/external/ssim.m#L1-L231)
- **署名 / Attribution**: 原始算法 **Zhou Wang**（`Copyright (c) 2009 Zhou Wang`，"SSIM Index with automatic downsampling, Version 1.0"）；本副本 **由 Adam Turcotte 与 Nicolas Robidoux（Laurentian University）重写**（`Last Modified: 2011-01-22`）；许可：**仅限教育/研究、非商业**（需保留版权与作者署名）。
- **签名 / Signature**: `[mssim, ssim_map] = ssim(img1, img2, K, window, L)`
- **功能 / Purpose**: `EN: Structural SIMilarity (SSIM) index between two images with automatic downsampling, per Wang et al. 2004.` / 中文：计算两幅图像之间的结构相似度 SSIM（Wang et al. 2004），并做**自动降采样**。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `img1` | `M by N` | 第一幅图像 / first image |
  | `img2` | `M by N` | 第二幅图像（须同尺寸）/ second image |
  | `K` | `1 by 2` | SSIM 稳定常数 / SSIM constants |
  | `window` | `H by W` | 局部统计窗 / local statistics window |
  | `L` | `scalar` | 图像动态范围 / dynamic range |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `mssim` | `scalar` | 平均 SSIM 值（`img1==img2` 时为 1）/ mean SSIM |
  | `ssim_map` | 尺寸小于输入 | SSIM 索引图 / SSIM index map |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `K` | `1 by 2` 非负 | `[0.01 0.03]` | SSIM 稳定常数 / constants |
  | `window` | `H by W` | `fspecial('gaussian',11,1.5)` | 局部统计窗 / local window |
  | `L` | `scalar>0` | `255` | 动态范围 / dynamic range |

- **变量流动 / Variable flow**: `img1,img2,K,window,L →(1 默认/校验)→ →(2 自动降采样 f=max(1,round(min(M,N)/256)))→ img1,img2 →(3 C1,C2 + window 归一化)→ →(4 filter2 求 gx,gy,gxy,g(x^2+y^2))→ →(5 组合 num/den)→ ssim_map →(6 mean2)→ mssim`
- **算法步骤 / Algorithm**:
  1. `nargin`/尺寸校验，非法输入返回 `-Inf`；按 `nargin`（2/3/4/5）分支设默认 `window=fspecial('gaussian',11,1.5)`、`K=[0.01 0.03]`、`L=255`（L92–L178）。
  2. `double` 化；**自动降采样**：`f=max(1,round(min(M,N)/256))`，先均值低通 `imfilter(...,'symmetric')` 再 `1:f:end` 抽样（L181–L196）。
  3. 稳定常数 `C1=(K(1)*L)^2`、`C2=(K(2)*L)^2`；`window` 归一化（L198–L200）。
  4. 用 `filter2(window,...,'valid')`（**4 次模糊**，比原版少 1 次）求得 `gx,gy,g(xy),g(x^2+y^2)`，逐像素组合出 `num` 与 `den`（L201–L217）。
  5. `C1,C2>0` 时 `ssim_map=num./den`；否则对 `w1.*w3==0` 退化情形单独处理，最后 `mssim=mean2(ssim_map)`（L218–L229）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 img1,img2,K,window,L"] --> B{"参数/尺寸合法 ?"}
    B -- 否 --> B1["mssim=-Inf; ssim_map=-Inf; return"]
    B -- 是 --> C["按 nargin 设默认 window,K,L"]
    C --> D["自动降采样 f=max(1,round(min(M,N)/256))"]
    D --> E["C1,C2; window 归一化"]
    E --> F["filter2 求 gx,gy,gxy,g(x^2+y^2)"]
    F --> G{"C1>0 且 C2>0 ?"}
    G -- 是 --> G1["ssim_map = num./den"]
    G -- 否 --> G2["按 w1.*w3 退化情形处理"]
    G1 --> Z["mssim = mean2(ssim_map)"]
    G2 --> Z
  ```

- **依赖 / Dependencies**: MATLAB **Image Processing Toolbox**（`fspecial`、`imfilter`、`filter2`、`mean2`）。
- **See also**: [external/ssim_index.m](file:///c:/VSWork/gptoolbox-1/external/ssim_index.m#L1-L214)（无降采样版）。
- **重叠 / Overlap**: 与 [external/ssim_index.m](file:///c:/VSWork/gptoolbox-1/external/ssim_index.m#L1-L214) 是**同一 SSIM 的近似重复实现**（共享 Turcotte/Robidoux 重写的核心）；`ssim` 额外含自动降采样。**注意：两者互不调用——`ssim` 并不 wrap `ssim_index`**（头注亦声明"different from ssim_index.m"）。

### `ssim_index` — 无自动降采样的结构相似度

- **文件 / File**: [external/ssim_index.m](file:///c:/VSWork/gptoolbox-1/external/ssim_index.m#L1-L214)
- **署名 / Attribution**: 原始算法 **Zhou Wang**（`Copyright (c) 2003 Zhou Wang`，"SSIM Index, Version 1.0"，HHMI/NYU）；本副本 **由 Adam Turcotte 与 Nicolas Robidoux 重写**（`Last Modified: 2011-01-22`）；许可：**仅限教育/研究、非商业**。
- **签名 / Signature**: `[mssim, ssim_map] = ssim_index(img1, img2, K, window, L)`
- **功能 / Purpose**: `EN: Structural SIMilarity (SSIM) index between two images, without automatic downsampling (per Wang et al. 2004).` / 中文：计算两幅图像的 SSIM，**不做自动降采样**（`ssim_map` 尺寸为 `size(img1)-size(window)+1`）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `img1` | `M by N` | 第一幅图像 / first image |
  | `img2` | `M by N` | 第二幅图像（须同尺寸）/ second image |
  | `K` | `1 by 2` | SSIM 稳定常数 / SSIM constants |
  | `window` | `H by W` | 局部统计窗 / local statistics window |
  | `L` | `scalar` | 图像动态范围 / dynamic range |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `mssim` | `scalar` | 平均 SSIM 值 / mean SSIM |
  | `ssim_map` | `M-H+1 by N-W+1` | SSIM 索引图 / SSIM index map |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `K` | `1 by 2` 非负 | `[0.01 0.03]` | SSIM 稳定常数 / constants |
  | `window` | `H by W` | `fspecial('gaussian',11,1.5)` | 局部统计窗 / local window |
  | `L` | `scalar>0` | `255` | 动态范围 / dynamic range |

- **变量流动 / Variable flow**: `img1,img2,K,window,L →(1 默认/校验)→ →(2 C1,C2 + window 归一化)→ →(3 filter2 求 gx,gy,gxy,g(x^2+y^2))→ →(4 组合 num/den)→ ssim_map →(5 mean2)→ mssim`
- **算法步骤 / Algorithm**:
  1. `nargin`/尺寸校验，非法输入（含 `nargin<2||nargin>5`、尺寸不等、图像小于窗）返回 `-Inf`；按 `nargin` 分支设默认 `window=fspecial('gaussian',11,1.5)`、`K=[0.01 0.03]`、`L=255`（L90–L176）。
  2. 计算 `C1=(K(1)*L)^2`、`C2=(K(2)*L)^2`；`window` 归一化；`img1,img2` 转 `double`（L178–L182）。
  3. 用 `filter2(window,...,'valid')`（4 次模糊）求得 `gx,gy,g(xy),g(x^2+y^2)`，逐像素组合出 `num` 与 `den`（L184–L200）。
  4. `C1,C2>0` 时 `ssim_map=num./den`；否则按 `w1.*w3==0` 退化情形处理；最后 `mssim=mean2(ssim_map)`（L196–L212）。
  5. 与 `ssim` 的唯一实质差别：**本函数无第 185–196 行的自动降采样步骤**，即不做低通 + 抽样。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 img1,img2,K,window,L"] --> B{"参数/尺寸合法 ?"}
    B -- 否 --> B1["mssim=-Inf; ssim_map=-Inf; return"]
    B -- 是 --> C["按 nargin 设默认 window,K,L"]
    C --> D["C1,C2; window 归一化; double 化"]
    D --> E["filter2 求 gx,gy,gxy,g(x^2+y^2)"]
    E --> F{"C1>0 且 C2>0 ?"}
    F -- 是 --> F1["ssim_map = num./den"]
    F -- 否 --> F2["按 w1.*w3 退化情形处理"]
    F1 --> Z["mssim = mean2(ssim_map)"]
    F2 --> Z
  ```

- **依赖 / Dependencies**: MATLAB **Image Processing Toolbox**（`fspecial`、`filter2`、`mean2`）。
- **See also**: [external/ssim.m](file:///c:/VSWork/gptoolbox-1/external/ssim.m#L1-L231)（带自动降采样版）。
- **重叠 / Overlap**: 与 [external/ssim.m](file:///c:/VSWork/gptoolbox-1/external/ssim.m#L1-L231) 互为**近重复实现**；本文件即 `ssim` 核心去掉自动降采样后的版本，**不被 `ssim` 调用**（无 wrap 关系）。

---

## 函数清单核对表 / Coverage Checklist

> 10 个文件逐一对应 10 个条目；锚点均可点击跳转至源码（行号取文件实际范围）。

| # | 函数 / File | 锚点 / Anchor | 来源 / 署名 | 一句话用途 / One-line purpose |
|---|---|---|---|---|
| 1 | `affine_fit` | [external/affine_fit.m](file:///c:/VSWork/gptoolbox-1/external/affine_fit.m#L1-L60) | Adrien Leygue (2013) | 点云最小二乘拟合仿射平面 |
| 2 | `inv2` | [external/inv2.m](file:///c:/VSWork/gptoolbox-1/external/inv2.m#L1-L46) | Bruno Luong (2010) | 批量 2×2 解析求逆/求解 |
| 3 | `inv3` | [external/inv3.m](file:///c:/VSWork/gptoolbox-1/external/inv3.m#L1-L59) | Bruno Luong (2010) | 批量 3×3 解析求逆/求解 |
| 4 | `luq` | [external/luq.m](file:///c:/VSWork/gptoolbox-1/external/luq.m#L1-L147) | Pawel Kowal (2006) | 带列置换的 LU 分解 |
| 5 | `spnull` | [external/spnull.m](file:///c:/VSWork/gptoolbox-1/external/spnull.m#L1-L20) | Jan Schellenberger (2009) | 稀疏矩阵快速零空间基 |
| 6 | `spspaces` | [external/spspaces.m](file:///c:/VSWork/gptoolbox-1/external/spspaces.m#L1-L106) | Pawel Kowal (2006) | 稀疏矩阵四基本子空间 |
| 7 | `combn` | [external/combn.m](file:///c:/VSWork/gptoolbox-1/external/combn.m#L1-L159) | Jos van der Geest (2013) | N 元（可重复）组合枚举 |
| 8 | `trandn` | [external/trandn.m](file:///c:/VSWork/gptoolbox-1/external/trandn.m#L1-L126) | Zdravko I. Botev (2016) | 截断正态分布批量采样 |
| 9 | `ssim` | [external/ssim.m](file:///c:/VSWork/gptoolbox-1/external/ssim.m#L1-L231) | Zhou Wang (2009) + Turcotte/Robidoux | 带自动降采样的 SSIM |
| 10 | `ssim_index` | [external/ssim_index.m](file:///c:/VSWork/gptoolbox-1/external/ssim_index.m#L1-L214) | Zhou Wang (2003) + Turcotte/Robidoux | 无自动降采样的 SSIM |

---

## 重叠与外部依赖汇总 / Overlap & External-Dependency Summary

| 类型 / Type | 函数 | 说明 |
|---|---|---|
| 依赖外部工具箱 / External toolbox | `inv2`, `inv3` | 提供 `B` 时依赖 James Tursa 的 `mtimesx`（仓库内**不存在**，需另行获取） |
| 依赖外部工具箱 / External toolbox | `ssim`, `ssim_index` | 依赖 MATLAB **Image Processing Toolbox**（`fspecial`/`imfilter`/`filter2`/`mean2`） |
| 许可限制 / Licence restriction | `luq`, `spspaces` | LREM_SOLVE toolbox，**仅非商业学术**使用 / noncommercial academic use only |
| `external/` 内重复 / Duplicate in external | `ssim` vs `ssim_index` | 同一 SSIM 的两份近重复实现；`ssim` = 核心 + 自动降采样；**互不调用** |
| 与本库模块重叠 / Overlaps other module | `spnull`, `spspaces`, `luq` | 与 [matrix/pagenull.m](file:///c:/VSWork/gptoolbox-1/matrix/pagenull.m#L1-L16)、[matrix/affine_null_space.m](file:///c:/VSWork/gptoolbox-1/matrix/affine_null_space.m#L1)、[matrix/lexmin.m](file:///c:/VSWork/gptoolbox-1/matrix/lexmin.m#L1) 的零空间/分解功能重叠（自有实现基于 SVD 更精确，`external` 版基于 `luq` 更稀疏/更快） |
| 与本库模块重叠 / Overlaps other module | `combn` | 与 [matrix/nmultichoosek.m](file:///c:/VSWork/gptoolbox-1/matrix/nmultichoosek.m#L1) 语义相近（均为带重复组合），但 `combn` 实际返回笛卡尔幂 |
| 内部依赖链 / Dependency chain | `spnull → spspaces → luq` | 三者构成唯一一条本章内部依赖链 |

---

## 交叉引用 / Cross references

- 本目录其它第三方文档：[90.1 色图与颜色](01-colormaps-and-color.md)、[90.3 MATLAB 环境与绘图](03-matlab-environment-and-plotting.md)、[90.4 exactgeodesic 工具箱](04-exactgeodesic-toolbox.md)、[90-external/README.md](README.md)
- gptoolbox 自有矩阵工具：[matrix/pagenull.m](file:///c:/VSWork/gptoolbox-1/matrix/pagenull.m#L1-L16)、[matrix/affine_null_space.m](file:///c:/VSWork/gptoolbox-1/matrix/affine_null_space.m#L1)、[matrix/diag_blk_inv.m](file:///c:/VSWork/gptoolbox-1/matrix/diag_blk_inv.m#L1)
- 文档写作规范：[appendices/03-doc-template.md](../appendices/03-doc-template.md)
