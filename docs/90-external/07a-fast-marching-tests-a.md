# 90.7a Toolbox Fast Marching — 测试与示例 A（第三方） / Fast Marching Tests and Demos A (Vendored)

> 本章覆盖 `external/toolbox_fast_marching/tests/` 中的 **23 个 demo/测试脚本**（文件 1–23，按文件名有序）。
> 它们是 Gabriel Peyre 的 **Toolbox Fast Marching**（BSD，约 2004–2007）被 vendored 进 gptoolbox 时随附的示例脚本，
> 用于**构造合成数据**并驱动 [90.5 求解器核心](05-fast-marching-solvers.md) 与 [90.5b 度量/辅助函数](05b-fast-marching-compute-helpers.md)
> 中的函数，最后用 `imageplot` / `plot_*` 可视化或 `saveas` 出图。
> The 23 scripts below are **scripts, not functions**: they build synthetic data, drive the vendored solvers, then
> visualise/compare results. 它们**不是单元测试**（无 `assert`），多数只把结果画出来或写盘。
>
> **运行前提 / How to run**：先把 `external/toolbox_fast_marching/` 加入 MATLAB 路径，再到该目录下执行
> `compile_mex`（见 [compile_mex.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compile_mex.m#L1-L40)）以编译
> `perform_front_propagation_2d` / `_3d` / `_circular` / `_mesh`、`skeleton`、`eucdist2` 等 mex 二进制；
> 然后在 `tests/` 中逐个运行脚本（需交互取点的脚本会调用 `ginput`）。仓库仅附带部分数据：`data/` 下有
> `brain.png`、`cavern.png`、`mountain.png`、`road2.png`、`stephanodiscusniagarae.jpg`、`mm.png`、`giraffe.gif` 等，
> **`disk.png`、`diatom.png`、`heart1.mat`、`brain1-crop-256.mat`、`toolbox_graph_data/off/` 网格等大量资源缺失**，
> 另有若干脚本调用**仓库中不存在的函数**（详见各行「依赖」与末尾核对表）。
>
> - 相关章节 / See also：[90.5 求解器核心](05-fast-marching-solvers.md)、[90.5b 度量与辅助](05b-fast-marching-compute-helpers.md)、
>   [appendices/03-doc-template.md](../appendices/03-doc-template.md)
> - 命名约定 / Notation：`W` 速度倒数权重；`D` 距离场（far=`Inf`）；`S` 状态；`Q` 最近种子索引；`L` = `constraint_map`；`M` 显示用灰度图。

---

## 本章概览 / Chapter Overview

| 脚本 / Script | 驱动的求解器 / Solver driven | 一句话功能 / One-line purpose |
|---|---|---|
| `test_active_contour` | `perform_active_contour` | 在 brain 图上以 snake 边缘能量驱动水平集活动轮廓 |
| `test_anisotropic_feth` | `perform_fast_marching`（各向异性）+ `fm2dAniso` | 对比 Prados(FM) 与 Feth 两套各向异性前向传播 |
| `test_anisotropic_fm_old` | `perform_fast_marching`（各向异性，旧用法） | 旧版各向异性 FM：多起点 Voronoi、渐进传播、测地线 |
| `test_anisotropic_fm` | `perform_fast_marching`（各向异性） | 2D/3D 各向异性张量场 FM 距离可视化 |
| `test_anisotropic` | `perform_fast_marching`（各向异性） | 各向异性传播的**残缺片段**（变量未定义，不可运行） |
| `test_bending_invariants` | `compute_bending_invariant` | 网格弯曲不变量的计算与嵌入显示 |
| `test_bug` | `perform_fast_marching`（3D） | 3D FM 在随机权重立方体上的计时/复现片段 |
| `test_circular_fast_marching_2d` | `perform_circular_fast_marching_2d` | 硅藻图上的圆环传播 + 路径提取 |
| `test_circular_prior` | `perform_circular_fast_marching_2d`（带 H） | 加圆环形状先验启发 `H` 的圆环传播 |
| `test_circular` | `perform_circular_fast_marching_2d` | 椭圆/图像上的圆环传播（含缩放与度量加权） |
| `test_constrained_map` | `perform_fast_marching`（`constraint_map`） | 用 `L` 把传播约束到圆盘区域并画 Voronoi/距离 |
| `test_distance_approximation` | `perform_farthest_point_sampling` + `compute_distance_landmark` | 采样策略对地标距离近似误差的影响 |
| `test_distance_compression` | `compute_heuristic_landmark` + `perform_fast_marching` | 用地标启发估计距离图并分析残差 |
| `test_eccentricity` | `compute_eccentricity_transform` | 2D 形状的偏心率变换（欧氏 vs 测地） |
| `test_eucldist` | `eucdist2` | 二值图的欧氏距离变换最小冒烟测试 |
| `test_farthest_sampling_2d` | `perform_farthest_point_sampling` | 2D 势场上的最远点采样 + Voronoi 三角剖分 |
| `test_farthest_sampling_3d` | `perform_farthest_point_sampling`（3D） | 体数据最远点采样 + 三角化/切片显示 |
| `test_farthest_sampling_mesh` | `perform_farthest_point_sampling_mesh` | 三角网格上的最远点采样与重网格化 |
| `test_farthest_sampling_shape` | `perform_farthest_point_sampling`（形状约束） | 形状内部的自适应最远点采样与网格生成 |
| `test_fast_marching_2d` | `perform_fast_marching`（2D）+ `compute_geodesic` | 2D 势场 FM + 多条测地线提取与显示 |
| `test_fast_marching_3d` | `perform_fast_marching`（3D）+ `compute_geodesic` | 3D 高斯势场 FM + 单条测地线显示 |
| `test_fmstar_2d` | `perform_fmstar_2d` | FM\* 2D：多权重的路径/圆周传播对比 |
| `test_fmstar_3d` | `perform_fmstar_3d` | FM\* 3D：constant/gaussian/heart 三型势场对比 |

---

## 逐脚本详解 / Per-script reference

### `test_active_contour` — 活动轮廓（水平集）演示
- **文件 / File**: [tests/test_active_contour.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_active_contour.m#L1-L89)（89 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。署名 **Gabriel Peyre / Toolbox Fast Marching (BSD)** —— 证据 [L3](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_active_contour.m#L3) `%   Copyright (c) 2007 Gabriel Peyre`。
- **被驱动的函数 / Functions exercised**: `perform_active_contour`、`divgrad`、`compute_levelset_shape`、`load_image`、`perform_blurring`、`rescale`
- **功能 / Purpose**: EN: Drive the level-set active-contour solver with a `snake` (edge-based) energy on a blurred brain image. / 中文：在模糊后的 brain 图像上用 snake 边缘能量驱动水平集活动轮廓。
- **工作区变量 / Workspace vars**: `options`（`bound/solver/motion` 相关键、`Tmax=2000`、`redistance_freq`、`dt`、`M`、`E`、`width`）、`n`、`d`、`D`（`n^2 × n^2`）、`a`、`motion`、`name`、`namec`、`D0`、`D1`、`M`、`sigma`、`G`、`eta`、`E`
- **变量流动 / Variable flow**: `n=32 →(1 逐点 divgrad∘divgrad 组装各向异性扩散算子 D)→ D →(2 n=150, load_image('brain') 归一→M)→(3 compute_levelset_shape 造初值 D0/D1 + 装配 options)→(4 perform_active_contour(D0,'snake',options))→ D`
- **算法步骤 / Algorithm**:
  1. `options.bound='per'`、`n=32`；以 `d=rand(n)` 为系数，循环 `n^2` 次用 `a=divgrad(divgrad(a)./d)` 组装隐式算子 `D`（`n^2 × n^2`）。
  2. 改 `n=150`；`M = rescale(sum(load_image('brain'),3))` 取灰度图；`options.solver='cg'`。
  3. 最终 `motion='snake'`（L27 最后一次赋值），取 `namec='square'`、`options.width=0.95*n`；`compute_levelset_shape` 造初始水平集 `D0` 与圆 `D1`。
  4. snake 分支：`G=divgrad(perform_blurring(M,4))`、`E=rescale(1./(0.01+sum(G.^2,3)),0.7,1)` 作边缘能量，设 `Tmax=2000`、`dt=1`。
  5. `D = perform_active_contour(D0,'snake',options)` 演化；**无断言、无绘图**，结果仅留在工作区。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=32 组装扩散算子 D"] --> B["n=150 载入 brain 归一为 M"]
    B --> C["motion=snake: compute_levelset_shape 造 D0"]
    C --> D["装配 snake 边缘能量 E 与 options"]
    D --> E["perform_active_contour(D0, snake, options)"]
    E --> Z["输出 D"]
  ```

- **依赖 / Dependencies**: `perform_active_contour`、`divgrad`、`compute_levelset_shape`、`load_image`（需 `data/brain.png`，**已附带**）、`perform_blurring`、`rescale`。内层 `n^2=1024` 次双重 `divgrad` 較耗时。
- **See also**: `perform_active_contour`、`perform_redistancing`

### `test_anisotropic_feth` — Prados(FM) 与 Feth 各向异性前向传播对比
- **文件 / File**: [tests/test_anisotropic_feth.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_anisotropic_feth.m#L1-L60)（60 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**（标题仅 `% test for Fethalah code and Prados code`）；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `perform_fast_marching`（各向异性）、`fm2dAniso`（**仓库缺失**）、`plot_tensor_field`（**缺失**）、`perform_tensor_recomp`、`perform_vf_normalization`、`perform_blurring`、`perform_histogram_equalization`、`imageplot`
- **功能 / Purpose**: EN: Compare Peyre's FMM (`perform_fast_marching`) against the vendored Feth anisotropic code (`fm2dAniso`) over a range of anisotropy ratios. / 中文：在多个各向异性比下对比 Peyre 的 FM 与 Feth 各向异性代码。
- **工作区变量 / Workspace vars**: `n`、`rep`、`U`、`sigma`、`options`、`aniso_list`、`p`、`x`、`Y`、`X`、`start_points`、`s`、`aniso`、`V`、`T`、`Ti`、`D,S,Q`、`hx,hy`、`D1,dUx,dUy,Vor,L`
- **变量流动 / Variable flow**: `n=500 →(1 随机张量场 U 迭代 blur 后归一)→ U →(2 循环 aniso_list：perform_tensor_recomp 造 T 与逆张量 Ti)→(3 perform_fast_marching(T,...)→D,S,Q 与 fm2dAniso(...)→D1,Vor)→(4 imageplot/plot_tensor_field 对比并 saveas 到 results/anisotropic-feth/)`
- **算法步骤 / Algorithm**:
  1. `n=500`；`randn('seed',12345)` 生成 `U=randn(n,n,2)`，`perform_blurring` 迭代 10 次并 `perform_vf_normalization` 得光滑方向场。
  2. `aniso_list=[1 .5 .2 .1 .05 .01 .001]`，在网格上取 `p=5` 的规则 `start_points`。
  3. 循环：`perform_tensor_recomp` 造张量 `T`（比率 `aniso`）与其逆 `Ti`；`options.use_feth_code=0`。
  4. `[D,S,Q]=perform_fast_marching(T,start_points,options)` 并 `tic/toc` 计时；再用 `[D1,dUx,dUy,Vor,L]=fm2dAniso([hx;hy],Ti,start_points)` 对比。
  5. `imageplot(s(Q))`/`imageplot(s(Vor+1))` 对比 Voronoi，`plot_tensor_field` 画距离；`saveas` 全部出图。**无定量断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=500 随机方向场 U (blur+归一)"] --> B{"遍历 aniso_list"}
    B --> C["perform_tensor_recomp 造 T 与逆 Ti"]
    C --> D["perform_fast_marching(T) 得 D,S,Q"]
    C --> E["fm2dAniso(Ti) 得 D1,Vor (缺失)"]
    D --> F["imageplot / plot_tensor_field 出图"]
    E --> F
    F --> B
  ```

- **依赖 / Dependencies**: `fm2dAniso`（**未定义**：源码仅存 `mex/anisotropic-fm-feth/fm2dAniso.cpp` 与 Mac 版 `fm2dAniso.mexmaci`，且 [compile_mex.m#L8-L9](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compile_mex.m#L8-L9) 已注释"这些其实不存在"）、`plot_tensor_field`（**未定义**）。⚠️ **在本仓库必然报错，不可运行**。
- **See also**: `test_anisotropic_fm`、`test_anisotropic_fm_old`、`perform_tensor_recomp`

### `test_anisotropic_fm_old` — 旧版各向异性 FM 用法
- **文件 / File**: [tests/test_anisotropic_fm_old.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_anisotropic_fm_old.m#L1-L220)（220 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `perform_fast_marching`（各向异性）、`compute_geodesic`、`plot_tensor_field`（**缺失**）、`apply_colormap`（**缺失**）、`perform_tensor_recomp`、`perform_tensor_decomp_3d`（**缺失**，仅 3D 分支）、`perform_histogram_equalization`、`perform_lic`（可选，`exist` 守卫）、`imageplot`、`imwrite`
- **功能 / Purpose**: EN: Legacy anisotropic-FM demo — multi-seed Voronoi, progressive propagation with `dmax`, and 50 geodesics on the boundary. / 中文：旧版各向异性 FM 演示：多起点 Voronoi、用 `dmax` 的渐进传播、边界 50 条测地线。
- **工作区变量 / Workspace vars**: `name`（默认 `varying-2d`）、`rep`、`n=200`、`s`、`U`、`sigma`、`options`、`aniso_list`、`aniso`、`start_points`、`end_points`、`x,y,t,npaths`、`ms,lw`、`D,S,Q`、`D1`、`paths`、`p`、`Y,X`、`dmax_list`、`M0`、`M`、`w`
- **变量流动 / Variable flow**: `n=200 →(1 U 随机 blur 归一方向场, 可选 perform_lic 造纹理 M)→ U,M →(2 aniso=.01 perform_tensor_recomp→T)→(3 perform_fast_marching(T,start_points)→D,S,Q; compute_geodesic 提路径)→(4 imageplot/plot_tensor_field 与 imwrite 出图)`
- **算法步骤 / Algorithm**:
  1. `name='varying-2d'`、`n=200`：`randn('seed',12345)` 生成方向场并迭代 `perform_blurring`+`perform_vf_normalization`；若存在 `perform_lic` 则合成纹理背景 `M`，否则 `M=U`。
  2. 第 1 段：单一中心起点，在边界布 50 个 `end_points`；对 `aniso_list=[.01 .05 .1 .2 .5 1]` 循环，`perform_fast_marching` 求 `D` 后用 `compute_geodesic` 抽 50 条路径并 `plot` 显示。
  3. 第 2 段：`p=3` 规则多起点，`perform_fast_marching` 求 `D,S,Q`；对 15 个 `dmax` 值（`options.dmax`）做渐进传播，`perform_histogram_equalization` + `apply_colormap('jet')` 合成彩色图 `A`，把 far 点填回背景后 `imwrite` 出帧。
  4. 第 3 段：`nstart=40` 随机多起点，循环 `aniso_list`，用 `apply_colormap(Q/D1)` 写 Voronoi/距离图，`plot_tensor_field(T1,M)` 写张量图。
  5. **无断言**；`apply_colormap` 与 `plot_tensor_field` 缺失 ⇒ 到第 2 段即报错。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=200 方向场 U (可选 LIC 纹理 M)"] --> B["aniso=.01 → perform_tensor_recomp → T"]
    B --> C["perform_fast_marching(T) → D,S,Q"]
    C --> D["compute_geodesic × 50 → paths 并 plot"]
    C --> E["按 dmax 渐进传播 + apply_colormap (缺失)"]
    C --> F["plot_tensor_field 画张量 (缺失)"]
    D --> Z["imwrite 出图"]
    E --> Z
    F --> Z
  ```

- **依赖 / Dependencies**: `plot_tensor_field`、`apply_colormap`（**均未定义**）；3D 分支的 `perform_tensor_decomp_3d`（**未定义**，默认 `varying-2d` 不触发）。⚠️ **默认路径即报错，不可运行**。
- **See also**: `test_anisotropic_fm`、`perform_fast_marching`、`perform_histogram_equalization`

### `test_anisotropic_fm` — 各向异性张量场 FM 距离可视化
- **文件 / File**: [tests/test_anisotropic_fm.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_anisotropic_fm.m#L1-L101)（101 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `perform_fast_marching`（各向异性）、`perform_tensor_recomp`、`perform_tensor_decomp_3d`（**缺失**，仅 3D 分支）、`plot_tensor_field`（**缺失**）、`perform_vf_normalization`、`perform_blurring`、`imageplot`
- **功能 / Purpose**: EN: Visualise anisotropic FMM distance fields for fixed/varying 2D or 3D tensor metrics. / 中文：对固定/变化 2D 或 3D 张量度量可视化各向异性 FM 距离场。
- **工作区变量 / Workspace vars**: `name`（默认 `varying-2d`）、`rep`、`n`、`s`、`u`、`U`、`sigma`、`aniso_list`、`ianiso`、`aniso`、`nstart`、`start_points`、`V`、`T`、`D`、`D1`、`T1`、`options`
- **变量流动 / Variable flow**: `name='varying-2d', n=200 →(1 方向场 U 迭代 blur 归一)→ U;(2 循环 aniso_list：perform_tensor_recomp(U,V,...)→T)→(3 perform_fast_marching(T,start_points)→D)→(4 imageplot(D1)/plot_tensor_field(T1,D1) 出图)`
- **算法步骤 / Algorithm**:
  1. `name='varying-2d'`（L4 生效，L2/L3 被注释）、`n=200`：`U=randn(n,n,2)` 经 10 次 `perform_blurring`+`perform_vf_normalization`。
  2. `aniso_list=[0.01 0.05 .1 .2 .5 1]`；`nstart=8` 随机起点并 `rescale` 到 `[.1,.9]`。
  3. 循环：由 `U` 与正交向量 `V=cat(3,-U(:,:,2),U(:,:,1))` 经 `perform_tensor_recomp(U,V,1,aniso)` 造张量 `T`。
  4. `D = perform_fast_marching(T, start_points)`；取中间切片 `D1`、`T1`。
  5. `imageplot(D1)` 与 `plot_tensor_field(T1,D1,options)` 依次出图并 `saveas` 到 `results/anisotropic-fm/`。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["name=varying-2d, n=200 方向场 U"] --> B{"遍历 aniso_list"}
    B --> C["V=正交(U); perform_tensor_recomp → T"]
    C --> D["perform_fast_marching(T, start_points) → D"]
    D --> E["imageplot(D1) 出图"]
    D --> F["plot_tensor_field(T1,D1) (缺失)"]
    E --> B
    F --> B
  ```

- **依赖 / Dependencies**: `plot_tensor_field`（**未定义**，L94 未注释）⇒ **默认路径报错**；3D 分支的 `perform_tensor_decomp_3d`（**未定义**，默认不触发）。⚠️ **不可完整运行**。
- **See also**: `test_anisotropic_fm_old`、`perform_fast_marching`、`perform_tensor_recomp`

### `test_anisotropic` — 各向异性传播（残缺片段）
- **文件 / File**: [tests/test_anisotropic.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_anisotropic.m#L1-L42)（42 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `perform_fast_marching`（各向异性）、`perform_tensor_recomp`、`perform_vf_normalization`、`perform_blurring`、`plot_tensor_field`（**缺失**）
- **功能 / Purpose**: EN: Intended anisotropic-propagation + tensor display; currently a broken fragment. / 中文：原意是各向异性传播并画张量场，实则残缺片段。
- **工作区变量 / Workspace vars**: `name`、`rep`、`n=200`、`s`、`U`、`sigma`、`it`、`aniso`、`V`、`T`、`D`、`options`（另**引用但从未定义** `start_points`、`T1`、`D1`）
- **变量流动 / Variable flow**: `n=200 →(1 U 随机 blur 归一)→ U →(2 aniso=.05 perform_tensor_recomp→T)→(3 perform_fast_marching(T,start_points) → D，但 start_points 未定义)→(4 plot_tensor_field 出图，T1/D1 未定义)`
- **算法步骤 / Algorithm**:
  1. `name='varying-2d'`、`n=200`：`U=randn(n,n,2)` 经 10 次 `perform_blurring`+`perform_vf_normalization`。
  2. `aniso=.05`；由 `U` 与正交向量 `V` 经 `perform_tensor_recomp(U,V,1,aniso)` 造张量 `T`。
  3. `D = perform_fast_marching(T, start_points)` —— **`start_points` 从未被赋值**（应为 undefined 变量）。
  4. `plot_tensor_field(T1, D1, options)` —— **`T1`/`D1` 也未定义**，且函数本身缺失。
  5. 结论：这是从其它脚本复制后未清理的**残缺片段**，不能运行，也不代表可用示例。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=200 方向场 U"] --> B["aniso=.05 → perform_tensor_recomp → T"]
    B --> C["perform_fast_marching(T, start_points) 但 start_points 未定义"]
    C --> D["plot_tensor_field(T1, D1) 变量与函数均缺失"]
    D --> E["运行即报错"]
  ```

- **依赖 / Dependencies**: `plot_tensor_field`（**未定义**）；`start_points`/`T1`/`D1` 为**未定义变量**。⚠️ **不可运行**。
- **See also**: `test_anisotropic_fm`、`test_farthest_sampling_2d`

### `test_bending_invariants` — 网格弯曲不变量
- **文件 / File**: [tests/test_bending_invariants.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_bending_invariants.m#L1-L50)（50 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `compute_bending_invariant`、`read_mesh`、`plot_mesh`
- **功能 / Purpose**: EN: Compute the bending-invariant embedding of a mesh and display the original vs. the embedding. / 中文：计算网格的弯曲不变量嵌入并对比显示原网格与嵌入网格。
- **工作区变量 / Workspace vars**: `vertex`、`faces`、`options`（`nlandmarks=200`、`name`）、`name`（默认 `bunny`）、`vertex1`、`rep`
- **变量流动 / Variable flow**: `name='bunny' →(1 read_mesh(name)→vertex,faces)→(2 options.nlandmarks=200 → compute_bending_invariant(vertex,faces,options)→vertex1)→(3 plot_mesh 原网格/嵌入 → saveas 到 results/bening-invariants/)`
- **算法步骤 / Algorithm**:
  1. 把 `toolbox/`、`../toolbox_graph_data/`、`../toolbox_graph_data/off/` 加入路径。
  2. `name` 依次赋值多个网格名，最终生效 `bunny`；`[vertex,faces]=read_mesh(name)`（依赖 `toolbox_graph_data/off` 下的 `.off` 文件）。
  3. `options.nlandmarks=200`；`vertex1 = compute_bending_invariant(vertex,faces,options)`。
  4. `plot_mesh(vertex,faces)` 与原图 `plot_mesh(vertex1,faces)` 各 `shading interp; camlight;`，分别 `saveas` 为 `-original.png`/`-invariant.png`。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["name=bunny"] --> B["read_mesh → vertex,faces"]
    B --> C["options.nlandmarks=200 → compute_bending_invariant → vertex1"]
    C --> D["plot_mesh 原网格 → saveas -original.png"]
    C --> E["plot_mesh 嵌入网格 → saveas -invariant.png"]
  ```

- **依赖 / Dependencies**: `read_mesh`（toolbox 内，**已附带**）、`compute_bending_invariant`（**已附带**）、`plot_mesh`（**已附带**）；但**需 `toolbox_graph_data/off/*.off` 网格数据**（仓库未附带）⇒ **缺数据不可运行**。
- **See also**: `compute_bending_invariant`、`read_mesh`

### `test_bug` — 3D FM 计时/复现片段
- **文件 / File**: [tests/test_bug.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_bug.m#L1-L49)（49 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头仅注释** `% addpath(genpath('fm'));`，无版权行；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `perform_fast_marching`（3D）、`rescale`
- **功能 / Purpose**: EN: Timing/bug-repro harness: run 3D FMM on a random-weight cube and print elapsed time. / 中文：计时/复现实验：在随机权重立方体上跑 3D FM 并打印耗时。
- **工作区变量 / Workspace vars**: `m=10`、`S`、`options`、`x`、`X,Y,Z`、`sigma`、`W`、`D`、`RS`
- **变量流动 / Variable flow**: `m=10 →(1 S 立方体块=1; 先造高斯 W 再被覆盖)→(2 W=rand 归一 →(3 perform_fast_marching(W,[2;2;2],options))→ D,RS →(4 fprintf 计时)`
- **算法步骤 / Algorithm**:
  1. `m=10`；`S=zeros(m,m,m); S(2:4,2:4,2:4)=1;`。
  2. 先构造高斯权重 `W=1./(1+exp(-(X²+Y²+Z²)/sigma²))`，随后被 `W=rescale(S,1e-5,1)` 与 `W=rescale(rand(m,m,m),.5,1)` **覆盖**。
  3. `tic; [D,RS]=perform_fast_marching(W,[2;2;2],options); fprintf(1,'m = %d, time = %f\n',m,toc);`。
  4. 紧跟 `return`（L33），下方 `for m=[10 20 50 100]` 的约束图扫描**不会执行**。**无断言**，功能是 3D FMM 的计时。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["m=10; S 立方体块=1"] --> B["W=rand(m,m,m) 归一 (高斯/约束分支被覆盖)"]
    B --> C["tic; perform_fast_marching(W,[2;2;2],options) → D,RS"]
    C --> D["fprintf 打印耗时"]
    D --> E["return (下方多规模循环不执行)"]
  ```

- **依赖 / Dependencies**: 需 3D mex `perform_front_propagation_3d`（由 `compile_mex` 编译；**3D 无纯 MATLAB 回退**）；合成数据无外部文件。⚠️ **未编译 mex 则报错**。
- **See also**: `test_fast_marching_3d`、`perform_fast_marching`

### `test_circular_fast_marching_2d` — 圆环传播 + 路径提取
- **文件 / File**: [tests/test_circular_fast_marching_2d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_circular_fast_marching_2d.m#L1-L57)（57 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。署名 **Gabriel Peyre / Toolbox Fast Marching (BSD)** —— 证据 [L3](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_circular_fast_marching_2d.m#L3) `%   Copyright (c) 2004 Gabriel Peyre`。
- **被驱动的函数 / Functions exercised**: `perform_circular_fast_marching_2d`、`compute_geodesic`、`compute_edge_energy`、`pick_start_end_point`、`plot_fast_marching_2d`、`rescale`
- **功能 / Purpose**: EN: Run circular (angular) FMM on a diatom image and extract/plot the geodesic path. / 中文：在硅藻图像上执行圆环（角向）FM 并提取/绘制测地路径。
- **工作区变量 / Workspace vars**: `name`、`file_name`、`M`、`m`、`W`、`start_points`、`end_points`、`center_point`、`Y,X`、`d`、`options`、`D`、`S`、`path`
- **变量流动 / Variable flow**: `name='stephanodiscusniagarae' →(1 imread 归一→M)→(2 compute_edge_energy(M)→W)→(3 pick_start_end_point→start/end/center)→(4 以到中心的距离 d 缩放 W)→(5 perform_circular_fast_marching_2d(W,start,center,options)→D,S)→(6 compute_geodesic→path →plot_fast_marching_2d)`
- **算法步骤 / Algorithm**:
  1. `name='stephanodiscusniagarae'`；`M=rescale(double(imread('data/stephanodiscusniagarae.jpg')))`。
  2. `W=compute_edge_energy(M)` 求速度倒数；`[start_points,end_points,center_point]=pick_start_end_point(M)`（**交互式 `ginput`**）。
  3. 用 `d=sqrt((X-center)²+(Y-center)²)`（中心置 `1e-9`）乘 `W`，避免零点附近收缩。
  4. `options.nb_iter_max=Inf; options.end_points=end_points;` `[D,S]=perform_circular_fast_marching_2d(W,start_points,center_point,options)`（**需 mex**）。
  5. `options.Tmax=sum(m)*2; path=compute_geodesic(D,end_points,options)`；缩放回原分辨率后 `plot_fast_marching_2d(M,[],path,start_points,end_points)`。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["imread 硅藻图 → rescale → M"] --> B["compute_edge_energy → W"]
    B --> C["pick_start_end_point (ginput 交互)"]
    C --> D["W = W .* 到中心距离 d"]
    D --> E["perform_circular_fast_marching_2d → D,S"]
    E --> F["compute_geodesic → path"]
    F --> G["plot_fast_marching_2d 显示"]
  ```

- **依赖 / Dependencies**: 需 mex `perform_circular_front_propagation_2d`（由 [compile_mex.m#L4](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compile_mex.m#L4) 编译）；`data/stephanodiscusniagarae.jpg`（**已附带**）；`compute_edge_energy`、`pick_start_end_point`、`plot_fast_marching_2d` 均已附带。⚠️ **未编译 mex 或需交互取点**。
- **See also**: `test_circular`、`test_circular_prior`、`perform_circular_fast_marching_2d`

### `test_circular_prior` — 带形状先验启发式的圆环传播
- **文件 / File**: [tests/test_circular_prior.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_circular_prior.m#L1-L80)（80 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。署名 **Gabriel Peyre / Toolbox Fast Marching (BSD)** —— 证据 [L3](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_circular_prior.m#L3) `%   Copyright (c) 2004 Gabriel Peyre`。
- **被驱动的函数 / Functions exercised**: `perform_circular_fast_marching_2d`（第 5 参 `H` 启发式）、`compute_geodesic`、`compute_edge_energy`、`pick_start_end_point`、`plot_fast_marching_2d`、`perform_image_resize`、`rescale`
- **功能 / Purpose**: EN: Circular FMM augmented with a circular shape-prior heuristic H. / 中文：在圆环 FM 中额外传入圆环形状先验启发 `H` 加速/约束。
- **工作区变量 / Workspace vars**: `n=400`、`name`、`file_name`、`M_original`、`M`、`W`、`m`、`start_points`、`end_points`、`center_point`、`Y,X`、`r`、`s`、`I`、`H`、`options`、`D`、`S`、`path`
- **变量流动 / Variable flow**: `n=400, name='stephanodiscusniagarae' →(1 imread→M, compute_edge_energy→W)→(2 perform_image_resize 缩放到 n)→(3 pick_start_end_point→start/end/center)→(4 由 r,s 构造启发式 H)→(5 perform_circular_fast_marching_2d(W,start,center,options,H)→D,S)→(6 compute_geodesic→path→plot)`
- **算法步骤 / Algorithm**:
  1. `n=400`、`name` 最终 `'stephanodiscusniagarae'`；`M=rescale(double(imread('data/'+name+'.png')))` —— **注意**：仓库仅附带 `.jpg`，`load_potential_map`/此处按 `.png` 找文件，**该资源缺失**。
  2. `W=compute_edge_energy(M)`；`perform_image_resize(M/W, m)` 把图缩放到 `n=400`（`warning off/on`）。
  3. `[start_points,end_points,center_point]=pick_start_end_point(M,1)`（`no_end_point=1`，仅取起点/中心，交互式）。
  4. 构造启发式：`r=|start-center|`、`s=|X-start|`，`H=r*(2π-2asin(s/(2r)))`，对 `s>2r` 的点 `H=π*r`（围绕中心的角向弧长估计）。
  5. `[D,S]=perform_circular_fast_marching_2d(W,start_points,center_point,options,H)`；`path=compute_geodesic(D,end_points,{...start_points...})`；缩放后 `plot_fast_marching_2d`。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["imread name.png → M; compute_edge_energy → W"] --> B["perform_image_resize 缩放到 n=400"]
    B --> C["pick_start_end_point(M,1) → start/center"]
    C --> D["构造圆环先验启发式 H"]
    D --> E["perform_circular_fast_marching_2d(W,start,center,options,H) → D,S"]
    E --> F["compute_geodesic → path → plot_fast_marching_2d"]
  ```

- **依赖 / Dependencies**: 需 mex `perform_circular_front_propagation_2d`（`perform_circular_fast_marching_2d` 支持第 5 参 `H`）；`perform_image_resize`（**已附带**）。⚠️ 数据 `data/stephanodiscusniagarae.png` **缺失**（仅有 `.jpg`）⇒ **缺数据不可运行**。
- **See also**: `test_circular`、`perform_circular_fast_marching_2d`

### `test_circular` — 椭圆/图像上的圆环传播
- **文件 / File**: [tests/test_circular.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_circular.m#L1-L85)（85 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。署名 **Gabriel Peyre / Toolbox Fast Marching (BSD)** —— 证据 [L3](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_circular.m#L3) `%   Copyright (c) 2004 Gabriel Peyre`。
- **被驱动的函数 / Functions exercised**: `perform_circular_fast_marching_2d`、`compute_geodesic`、`compute_edge_energy`、`pick_start_end_point`、`plot_fast_marching_2d`、`perform_image_resize`、`rescale`
- **功能 / Purpose**: EN: Circular FMM on either a synthetic ellipse mask or an image, with metric weighting near the center. / 中文：在合成椭圆掩膜或图像上做圆环 FM，并对中心附近度量加权。
- **工作区变量 / Workspace vars**: `n=400`、`name`、`Y,X`、`ax,ay`、`M`、`M_original`、`epsi`、`s`、`W`、`m`、`start_points`、`end_points`、`center_point`、`d`、`options`、`D`、`S`、`path`
- **变量流动 / Variable flow**: `n=400 →(1 name='ellipse' 时造椭圆掩膜 M，否则 imread)→(2 W=compute_edge_energy(M,3,0.05))→(3 perform_image_resize 缩放到 n)→(4 pick_start_end_point→start/end/center)→(5 W=W.*d 中心加权)→(6 perform_circular_fast_marching_2d→D,S→compute_geodesic→plot)`
- **算法步骤 / Algorithm**:
  1. `n=400`、`name` 最终 `'stephanodiscusniagarae'`（若改 `'ellipse'` 则用 `(X/0.8)²+(Y/0.6)²<1` 合成掩膜）。
  2. `epsi=0.05; s=3; W=compute_edge_energy(M,s,epsi)`。
  3. `perform_image_resize(M/W, m)` 把图缩放到 `n`；`[start_points,end_points,center_point]=pick_start_end_point(M)`（交互式）。
  4. 用 `d=sqrt((X-center)²+(Y-center)²)`（中心置 `1e-9`）乘 `W` 加权，避免零点收缩。
  5. `[D,S]=perform_circular_fast_marching_2d(W,start_points,center_point,options)`；`compute_geodesic` 提取路径，缩放后用 `plot_fast_marching_2d` 显示。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["name=ellipse 合成掩膜 或 imread 图像 → M"] --> B["W=compute_edge_energy(M,3,0.05)"]
    B --> C["perform_image_resize 缩放到 n=400"]
    C --> D["pick_start_end_point → start/end/center"]
    D --> E["W = W .* 到中心距离 d"]
    E --> F["perform_circular_fast_marching_2d → D,S"]
    F --> G["compute_geodesic → path → plot_fast_marching_2d"]
  ```

- **依赖 / Dependencies**: 需 mex `perform_circular_front_propagation_2d`；`perform_image_resize`（**已附带**）。默认 `name` 需 `data/stephanodiscusniagarae.png`（**缺失**）；改 `name='ellipse'` 可完全合成 ⇒ **默认缺数据不可运行，改用 ellipse 可运行（仍需 mex）**。
- **See also**: `test_circular_fast_marching_2d`、`test_circular_prior`、`perform_circular_fast_marching_2d`

### `test_constrained_map` — 约束传播（`constraint_map`）
- **文件 / File**: [tests/test_constrained_map.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_constrained_map.m#L1-L32)（32 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `perform_fast_marching`（`options.constraint_map`）、`load_potential_map`、`rescale`
- **功能 / Purpose**: EN: Constrain FMM propagation to a disk region via `constraint_map` and show Voronoi/state and distance. / 中文：用 `constraint_map` 把 FM 传播限制在圆盘内，并画状态/Voronoi 与距离。
- **工作区变量 / Workspace vars**: `n=300`、`name`（默认 `constant`）、`p=80`、`start_points`、`x`、`Y,X`、`L`、`I`、`options`、`M`、`W`、`D`、`Z`、`Q`
- **变量流动 / Variable flow**: `n=300, p=80 →(1 start_points 随机)→(2 由 X²+Y²<0.6² 造约束图 L (±Inf)→options.constraint_map=L)→(3 load_potential_map('constant',n)→M,W)→(4 perform_fast_marching(W,start_points,options)→D,Z,Q)→(5 subplot imagesc(Q')/imagesc(D') + 起点)`
- **算法步骤 / Algorithm**:
  1. `n=300`、`name='constant'`（`load_potential_map` 对 `constant` **合成** `M=ones*0.5`，无需外部文件）。
  2. `p=80` 随机 `start_points`；构造约束图 `L=zeros(n)-Inf`，令 `X²+Y²<0.6²` 处为 `+Inf`，赋给 `options.constraint_map`。
  3. `[M,W]=load_potential_map(name,n)`。
  4. `[D,Z,Q]=perform_fast_marching(W,start_points,options)` —— 只有圆盘内点可扩展，其余保持 `Inf`。
  5. `subplot` 左画 `imagesc(Q')`（Voronoi/状态）、右画 `imagesc(D')`，均叠画起点 `k.`。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=300; p=80 随机 start_points"] --> B["由圆盘 X²+Y²<0.6² 造 L (±Inf) → options.constraint_map"]
    B --> C["load_potential_map('constant',n) → M,W"]
    C --> D["perform_fast_marching(W,start_points,options) → D,Z,Q"]
    D --> E["subplot: imagesc(Q') / imagesc(D') + 起点"]
  ```

- **依赖 / Dependencies**: 需 mex `perform_front_propagation_2d`；`load_potential_map`（**已附带**，`constant` 为合成）。⚠️ **未编译 mex 则报错**；无外部数据依赖。
- **See also**: `generate_constrained_map`、`perform_fast_marching`

### `test_distance_approximation` — 采样策略对距离近似误差的影响
- **文件 / File**: [tests/test_distance_approximation.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_distance_approximation.m#L1-L233)（233 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `my_eval_distance`（**脚本内不可见**）、`get_color_from_index`（**未定义**）、`load_potential_map`、`perform_farthest_point_sampling`/`_boundary`、`perform_farthest_landmark_sampling`、`compute_distance_landmark`、`pick_start_end_point`、`num2string_fixeddigit`
- **功能 / Purpose**: EN: Quantify how the landmark-sampling strategy affects the accuracy of landmark-based distance approximation (error vs. #landmarks / memory). / 中文：量化不同地标采样策略对"地标距离近似"精度的影响（误差随地标数/内存变化）。
- **工作区变量 / Workspace vars**: `n=80`、`rep`、`name`（默认 `constant`）、`A`、`start_points`、`end_points`、`M`、`W`、`D`、`m`、`DL_base`、`base_points`、`err`、`err_list`、`lgd`、`landmark`、`DL_landmark`、`nbr_landmark_list`、`ldm_order_list`
- **变量流动 / Variable flow**: `n=80 →(1 load_potential_map(name,n)→M,W)→(2 由 A 或交互定 start/end)→(3 my_eval_distance 求"真"距离 D)→(4 对 50 个 base_points 求参考距离 DL_base)→(5 循环地标数/采样法：perform_farthest_point_sampling(_boundary)/_landmark_sampling 选点 → compute_distance_landmark 估距→求相对误差 err)→(6 画 log-log 误差曲线并 saveas)`
- **算法步骤 / Algorithm**:
  1. `n=80`；`name` 最终 `'constant'`；`[M,W]=load_potential_map(name,n)`（`constant` 合成）。
  2. 由 `A`（对 road2/mountain/constant 预置）或 `pick_start_end_point(M)` 定 `start_points`/`end_points`。
  3. `D=my_eval_distance(W,end_points)` 得"真"距离并出图 —— **`my_eval_distance` 只是 `perform_farthest_point_sampling.m` 的局部子函数，脚本无法调用** ⇒ 报错。
  4. 取 50 个 `base_points`，循环对每个求参考距离 `DL_base(:,:,s)=my_eval_distance(W,base_points(:,s))`（同一缺失问题）。
  5. 双层循环：地标数 `[16:16:128]`，采样法 `{'rand','farthest','trialerror','farthestboundary'}`；用 `compute_distance_landmark(...,'multiproxy0..5')` 估距，对 base_points 求 RMSE 得 `err`，最后画 `log10(#landmarks)` / `log10(memory)` vs `log10(Error)`。**无 `assert`，误差以曲线呈现**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=80; load_potential_map → M,W"] --> B["start/end (A 预置 或 ginput)"]
    B --> C["D=my_eval_distance(W,end_points) (缺失)"]
    C --> D["50 个 base_points → DL_base 参考距离"]
    D --> E{"双层循环: 地标数 × 采样法"}
    E --> F["perform_farthest_point_sampling(_boundary)/_landmark_sampling 选地标"]
    F --> G["compute_distance_landmark 估距 → RMSE err"]
    G --> H["plot log-log 误差曲线 + saveas"]
    G --> E
  ```

- **依赖 / Dependencies**: `my_eval_distance`（**非可调用**）、`get_color_from_index`（**未定义**）；其余 `load_potential_map`/`perform_farthest_point_sampling(_boundary)`/`perform_farthest_landmark_sampling`/`compute_distance_landmark` 均已附带。⚠️ **必然报错，不可运行**。
- **See also**: `test_distance_compression`、`test_landmark_error`、`compute_distance_landmark`

### `test_distance_compression` — 用地标启发估计距离图并看残差
- **文件 / File**: [tests/test_distance_compression.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_distance_compression.m#L1-L99)（99 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `compute_heuristic_landmark`、`perform_fast_marching`、`perform_farthest_point_sampling`、`load_potential_map`、`mmax`
- **功能 / Purpose**: EN: Estimate a distance map from previous landmark distances via a heuristic, then inspect the residual (`DL-DE`) maps and histograms. / 中文：由已有地标距离用启发式估计距离图，检视残差 `DL-DE` 的分布与直方图。
- **工作区变量 / Workspace vars**: `rep`、`n=150`、`name`（默认 `road2`）、`landmark_init`（默认 `rand`）、`p=30`、`landmarks`、`DL`、`DE`、`DW`、`M`、`W`、`m`、`k`
- **变量流动 / Variable flow**: `n=150 →(1 load_potential_map(name,n)→M,W)→(2 选 30 个地标 landmarks)→(3 对每个地标 perform_fast_marching→DL(:,:,k) 真距离)→(4 DE(:,:,k)=compute_heuristic_landmark(DL(:,:,1:k-1),landmarks(:,k)) 估计)→(5 DW=DL-DE 残差→imagesc 前 9 帧 + hist 16 帧)→ return`
- **算法步骤 / Algorithm**:
  1. `n=150`；`name` 最终 `'road2'`；`[M,W]=load_potential_map(name,n)`（需 `data/road2.png`，**已附带**）。
  2. `landmark_init='rand'`：`landmarks=floor(rand(2,30)*n)+1`（否则 `perform_farthest_point_sampling`）。
  3. 循环 `p=30` 个地标，各跑一次 `perform_fast_marching(W,landmarks(:,k))` 得真距离 `DL(:,:,k)`。
  4. 从第 2 个起 `DE(:,:,k)=compute_heuristic_landmark(DL(:,:,1:k-1),landmarks(:,k))` 估计距离。
  5. `DW=DL-DE`；`clf` 输出前 9 帧残差 `imagesc(abs(DW(:,:,k)))`，再 `hist` 对比 16 个点的 `|DW|` 与 `|DL|`；随后 `return`（后续矢量量化段不执行）。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=150; load_potential_map('road2') → M,W"] --> B["rand 选 30 个地标 landmarks"]
    B --> C["perform_fast_marching × 30 → DL 真距离"]
    C --> D["compute_heuristic_landmark → DE 估计距离"]
    D --> E["DW = DL - DE 残差"]
    E --> F["imagesc 前 9 帧 + hist 16 帧"]
    F --> G["return"]
  ```

- **依赖 / Dependencies**: 需 mex `perform_front_propagation_2d`；`compute_heuristic_landmark`、`perform_farthest_point_sampling`、`mmax`（**均已附带**）；`data/road2.png`（**已附带**）⇒ **编译 mex 后基本可运行**。
- **See also**: `compute_heuristic_landmark`、`test_distance_approximation`、`test_fmstar_error`

### `test_eccentricity` — 2D 形状偏心率变换
- **文件 / File**: [tests/test_eccentricity.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_eccentricity.m#L1-L154)（154 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `compute_eccentricity_transform`、`display_eccentricity`、`perform_fast_marching`（含 `constraint_map`）、`convert_distance_color`、`load_image`、`rescale`
- **功能 / Purpose**: EN: Compare Euclidean vs. geodesic distances inside a binary shape and compute its eccentricity transform. / 中文：对比二值形状内的欧氏 vs 测地距离，并计算其偏心率变换。
- **工作区变量 / Workspace vars**: `name`（默认 `giraffe`）、`method`（默认 `mean`）、`metric`（默认 `geodesic`）、`options`、`s`、`rep`、`testname`、`n=256`、`M`、`x,y`、`start_points`、`W`、`L`、`D0,S,Q`、`D1`、`E`、`Ind`、`eccx,eccy`、`J`、`I`、`v`
- **变量流动 / Variable flow**: `n=256 →(1 M=rescale(load_image(name,n))>0.5 二值形状)→(2 W=ones(n); 无约束 FM→D0；加 L 约束 FM→D1)→(3 display_eccentricity(D0/D1) 对比)→(4 compute_eccentricity_transform(M,options)→E,Ind)→(5 画 E、偏心点 J、直方图，并 imwrite 出图)`
- **算法步骤 / Algorithm**:
  1. `name` 最终 `'giraffe'`、`method='mean'`、`metric='geodesic'`；`n=256`；`M=double(M>0.5)` 得形状掩膜（必要时取反使边界为黑）。
  2. `x=y=50` 为起点；`W=ones(n)`；先**无约束** `[D0,S,Q]=perform_fast_marching(W,start_points,options)` 得"欧氏"距离。
  3. 造 `L=ones`（`M==1` 处 `+Inf`，其余 `-Inf`）作 `constraint_map`，再跑一次得测地距离 `D1`；`D0(M==0)=0; D1(M==0)=0`。
  4. `subplot` 用 `display_eccentricity(D0/D1,1)` 对比并叠画起点。
  5. `options.s=s; options.nb_samples=300; [E,Ind]=compute_eccentricity_transform(M,options)`；画偏心率 `E`、对比 `ecc` 分支的偏心点、直方图；最后 `convert_distance_color` 合成彩色图并 `imwrite`。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=256; load_image → 二值形状 M"] --> B["W=ones; perform_fast_marching → D0 欧氏"]
    B --> C["L 约束 X; perform_fast_marching → D1 测地"]
    C --> D["display_eccentricity 对比 D0 / D1"]
    D --> E["compute_eccentricity_transform(M,options) → E,Ind"]
    E --> F["画 E / 偏心点 / 直方图 + imwrite"]
  ```

- **依赖 / Dependencies**: 需 mex `perform_front_propagation_2d`；`compute_eccentricity_transform`、`display_eccentricity`、`convert_distance_color`（**均已附带**）；`load_image('giraffe')` 需 `data/giraffe.gif`（**已附带**）⇒ **编译 mex 后可运行**。
- **See also**: `compute_eccentricity_transform`、`display_eccentricity`、`test_eucldist`

### `test_eucldist` — 欧氏距离变换最小冒烟测试
- **文件 / File**: [tests/test_eucldist.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_eucldist.m#L1-L5)（5 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `eucdist2`、`load_image`
- **功能 / Purpose**: EN: Minimal smoke test of the `eucdist2` Euclidean distance transform on a binary disk. / 中文：`eucdist2` 欧氏距离变换在二值圆盘上的最小冒烟测试。
- **工作区变量 / Workspace vars**: `M`、`D`、`L`
- **变量流动 / Variable flow**: `M=load_image('disk',128) →(1 M=logical(M) 二值)→(2 [D,L]=eucdist2(M)) → D,L`
- **算法步骤 / Algorithm**:
  1. `M = load_image('disk',128)` 载入圆盘图。
  2. `M = logical(M)` 转二值掩膜。
  3. `[D,L] = eucdist2(M)` 求欧氏距离变换（返回距离 `D` 与最近点索引 `L`）。**无断言、无绘图**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load_image('disk',128) → M"] --> B["M = logical(M)"]
    B --> C["[D,L] = eucdist2(M)"]
    C --> Z["输出 D,L"]
  ```

- **依赖 / Dependencies**: `eucdist2`（仓库**已附带预编译** `eucdist2.dll` / `eucdist2.mexglx`，或经 [compile_mex.m#L14](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compile_mex.m#L14) 由 `mex/eucdist2.c` 重编译）；`load_image('disk')` 需 `data/disk.png`（**缺失**）⇒ **缺数据不可运行**。
- **See also**: `eucdist2`、`test_eccentricity`

### `test_farthest_sampling_2d` — 2D 势场最远点采样 + Voronoi 剖分
- **文件 / File**: [tests/test_farthest_sampling_2d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_farthest_sampling_2d.m#L1-L143)（143 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `perform_farthest_point_sampling`、`perform_fast_marching`、`compute_voronoi_triangulation`、`compute_edges`、`plot_tensor_field`（**缺失**）、`plot_edges`、`load_potential_map`、`perform_tensor_recomp`、`perform_histogram_equalization`、`num2string_fixeddigit`
- **功能 / Purpose**: EN: Farthest-point sampling on a 2D potential (or anisotropic tensor) field, followed by Voronoi triangulation and display. / 中文：在 2D 势场（或各向异性张量场）上做最远点采样，再 Voronoi 三角剖分与显示。
- **工作区变量 / Workspace vars**: `n=300`、`name`（默认 `anisotropic`）、`mask_name`、`mask`、`options`、`rep`、`aniso_type`、`u,U,sigma`、`aniso`、`V,W`、`M0,M`、`ms,lw`、`I,vx,vy`、`landmark`、`nbr_landmarks`、`D,Z,Q`、`Q1`、`faces`、`edges`、`sel`
- **变量流动 / Variable flow**: `n=300 →(1 依 name 造 M/W；anisotropic 时由张量场得 W 并 plot_tensor_field)→(2 取 mask 内一点为 landmark 起点)→(3 循环 nbr_landmarks：perform_farthest_point_sampling(W,landmark,k,options))→(4 perform_fast_marching(W,landmark)→D,Z,Q; compute_voronoi_triangulation→faces; compute_edges→edges)→(5 imageplot 出 Voronoi/采样/三角化三图并 saveas)`
- **算法步骤 / Algorithm**:
  1. `n=300`；`name` 最终 `'anisotropic'`；`mask` 全 1，按 `mask>.5` 填 `options.constraint_map` 的 ±Inf。
  2. `aniso_type='varying'`：`U=randn(n,n,2)` 迭代 `perform_blurring`+`perform_vf_normalization`，`aniso=.01`，`W=perform_tensor_recomp(U,V,1,aniso)`；若存在 `perform_lic` 合成纹理 `M`，否则 `M=U`。
  3. `clf; plot_tensor_field(W,M); saveas(...)` —— **`plot_tensor_field` 未定义** ⇒ 在此即报错。
  4. （此后逻辑）取 `mask==1` 首点为起点；对 `nbr_landmarks=[100 200 500 800 1200]` 逐步 `perform_farthest_point_sampling`；`perform_fast_marching` 求 `D,Z,Q`；`compute_voronoi_triangulation`/`compute_edges` 得三角网；`imageplot` 显示 Voronoi、采样距离、三角化并 `saveas`。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=300; name=anisotropic → 张量场 W, 背景 M"] --> B["plot_tensor_field(W,M) 缺失 ⇒ 报错"]
    B -.-> C["取 mask 内起点 → landmark"]
    C --> D{"遍历 nbr_landmarks ∈ {100..1200}"}
    D --> E["perform_farthest_point_sampling(W,landmark,k,options)"]
    E --> F["perform_fast_marching → D,Z,Q"]
    F --> G["compute_voronoi_triangulation + compute_edges"]
    G --> H["imageplot Voronoi/采样/三角化 + saveas"]
    H --> D
  ```

- **依赖 / Dependencies**: `plot_tensor_field`（**未定义**，默认 `name='anisotropic'` 必然命中）⇒ **不可运行**；另需 mex `perform_front_propagation_2d`；`plot_edges` 用 gptoolbox 的 `mesh/plot_edges`（**已附带**）。
- **See also**: `perform_farthest_point_sampling`、`compute_voronoi_triangulation`、`test_farthest_sampling_shape`

### `test_farthest_sampling_3d` — 体数据最远点采样 + 三角化/切片
- **文件 / File**: [tests/test_farthest_sampling_3d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_farthest_sampling_3d.m#L1-L97)（97 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `perform_farthest_point_sampling`（3D）、`perform_fast_marching`（3D）、`compute_voronoi_triangulation`、`delaunay3`（MATLAB 旧内置）、`perform_image_extension`（**缺失**，仅 geodesic 分支）、`plot_mesh`、`vol3d`、`imageplot`、`rescale`、`crop`、`clamp`
- **功能 / Purpose**: EN: Farthest-point sampling of a volumetric potential, then Delaunay (or Voronoi) tetrahedralisation and cutting-plane display. / 中文：对体势场做最远点采样，再 Delaunay（或 Voronoi）四面体化并切片显示。
- **工作区变量 / Workspace vars**: `n=150`、`name`（默认 `medical`）、`rep`、`W`、`x,X,Y,Z`、`r`、`M`、`method`（默认 `delaunay`）、`k`、`W1`、`ms,lw`、`npoints`、`vertex`、`faces`、`w,t,delta,offlist`、`options`、`D,Z,Q`
- **变量流动 / Variable flow**: `n=150 →(1 依 name 造 W：constant/sphere 合成，medical 载 brain1-crop-256.mat)→(2 delaunay 分支跳过 padding)→(3 循环 npoints：perform_farthest_point_sampling(W,vertex,k))→(4 delaunay3 或 FMM+compute_voronoi_triangulation→faces)→(5 vol3d/plot_mesh 显示采样与切片并 saveas)`
- **算法步骤 / Algorithm**:
  1. `n=150`；`name` 最终 `'medical'`：`load brain1-crop-256.mat; M=crop(M,n); W=rescale(-abs(M-median(M(:)))); W=rescale(clamp(W,.6,1),.2,1)`（其它分支为合成 sphere/constant）。
  2. `method='delaunay'`（默认）：该分支**不**做 `perform_image_extension` 边界 padding；仅 `method='geodesic'` 才调用（**该函数未定义**）。
  3. `for npoints=[400 800 1600 3000 5000 10000]`：`vertex=perform_farthest_point_sampling(W,vertex,npoints-size(vertex,2))`（3D，需 mex）。
  4. 三角化：`faces=delaunay3(vertex(1,:),vertex(2,:),vertex(3,:))'`（delaunay 法）或 `perform_fast_marching(W1,...)+compute_voronoi_triangulation`（geodesic 法）。
  5. 用 `vol3d`/`imageplot` 显示点云，再在 5 个切片平面上 `plot_mesh(vertex,faces,options)`；每步 `saveas`。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=150; name=medical → 载 brain1-crop-256.mat → W"] --> B{"method"}
    B -- geodesic --> B1["perform_image_extension padding (缺失)"]
    B -- delaunay --> C["跳过 padding"]
    C --> D{"遍历 npoints"}
    D --> E["perform_farthest_point_sampling(W,vertex,k)"]
    E --> F{"method=delaunay ?"}
    F -- 是 --> G["delaunay3 → faces"]
    F -- 否 --> H["perform_fast_marching + compute_voronoi_triangulation → faces"]
    G --> I["vol3d / plot_mesh 切片显示 + saveas"]
    H --> I
    I --> D
  ```

- **依赖 / Dependencies**: 默认 `name='medical'` 需 `brain1-crop-256.mat`（**缺失**）；`perform_image_extension`（**未定义**，仅 geodesic 分支）；`delaunay3`（MATLAB 旧内置，新版本可能移除）；`vol3d`、`crop`、`clamp`、`perform_farthest_point_sampling`（**已附带**）。⚠️ **缺数据不可运行**。
- **See also**: `perform_farthest_point_sampling`、`test_farthest_sampling_2d`、`test_vol3d`

### `test_farthest_sampling_mesh` — 三角网格上的最远点采样与重网格化
- **文件 / File**: [tests/test_farthest_sampling_mesh.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_farthest_sampling_mesh.m#L1-L82)（82 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `perform_farthest_point_sampling_mesh`、`perform_fast_marching_mesh`、`compute_voronoi_triangulation_mesh`、`perform_faces_reorientation`（**缺失**）、`read_mesh`、`plot_mesh`、`num2string_fixeddigit`
- **功能 / Purpose**: EN: Farthest-point sampling on a triangle mesh, then Voronoi remeshing and display. / 中文：在三角网格上做最远点采样，再 Voronoi 重网格化并显示。
- **工作区变量 / Workspace vars**: `n=300`、`repimg`、`name`（默认 `pelvis`）、`vertex`、`faces`、`options`、`tmp,I,J`、`save_images`、`i`、`landmark`、`nbr_landmarks`、`D,Z,Q`、`vertex_voronoi`、`faces_voronoi`、`col`
- **变量流动 / Variable flow**: `name='pelvis' →(1 read_mesh(name)→vertex,faces 归一为 3×N/N×M)→(2 循环 nbr_landmarks：perform_farthest_point_sampling_mesh(vertex,faces,landmark,k))→(3 perform_fast_marching_mesh→D,Z,Q; compute_voronoi_triangulation_mesh→vertex_voronoi,faces_voronoi)→(4 perform_faces_reorientation 定向→plot_mesh 显示采样/重网格并 saveas)`
- **算法步骤 / Algorithm**:
  1. `name` 最终 `'pelvis'`（另含 nefertiti/beetle/…）；`[vertex,faces]=read_mesh(name)`，并按行列数把顶点/面转成 `d×N` / `l×M` 约定。
  2. 若 `name='brain'` 会排序并去掉一半顶点（默认不触发）。
  3. `for nbr_landmarks=[200 500 1000 2000 5000 10000]`：`landmark=perform_farthest_point_sampling_mesh(vertex,faces,landmark,k)`（**需 mesh FMM mex**）。
  4. `[D,Z,Q]=perform_fast_marching_mesh(vertex,faces,landmark)`；`[vertex_voronoi,faces_voronoi]=compute_voronoi_triangulation_mesh(Q,vertex,faces)`。
  5. `faces_voronoi=perform_faces_reorientation(vertex_voronoi,faces_voronoi,options)` —— **该函数未定义** ⇒ 报错；随后本应 `plot_mesh` 显示采样点与重网格面并 `saveas`。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["name=pelvis → read_mesh → vertex,faces"] --> B{"遍历 nbr_landmarks ∈ {200..10000}"}
    B --> C["perform_farthest_point_sampling_mesh(vertex,faces,landmark,k)"]
    C --> D["perform_fast_marching_mesh → D,Z,Q"]
    D --> E["compute_voronoi_triangulation_mesh → vertex_voronoi,faces_voronoi"]
    E --> F["perform_faces_reorientation (缺失) ⇒ 报错"]
    F -.-> G["plot_mesh 显示采样/重网格 + saveas"]
    G --> B
  ```

- **依赖 / Dependencies**: `perform_faces_reorientation`（**未定义**）⇒ **不可运行**；`read_mesh` 需 `../toolbox_graph_data/off/*.off`（**缺失**）；另需 mesh FMM mex `perform_front_propagation_mesh`。
- **See also**: `perform_farthest_point_sampling_mesh`、`compute_voronoi_triangulation_mesh`、`test_propagation_mesh`

### `test_farthest_sampling_shape` — 形状内部自适应最远点采样与网格生成
- **文件 / File**: [tests/test_farthest_sampling_shape.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_farthest_sampling_shape.m#L1-L84)（84 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。**文件头无版权行**；按目录惯例署名 Gabriel Peyre / Toolbox Fast Marching (BSD)。
- **被驱动的函数 / Functions exercised**: `perform_farthest_point_sampling`（形状约束）、`perform_fast_marching`、`compute_voronoi_triangulation`、`compute_shape_boundary`、`compute_edges`、`plot_edges`、`load_image`、`rescale`
- **功能 / Purpose**: EN: Adaptive farthest-point sampling inside a shape (bounds to boundary distance) and mesh generation. / 中文：在形状内部做自适应最远点采样（用边界距离构造"自适应"权重）并生成网格。
- **工作区变量 / Workspace vars**: `name='mm'`、`n=400`、`Ma`、`M`、`repimg`、`M1`、`mask`、`boundary`、`Ibound`、`p=400`、`use_adaptive=1`、`D,Z,Q`、`R`、`D1`、`H`、`W`、`L`、`I`、`vertex`、`options`、`faces`
- **变量流动 / Variable flow**: `n=400 →(1 Ma=load_image('mm') 归一, 内嵌到 M 并留边距)→(2 mask=1-(M1==M1(1)); boundary=compute_shape_boundary(mask))→(3 use_adaptive: perform_fast_marching(ones(n),boundary)→D; 由 D 造自适应权重 W)→(4 perform_farthest_point_sampling(W,vertex,p,options)→vertex)→(5 perform_fast_marching→D,Z,Q; compute_voronoi_triangulation→faces; plot_edges 显示并 saveas)`
- **算法步骤 / Algorithm**:
  1. `name='mm'`、`n=400`；`Ma=load_image(name,n-10)`，把非零区域内嵌到 `M`（`6:n-5` 行李白边距）。
  2. `M1=mean(M,3); mask=1-(M1==M1(1))`（背景为 0）；`boundary=compute_shape_boundary(mask)'`，边界像素强制为 1。
  3. `use_adaptive=1`：`[D,Z,Q]=perform_fast_marching(ones(n),boundary)` 求到边界的距离；`D1=min(rescale(D),0.8)`、`H=sqrt(R²-(D1-R)²)*n` 构造中间大的自适应场，`W=rescale(D,0.1,1)`。
  4. 约束 `L`（`mask` 内 `+Inf`）赋 `options.constraint_map`；`vertex=perform_farthest_point_sampling(W,[n/2;n/2],p,options)`（`p=400`，需 mex）。
  5. `[D,Z,Q]=perform_fast_marching(W,vertex,options)`；`faces=compute_voronoi_triangulation(Q,vertex)`；`plot_edges(compute_edges(faces),...)` 叠在形状上显示并 `saveas`。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=400; load_image('mm') → M, mask"] --> B["compute_shape_boundary(mask) → boundary"]
    B --> C{"use_adaptive ?"}
    C -- 是 --> D["perform_fast_marching(ones,boundary) → D; 造自适应权重 W"]
    C -- 否 --> D2["W = ones(n)"]
    D --> E["perform_farthest_point_sampling(W,[n/2;n/2],p,options) → vertex"]
    D2 --> E
    E --> F["perform_fast_marching(W,vertex) + compute_voronoi_triangulation → faces"]
    F --> G["plot_edges 叠形状显示 + saveas"]
  ```

- **依赖 / Dependencies**: 需 mex `perform_front_propagation_2d`；`compute_shape_boundary`、`compute_voronoi_triangulation`、`perform_farthest_point_sampling`、`plot_edges`（gptoolbox `mesh/plot_edges`）**均已附带**；`load_image('mm')` 需 `data/mm.png`（**已附带**）⇒ **编译 mex 后可运行**。
- **See also**: `perform_farthest_point_sampling`、`compute_shape_boundary`、`batch_shape_meshing`

### `test_fast_marching_2d` — 2D FM + 多条测地线提取
- **文件 / File**: [tests/test_fast_marching_2d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fast_marching_2d.m#L1-L56)（56 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。署名 **Gabriel Peyre / Toolbox Fast Marching (BSD)** —— 证据 [L3](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fast_marching_2d.m#L3) `%   Copyright (c) 2004 Gabriel Peyre`。
- **被驱动的函数 / Functions exercised**: `perform_fast_marching`（2D）、`compute_geodesic`、`convert_distance_color`、`load_potential_map`、`pick_start_end_point`、`imageplot`、`rescale`
- **功能 / Purpose**: EN: Baseline 2D FMM demo — one start point, 30 random end points, extract and plot all geodesics over the distance field. / 中文：2D FM 基线演示：单起点、30 个随机终点，提取并叠画全部测地线到距离场上。
- **工作区变量 / Workspace vars**: `n=600`、`name`（默认 `mountain`）、`M`、`W`、`rep`、`start_points`、`options`、`D`、`S`、`npaths`、`end_points`、`paths`、`ms,lw`、`A`
- **变量流动 / Variable flow**: `n=600 →(1 load_potential_map(name,n)→M,W)→(2 imwrite 势场图)→(3 pick_start_end_point(M)→start_points)→(4 perform_fast_marching(W,start_points,options)→D,S)→(5 30 个随机终点循环 compute_geodesic(D,end_points(:,i))→paths)→(6 convert_distance_color(D)→A, imageplot 叠画路径与端点→saveas)`
- **算法步骤 / Algorithm**:
  1. `n=600`；`name` 最终 `'mountain'`；`[M,W]=load_potential_map(name,n)`（需 `data/mountain.png`，**已附带**）；`imwrite(rescale(W),...)` 存势场图。
  2. `start_points=pick_start_end_point(M)`（**交互式 `ginput`**）。
  3. `options.nb_iter_max=Inf; [D,S]=perform_fast_marching(W,start_points,options)`（需 mex）。
  4. `npaths=30` 个随机终点；循环 `paths{i}=compute_geodesic(D,end_points(:,i))`（若返回长度 2 则沿用上一条）。
  5. `A=convert_distance_color(D)`；`imageplot(A)` 后 `plot` 每条路径、终点 `b.`、起点 `r.`，`saveas` 到 `results/geodesic-2d/`。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=600; load_potential_map('mountain') → M,W"] --> B["pick_start_end_point(M) → start_points"]
    B --> C["perform_fast_marching(W,start_points) → D,S"]
    C --> D["30 个随机终点: compute_geodesic → paths"]
    D --> E["convert_distance_color(D) → A; imageplot 叠画路径"]
    E --> F["saveas 到 results/geodesic-2d/"]
  ```

- **依赖 / Dependencies**: 需 mex `perform_front_propagation_2d`；`load_potential_map`/`compute_geodesic`/`convert_distance_color`（**均已附带**）；`data/mountain.png`（**已附带**）⇒ **编译 mex 并经交互取点后可运行**。
- **See also**: `test_fast_marching_3d`、`compute_geodesic`、`convert_distance_color`

### `test_fast_marching_3d` — 3D 高斯势场 FM + 单条测地线
- **文件 / File**: [tests/test_fast_marching_3d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fast_marching_3d.m#L1-L23)（23 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。署名 **Gabriel Peyre / Toolbox Fast Marching (BSD)** —— 证据 [L3](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fast_marching_3d.m#L3) `%   Copyright (c) 2004 Gabriel Peyre`。
- **被驱动的函数 / Functions exercised**: `perform_fast_marching`（3D）、`compute_geodesic`、`plot_fast_marching_3d`
- **功能 / Purpose**: EN: Minimal 3D FMM demo on a synthetic Gaussian potential, extract one geodesic between two corners of the cube. / 中文：合成高斯势场上的最小 3D FM 演示，提取立方体两角之间的一条测地线。
- **工作区变量 / Workspace vars**: `n=40`、`x`、`X,Y,Z`、`sigma`、`W`、`k=5`、`start_points`、`end_points`、`options`、`D`、`S`、`path`
- **变量流动 / Variable flow**: `n=40 →(1 x=linspace(-1,1,n); meshgrid→X,Y,Z)→(2 W=1./(1+exp(-(X²+Y²+Z²)/sigma²))) 高斯谷)→(3 start=(35,5,5), end=(5,35,35)→options.nb_iter_max=Inf)→(4 perform_fast_marching(W,start,options)→D,S)→(5 compute_geodesic(D,end)→path → plot_fast_marching_3d)`
- **算法步骤 / Algorithm**:
  1. `n=40`；`x=-1:2/(n-1):1`，`[X,Y,Z]=meshgrid(x,x,x)`，`sigma=0.4`。
  2. `W=1./(1+exp(-(X²+Y²+Z²)/sigma²))`（中心处小而慢、向外趋 1 的"高斯谷"），路径倾向绕开立方体中心。
  3. `k=5`；`start_points=[n-k;k;k]`、`end_points=[k;n-k;n-k]`；`options.nb_iter_max=Inf`。
  4. `[D,S]=perform_fast_marching(W,start_points,options)`（需 3D mex）；`path=compute_geodesic(D,end_points)`。
  5. `plot_fast_marching_3d(D,S,path,start_points,end_points)` 显示。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=40; meshgrid → X,Y,Z"] --> B["W=1./(1+exp(-(X²+Y²+Z²)/sigma²))"]
    B --> C["start=[35;5;5], end=[5;35;35]"]
    C --> D["perform_fast_marching(W,start,options) → D,S"]
    D --> E["compute_geodesic(D,end) → path"]
    E --> F["plot_fast_marching_3d 显示"]
  ```

- **依赖 / Dependencies**: 需 3D mex `perform_front_propagation_3d`（由 `compile_mex` 编译；**3D 无纯 MATLAB 回退**）；`compute_geodesic`、`plot_fast_marching_3d`（**已附带**）；**无外部数据**⇒ **编译 mex 后可运行**。
- **See also**: `test_fast_marching_2d`、`test_bug`、`plot_fast_marching_3d`

### `test_fmstar_2d` — FM\* 2D 多权重对比
- **文件 / File**: [tests/test_fmstar_2d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_2d.m#L1-L122)（122 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。署名 **Gabriel Peyre / Toolbox Fast Marching (BSD)** —— 证据 [L5](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_2d.m#L5) `%   Copyright (c) 2004 Gabriel Peyre`。
- **被驱动的函数 / Functions exercised**: `perform_fmstar_2d`、`compute_geodesic`、`compute_edge_energy`、`pick_start_end_point`、`plot_fast_marching_2d`、`load_potential_map`
- **功能 / Purpose**: EN: Run FM\* (heuristic FMM) on a 2D potential for a sweep of heuristic weights, comparing paths. / 中文：在一组启发权重下对 2D 势场运行 FM\*（启发式 FMM）并对比路径。
- **工作区变量 / Workspace vars**: `save_image`、`n=600`、`rep`、`energy_type`（默认 `edge`）、`propagation_type`（默认 `circular`）、`geodesic_type`（默认 `punctual`）、`group_witdh`、`name`（默认 `diatom2`）、`M`、`W`、`start_point,end_point,center_point`、`start_points,end_points`、`s,epsi`、`weight_list`、`reduc_factor`、`k`、`options`、`weight`、`D,S`、`d`、`opt`、`path`
- **变量流动 / Variable flow**: `n=600 →(1 load_potential_map(name,n)→M,W)→(2 pick_start_end_point(取 center)→3 组装 start/end 点集→(4 energy_type=edge: W=compute_edge_energy(M,s,epsi,center))→(5 循环 weight_list: perform_fmstar_2d(W,start_points,end_points,options)→D,S)→(6 找最小作用终点 compute_geodesic→path → plot_fast_marching_2d + saveas)`
- **算法步骤 / Algorithm**:
  1. `n=600`；`name` 最终 `'diatom2'`；`[M,W]=load_potential_map(name,n)`（`other` 分支走 `imread(data/diatom2.png)` —— **该资源缺失**）。
  2. `propagation_type='circular'`、`geodesic_type='punctual'`：`pick_start_end_point(M)` 取起点/终点/中心（交互式），设 `options.center_point`。
  3. `energy_type='edge'`：圆周模式 `W=compute_edge_energy(M,s,epsi,center_point)`（`s=3, epsi=1e-6`）。
  4. `weight_list=[0,0.5,0.8,0.9,0.95,1,1.02,1.05]`（启发权重）；设 `reduc_factor=.8`、`options.Tmax`、`options.start_points`。
  5. 循环：`[D,S]=perform_fmstar_2d(W,start_points,end_points,options)`；在 `end_points` 上取作用最小者，`compute_geodesic(D,end_point_min,opt)`；`plot_fast_marching_2d(M,S,path,...)` 并将 `.eps`/`.png` 存到 `images/multires/`。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=600; load_potential_map(name) → M,W"] --> B["pick_start_end_point → start/end/center"]
    B --> C["W=compute_edge_energy(M,s,epsi,center)"]
    C --> D{"遍历 weight_list (启发权重)"}
    D --> E["perform_fmstar_2d(W,start_points,end_points,options) → D,S"]
    E --> F["取最小作用终点 → compute_geodesic → path"]
    F --> G["plot_fast_marching_2d + saveas"]
    G --> D
  ```

- **依赖 / Dependencies**: 需 mex `perform_circular_front_propagation_2d`（`propagation_type='circular'`）与 FMM 后端；`perform_fmstar_2d`、`compute_edge_energy`、`plot_fast_marching_2d`（**已附带**）。默认 `name='diatom2'` 需 `data/diatom2.png`（**缺失**）⇒ **缺数据不可运行**。
- **See also**: `test_fmstar_3d`、`test_fmstar_weight_2d`、`perform_fmstar_2d`

### `test_fmstar_3d` — FM\* 3D 三型势场对比
- **文件 / File**: [tests/test_fmstar_3d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_3d.m#L1-L102)（102 行）
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments)。署名 **Gabriel Peyre / Toolbox Fast Marching (BSD)** —— 证据 [L5](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_3d.m#L5) `%   Copyright (c) 2004 Gabriel Peyre`。
- **被驱动的函数 / Functions exercised**: `perform_fmstar_3d`、`compute_geodesic`、`compute_edge_energy`、`perform_image_resize`、`plot_volumetric_data`、`plot_fast_marching_3d`
- **功能 / Purpose**: EN: Run 3D FM\* with constant/gaussian/heart potentials over a sweep of weights and display paths in volume. / 中文：对 constant/gaussian/heart 三型 3D 势场，在一组权重下运行 FM\* 并显示体路径。
- **工作区变量 / Workspace vars**: `save_image=0`、`n=100`、`type`（默认 `gaussian`）、`reduc_factor`、`W`、`M`、`x,X,Y,Z`、`sigma`、`a,b`、`epsi,s`、`k=5`、`AZ`、`weight_list`、`start_points`、`end_points`、`D,S`、`path`、`options`
- **变量流动 / Variable flow**: `n=100 →(1 依 type 造 W/M：constant/gaussian 合成，heart 载 heart1 并 resize+compute_edge_energy)→(2 依 type 设 start/end 与 weight_list)→(3 plot_volumetric_data(M) 预览)→(4 循环 weight_list: perform_fmstar_3d(W,start,end,reduc_factor,weight)→D,S)→(5 compute_geodesic(D,end)→path → plot_fast_marching_3d + (可选 saveas))`
- **算法步骤 / Algorithm**:
  1. `n=100`；`type` 最终 `'gaussian'`：`W=1./(1+exp(-(X²+Y²+Z²)/sigma²))`（`sigma=.8`），`M=W`（constant/heart 分支分别为全 1 与 `heart1` 数据）。
  2. `n=length(W)`；`type='gaussian'` 分支定起点 `[n-k;k;k]`、终点 `[k;n-k;n-k]`，`AZ=125+180`，`weight_list=[0,0.5,1,1.1,1.2]`。
  3. `plot_volumetric_data(M); view(AZ,30)` 预览势场。
  4. `for weight=weight_list`：`[D,S]=perform_fmstar_3d(W,start_points,end_points,reduc_factor,weight)`（`reduc_factor=1`）。
  5. `path=compute_geodesic(D,end_points)`；`plot_fast_marching_3d(M,S,path,start_points,end_points,{plot_planes=0})` 显示并 `camlight; view(AZ,30)`；`save_image=0` 故不落盘。**无断言**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=100; 依 type(constant/gaussian/heart) 造 W,M"] --> B["依 type 设 start/end 与 weight_list"]
    B --> C["plot_volumetric_data(M) 预览"]
    C --> D{"遍历 weight_list"}
    D --> E["perform_fmstar_3d(W,start,end,reduc_factor,weight) → D,S"]
    E --> F["compute_geodesic(D,end) → path"]
    F --> G["plot_fast_marching_3d 显示 (save_image=0)"]
    G --> D
  ```

- **依赖 / Dependencies**: 需 3D mex `perform_front_propagation_3d` 与 `perform_fmstar_3d`（**已附带**）；`plot_volumetric_data`、`plot_fast_marching_3d`、`compute_geodesic`（**已附带**）；`perform_image_resize`（heart 分支，**已附带**）。默认 `type='gaussian'` 为**纯合成**，`save_image=0` 不写盘 ⇒ **编译 mex 后可运行**。
- **See also**: `test_fmstar_2d`、`perform_fmstar_3d`、`test_fast_marching_3d`

---

## 函数清单核对表 / Coverage Checklist

| # | 脚本 / Script | 驱动求解器 / Solver | 可否运行 / Runnable | 阻塞原因 / Blocker |
|---|---|---|---|---|
| 1 | `test_active_contour` | `perform_active_contour` | 是（待数据） | 需 `data/brain.png`（已附带）；重计算 `n^2` 循环 |
| 2 | `test_anisotropic_feth` | 各向异性 FMM + `fm2dAniso` | **否** | `fm2dAniso`、`plot_tensor_field` 未定义 |
| 3 | `test_anisotropic_fm_old` | 各向异性 FMM（旧） | **否** | `plot_tensor_field`、`apply_colormap` 未定义 |
| 4 | `test_anisotropic_fm` | 各向异性 FMM | **否** | `plot_tensor_field` 未定义（`perform_tensor_decomp_3d` 亦缺） |
| 5 | `test_anisotropic` | 各向异性 FMM | **否** | `start_points`/`T1`/`D1` 未定义 + `plot_tensor_field` 缺失 |
| 6 | `test_bending_invariants` | `compute_bending_invariant` | 否（缺数据） | 需 `toolbox_graph_data/off/*.off` |
| 7 | `test_bug` | 3D FMM | 是（需 mex） | 3D mex `perform_front_propagation_3d` |
| 8 | `test_circular_fast_marching_2d` | `perform_circular_fast_marching_2d` | 是（需 mex+交互） | 需 circular mex 与 `ginput` |
| 9 | `test_circular_prior` | `perform_circular_fast_marching_2d`(+H) | 否（缺数据） | 需 `data/stephanodiscusniagarae.png`（仅有 `.jpg`） |
| 10 | `test_circular` | `perform_circular_fast_marching_2d` | 部分（缺数据） | 默认需 `.png` 缺失；改 `name='ellipse'` 可用 |
| 11 | `test_constrained_map` | 2D FMM(`constraint_map`) | 是（需 mex） | 合成数据 + 2D mex |
| 12 | `test_distance_approximation` | FPM 采样 + `compute_distance_landmark` | **否** | `my_eval_distance` 不可见、`get_color_from_index` 未定义 |
| 13 | `test_distance_compression` | `compute_heuristic_landmark` + FMM | 是（需 mex） | `data/road2.png` 已附带 |
| 14 | `test_eccentricity` | `compute_eccentricity_transform` | 是（需 mex） | `data/giraffe.gif` 已附带 |
| 15 | `test_eucldist` | `eucdist2` | 否（缺数据） | 需 `data/disk.png`（缺失）；`eucdist2.dll` 已附带 |
| 16 | `test_farthest_sampling_2d` | `perform_farthest_point_sampling` | **否** | 默认 `name='anisotropic'` 命中 `plot_tensor_field` 缺失 |
| 17 | `test_farthest_sampling_3d` | 3D `perform_farthest_point_sampling` | 否（缺数据） | 需 `brain1-crop-256.mat`；`perform_image_extension` 缺失（geodesic 分支） |
| 18 | `test_farthest_sampling_mesh` | `perform_farthest_point_sampling_mesh` | **否** | `perform_faces_reorientation` 未定义 + 缺 `.off` 网格 |
| 19 | `test_farthest_sampling_shape` | `perform_farthest_point_sampling` | 是（需 mex） | `data/mm.png` 已附带 |
| 20 | `test_fast_marching_2d` | 2D FMM + `compute_geodesic` | 是（需 mex+交互） | 需 `ginput`；`data/mountain.png` 已附带 |
| 21 | `test_fast_marching_3d` | 3D FMM + `compute_geodesic` | 是（需 mex） | 纯合成数据 |
| 22 | `test_fmstar_2d` | `perform_fmstar_2d` | 否（缺数据） | 默认 `name='diatom2'` 需 `data/diatom2.png`（缺失） |
| 23 | `test_fmstar_3d` | `perform_fmstar_3d` | 是（需 mex） | 默认 `type='gaussian'` 纯合成 |

**缺失依赖汇总 / Missing-dependency summary**（本章 23 个脚本实际命中者）：

| 未定义函数 / Undefined function | 命中脚本 / Scripts that fail | 备注 / Notes |
|---|---|---|
| `plot_tensor_field` | #2、#3、#4、#5、#16 | 全仓库不存在 |
| `apply_colormap` | #3、#4 | 全仓库不存在 |
| `fm2dAniso` | #2 | 仅存 `mex/anisotropic-fm-feth/fm2dAniso.cpp` 与 Mac `.mexmaci`，`compile_mex` 未编译 |
| `perform_tensor_decomp_3d` | #3、#4（仅 3D 分支） | 全仓库不存在 |
| `perform_image_extension` | #17（仅 geodesic 分支） | 全仓库不存在 |
| `perform_faces_reorientation` | #18 | 全仓库不存在 |
| `my_eval_distance` | #12 | 仅为 `perform_farthest_point_sampling(.m/_mesh.m)` 的**局部子函数**，脚本不可调用 |
| `get_color_from_index` | #12 | 全仓库不存在 |

> 任务提到的 `image_3d_resize`、`compute_voronoi_mesh`、`compute_euclidean_distance`、`matrix_sampling_get` 在本章 23 个脚本中**均未被调用**（`compute_voronoi_mesh` 仅由不在本章范围的 `test_voronoi_segmentation.m` 使用）；`image_resize` 虽有同名独立函数缺失，但本章脚本调用的是封装 `perform_image_resize`（**已附带**），不受影响。
>
> **mex 前置条件 / mex prerequisite**：`perform_front_propagation_2d`（2D FMM，含 `_slow` 纯 MATLAB 回退）、`perform_front_propagation_3d`（3D FMM，**无回退**）、`perform_circular_front_propagation_2d`、`perform_front_propagation_mesh`（GW 测地库）、`skeleton`、`eucdist2` 均由 [compile_mex.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compile_mex.m#L1-L40) 编译；仓库已附带部分预编译件（`eucdist2.dll`、`eucdist2.mexglx`、`skeleton.dll`），在对应平台上可免编译，其余需自行构建。
