# 90.7b Toolbox Fast Marching — 测试与示例 B（第三方） / Fast Marching Tests and Demos B (Vendored)

> 本章逐文件覆盖 `external/toolbox_fast_marching/tests/` 中**按字典序第 24–45 个**（共 22 个）demo/test **脚本** ——
> Gabriel Peyre 的 **Toolbox Fast Marching**（BSD，约 2004–2007）被 vendored 进 gptoolbox 的示例层。
> 与 [90.5 求解器核心](05-fast-marching-solvers.md) 的关系：**这些脚本不实现算法**，只是**构造合成数据 → 驱动求解器 → 可视化/统计**；
> 前 23 个脚本见姊妹文档 [90.7a 测试与示例 A](07a-fast-marching-tests-a.md)。
>
> This chapter covers, file by file, the **22 demo/test scripts** ranked 24–45 (alphabetical) in
> `external/toolbox_fast_marching/tests/`. These are **scripts, not functions**: they build synthetic data,
> drive the solvers documented in [90.5](05-fast-marching-solvers.md), and plot or tally results.
>
> **关键前提 / Key prerequisites**：绝大多数脚本依赖 [compile_mex.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compile_mex.m#L1-L40)
> 编译出的 mex（`perform_front_propagation_2d`/`_3d`/`_mesh`、`skeleton`、`eucdist2`）；否则 `perform_fast_marching` 在 2D 会退化到
> 纯 MATLAB 回退 [perform_front_propagation_2d_slow.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/perform_front_propagation_2d_slow.m#L1-L199)、在 3D 直接报错。
> 另有多个脚本调用**仓库中不存在**的函数（`norme`、`plot_curve`、`plot_scattered`、`compute_hausdorff_distance`、
> `get_color_from_index`、`compute_voronoi_mesh`），或使用未定义变量/缺失数据文件而**无法运行**，逐条标注如下。

- 语言 / Language：中英对照（中文为主）
- 相关章节 / See also：[90-external/README.md](README.md)、[90.5 求解器核心](05-fast-marching-solvers.md)、
  [90.7a 测试与示例 A](07a-fast-marching-tests-a.md)、[appendices/03-doc-template.md](../appendices/03-doc-template.md)

---

## 脚本总表 / Script overview

| 脚本 / Script | 驱动的求解器 / Solver driven | 一句话功能 / One-line purpose |
|---|---|---|
| `test_fmstar_error` | `perform_fast_marching`（FM\* 启发式） | 量化启发式权重 λ 对 2D 距离场误差的影响 |
| `test_fmstar_landmark` | `perform_fmstar_2d` + `compute_heuristic_landmark` | 地标启发式 FM\* 的计算量节省统计 |
| `test_fmstar_path_planing` | `perform_fmstar_3d`（3D FMM） | 带方向约束的 3D 路径规划（FM\* 权重对比） |
| `test_fmstar_weight_2d` | `perform_fmstar_2d` | FM\* 启发式权重/多分辨率下的路径误差与内存节省 |
| `test_geodesic_interpolation` | `perform_geodesic_interpolation` | 按测地距离对稀疏采样点做全图插值/上色 |
| `test_geodesic_vs_euclidean` | `perform_fast_marching`（2D） | 形状内测地距离 vs 欧氏距离着色对比 |
| `test_heuristic_mesh` | `perform_fast_marching_mesh` | 网格上 `λ·H` 启发式传播与着色 |
| `test_influence` | `perform_front_propagation_2d_slow` | 用父指针回溯 FMM 影响域（纯 MATLAB） |
| `test_landmark_error` | `perform_fast_marching` + 地标采样 | 采样策略对地标近似距离误差下降的影响 |
| `test_landmark` | `compute_heuristic_landmark` | 地标近似距离场的计算与可视化 |
| `test_multiple_paths_2d` | `perform_fast_marching`（2D） | 点到线 vs 逐点的多路径提取 |
| `test_multiple_paths_3d` | `perform_fast_marching`（3D） | 3D “花瓶”曲面多路径提取 |
| `test_path_planing` | `perform_fast_marching`（3D） | 带方向约束的 3D 路径规划（简版） |
| `test_propagation_2d` | `perform_fast_marching`（2D） | 2D 前向传播随迭代推进的动画 |
| `test_propagation_mesh` | `perform_fast_marching_mesh` | 网格上前向传播并提取多条测地线 |
| `test_propagation_shape` | `perform_fast_marching`（受约束） | 平面形状内受约束传播并提取到边界的测地线 |
| `test_redistancing` | `perform_redistancing` | 同零水平集重新初始化（插值 vs 非插值） |
| `test_segmentation` | `perform_fast_marching`（2D） | 用到达时间阈值做图像分割 |
| `test_skeleton` | `perform_fast_marching` + `skeleton` | 骨架的欧氏距离变换 vs 测地距离变换 |
| `test_vol3d` | `perform_fast_marching`（3D） | 脑部体数据可视化 + 3D 传播与路径 |
| `test_voronoi_segmentation` | `compute_voronoi_mesh` + `perform_lloyd_mesh` | 网格精确 Voronoi + Lloyd 松弛/渐进播种 |
| `test_voronoi_triangulation` | `perform_fast_marching`（2D） | 2D Voronoi 单元与三角化/边可视化 |

---

## 逐脚本详解 / Per-script reference

### `test_fmstar_error` — FM\* 启发式权重的误差标定
- **文件 / File**: [tests/test_fmstar_error.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_error.m#L1-L64)
- **类型 / Type**: 脚本（无输入/输出参数）/ script (no arguments) — 创建 `name`,`n`,`M`,`W`,`start_points`,`end_points`,`H`,`D`,`weight_list`,`err`,`D1`,`S`,`options`
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）—— 证据：[#L5](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_error.m#L5) `% Copyright (c) 2004 Gabriel Peyre`
- **被驱动的函数 / Functions exercised**: `load_potential_map`, `pick_start_end_point`, `perform_fast_marching`, `norme`（**缺失**）
- **功能 / Purpose**: `EN: Sweep the FM* heuristic weight λ, measure the L2 deviation of the resulting 2D distance field versus the reference FM distance, and plot log-log error.` / 中文：扫描 FM\* 启发式权重 λ，度量所得 2D 距离场相对参考测地距离的偏差，并画双对数误差曲线。
- **变量流动 / Variable flow**: `name='constant', n=600 →(1 load_potential_map)→ M,W →(2 pick_start_end_point)→ start_points,end_points →(3 perform_fast_marching(W,end_points)→H；W,start_points→D 参考) →(4 循环 λ∈0:0.1:1.1，perform_fast_marching(...,w*H)→D1，norme(D-D1)→err) →(5 plot 双对数误差)`
- **算法步骤 / Algorithm**:
  1. 生成 600×600 势场 `[M,W]`；交互式挑选起/终点。
  2. 用终点算启发式距离场 `H`，用起点算参考距离 `D`。
  3. 对 λ 列表逐个带启发式传播得 `D1`，累计 `err(i)=norme(D-D1)` 并 `imagesc(D-D1)`。
  4. 画 `log10(λ)` vs `log10(err)`，检查误差随 λ 单调性。
- **⚠️ 不可运行 / Not runnable**: `norme` 在仓库中**未定义**；且 [#L52](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_error.m#L52) 以**第 4 个位置参数**传启发式调用 `perform_fast_marching`，而当前签名仅 3 个参数（见 [90.5](05-fast-marching-solvers.md)），会报 “Too many input arguments”。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=600, load_potential_map"] --> B["M,W"]
    B --> C["pick_start_end_point"]
    C --> D["H=FM(W,end); D=FM(W,start)"]
    D --> E["循环 λ: FM(W,start,opt,w*H)->D1, norme"]
    E --> F["plot log10(err) vs log10(λ)"]
  ```

- **依赖 / Dependencies**: 2D FMM mex（`compile_mex.m`）；`load_potential_map`,`pick_start_end_point`；**缺失** `norme`
- **See also**: [90.5 `perform_fast_marching`](05-fast-marching-solvers.md)、`test_fmstar_2d`（[90.7a](07a-fast-marching-tests-a.md)）

### `test_fmstar_landmark` — 地标启发式 FM\* 的计算量节省
- **文件 / File**: [tests/test_fmstar_landmark.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_landmark.m#L1-L150)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=300`,`M`,`W`,`start_points`/`end_points`（25 组 cell）、`landmark`,`DL`,`front_size`,`options`
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）—— 证据：[#L5](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_landmark.m#L5) `% Copyright (c) 2004 Gabriel Peyre`
- **被驱动的函数 / Functions exercised**: `load_potential_map`, `pick_start_end_point`, `perform_fast_marching`, `compute_geodesic`, `plot_fast_marching_2d`, `compute_heuristic_landmark`, `perform_fmstar_2d`, `saveas`
- **功能 / Purpose**: `EN: Build FM* heuristics from an increasing set of landmark distance maps and measure the front-size/computation saving versus plain FM.` / 中文：用递增的地标距离场构造 FM\* 启发式，统计前沿规模与相对普通 FM 的计算节省。
- **变量流动 / Variable flow**: `n=300 →(load_potential_map 'mountain')→ M,W →(25×pick_start_end_point)→ start_points/end_points →(循环地标数: perform_fast_marching→DL) →(compute_heuristic_landmark→heuristic) →(perform_fmstar_2d→D,S；compute_geodesic→path；plot_fast_marching_2d) →(front_size→节省百分比曲线)`
- **算法步骤 / Algorithm**:
  1. 生成 300² 势场；挑 25 组起/终点并保存为 cell。
  2. 无启发式传播，记录基准前沿大小 `front_size(1)`。
  3. 地标数从 1 递增到 40，逐个算地标距离场 `DL`，`compute_heuristic_landmark` 得启发式。
  4. 对每组起点跑 `perform_fmstar_2d`，取路径、可视化（EPS/PNG），累计前沿大小。
  5. 画 “地标数 vs 计算节省(%)”。
- **⚠️ 不可运行 / Not runnable**: `perform_fmstar_2d` 内部以 4 参调用 `perform_fast_marching`（签名不符），运行时即错。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load_potential_map -> M,W"] --> B["25 组 起点/终点"]
    B --> C["基准 FM: front_size(1)"]
    C --> D["循环地标数: FM->DL -> compute_heuristic_landmark"]
    D --> E["perform_fmstar_2d -> path -> plot"]
    E --> F["节省百分比曲线"]
  ```

- **依赖 / Dependencies**: 2D mex；`compute_heuristic_landmark`,`compute_geodesic`,`plot_fast_marching_2d`；**受阻于** `perform_fmstar_2d` 参数不匹配
- **See also**: [90.5 `perform_fmstar_2d`](05-fast-marching-solvers.md)、`test_landmark_error`

### `test_fmstar_path_planing` — 带方向约束的 3D FM\* 路径规划
- **文件 / File**: [tests/test_fmstar_path_planing.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_path_planing.m#L1-L80)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `name`,`map_name`,`M`,`ntheta=41`,`obj`,`M1`,`W`,`start_points`,`end_points`,`weight_list`,`options`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件（证据：位于 `external/toolbox_fast_marching/tests/`，且调用本套件专属函数）
- **被驱动的函数 / Functions exercised**: `imread`, `rescale`, `generate_constrained_map`, `pick_start_end_point`, `perform_fast_marching`（3D）, `compute_geodesic`, `plot_constrained_path_planing`, `perform_fmstar_3d`, `plot_fast_marching_3d`
- **功能 / Purpose**: `EN: Solve constrained path planning on a room map expanded into (x,y,orientation) volume, comparing FM* heuristic weights with 3D FMM.` / 中文：把房间图扩展为 (x,y,朝向) 体，用 3D FMM/FM\* 做带方向约束的路径规划并对比不同启发式权重。
- **变量流动 / Variable flow**: `data/room1.png →(imread,rescale)→ M →(generate_constrained_map,ntheta=41,obj)→ M1 →(W=1-M1+0.001) →M,W →(pick_start_end_point + 方向角)→ start_points/end_points(3×1) →(perform_fast_marching→D；compute_geodesic→path1；plot_constrained_path_planing) →(循环权重: perform_fmstar_3d→D,path；plot_fast_marching_3d)`
- **算法步骤 / Algorithm**:
  1. 读 `data/room1.png`，反相并归一为障碍图 `M`。
  2. `generate_constrained_map` 生成朝向约束图 `M1`，权重 `W=1-M1+0.001`。
  3. 选起/终点并附方向角（第 3 维为朝向索引）。
  4. 先做一次 3D 前向传播并取最短路径、用 `plot_constrained_path_planing` 渲染。
  5. 遍历 `weight_list=[0 … 1.1]`，用 `perform_fmstar_3d` 传播、取路径、`plot_fast_marching_3d` 可视化。
- **可运行性 / Runnability**: 需 **3D mex**；`reduc_factor_list` 全为 1，故 `perform_fmstar_3d` 走 `reduc_factor==1` 分支，**规避**了缺失的 `image_3d_resize`。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["room1.png -> M"] --> B["generate_constrained_map -> M1 -> W"]
    B --> C["pick_start_end_point + theta -> start/end(3x1)"]
    C --> D["perform_fast_marching 3D -> path1 -> plot_constrained_path_planing"]
    D --> E{"循环 weight_list"}
    E --> F["perform_fmstar_3d -> path -> plot_fast_marching_3d"]
  ```

- **依赖 / Dependencies**: **3D mex**；`generate_constrained_map`,`plot_constrained_path_planing`,`perform_fmstar_3d`,`plot_fast_marching_3d`
- **See also**: [90.5 `perform_fmstar_3d`](05-fast-marching-solvers.md)、`test_path_planing`

### `test_fmstar_weight_2d` — FM\* 权重/多分辨率的误差与内存节省
- **文件 / File**: [tests/test_fmstar_weight_2d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_weight_2d.m#L1-L166)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=500`,`M`,`W`,`start_points`,`end_points`,`weight_list`,`reduc_factor_list`,`paths`,`fronts`,`err_rms`,`err_inf`
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）—— 证据：[#L5](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_fmstar_weight_2d.m#L5) `% Copyright (c) 2004 Gabriel Peyre`
- **被驱动的函数 / Functions exercised**: `load_potential_map`, `pick_start_end_point`, `perform_fmstar_2d`, `compute_geodesic`, `plot_fast_marching_2d`, `perform_curve_extraction`, `plot_curve`（**缺失**）, `compute_hausdorff_distance`（**缺失**）
- **功能 / Purpose**: `EN: Vary the FM* heuristic weight (or the multiresolution reduction factor) and quantify Hausdorff path error plus memory saving.` / 中文：改变 FM\* 启发式权重（或多分辨率缩减因子），量化路径 Hausdorff 误差与内存节省。
- **变量流动 / Variable flow**: `n=500 →(load_potential_map 'mountain')→ M,W →(pick_start_end_point)→ 起/终点 →(循环 weight/reduc_factor: perform_fmstar_2d→D,S；compute_geodesic→path；perform_curve_extraction→front) →(compute_hausdorff_distance→err_rms/err_inf→归一化→plot)`
- **算法步骤 / Algorithm**:
  1. 生成 500² 势场并挑起/终点。
  2. 对权重/缩减因子列表逐个跑 `perform_fmstar_2d`，累计 `S` 与前沿规模。
  3. 每个设置取路径 `paths{i}` 与前缘 `fronts{i}`，并归一化坐标。
  4. 画前缘/路径叠加图、水平集。
  5. 以 `paths{1}` 为基准用 `compute_hausdorff_distance` 算 RMS/INF 误差，并画内存节省曲线。
- **⚠️ 不可运行 / Not runnable**: `plot_curve` 与 `compute_hausdorff_distance` 在仓库中**均未定义**；且 `perform_fmstar_2d` 存在 4 参调用 bug。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load_potential_map -> M,W"] --> B["pick_start_end_point"]
    B --> C{"循环 weight / reduc_factor"}
    C --> D["perform_fmstar_2d -> path, front"]
    D --> E["compute_hausdorff_distance(缺失) -> err"]
    E --> F["plot 误差 / 内存节省"]
  ```

- **依赖 / Dependencies**: 2D mex；`perform_curve_extraction`,`plot_fast_marching_2d`；**缺失** `plot_curve`,`compute_hausdorff_distance`
- **See also**: [90.5 `perform_fmstar_2d`](05-fast-marching-solvers.md)、`test_fmstar_error`

### `test_geodesic_interpolation` — 测地距离插值/上色
- **文件 / File**: [tests/test_geodesic_interpolation.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_geodesic_interpolation.m#L1-L116)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `name`,`n=300`,`M`,`M0`,`W`,`points`,`f`,`sigma_list`,`alpha_list`,`A`,`G`,`CM`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `load_image`, `crop`, `rescale`, `compute_edge_energy`, `perform_geodesic_interpolation`, `plot_scattered`（**缺失**）, `clamp`, `qr`（内建）, `ginput`
- **功能 / Purpose**: `EN: Interactive colorization/interpolation: user picks sample points, geodesic distances drive a powerlaw/gaussian weighting to fill the whole image.` / 中文：交互式上色/插值：用户点选样本点，用测地距离驱动的 powerlaw/gaussian 权重填充整幅图。
- **变量流动 / Variable flow**: `name='cartoon', n=300 →(load_image, crop, rescale)→ M →(compute_edge_energy→W=1/rescale(W)) →(ginput 采点)→ points,f →(循环 σ,α: perform_geodesic_interpolation→A,G) →(clamp,imagesc 拼图)`
- **算法步骤 / Algorithm**:
  1. 读图并裁到 `n×n`，由边缘能量得传播权重 `W`。
  2. 交互 `ginput` 点选样本点并赋色 `f`（可选 RGB2COL 变换）。
  3. 对 `sigma_list`/`alpha_list` 组合调用 `perform_geodesic_interpolation` 得插值结果 `A`。
  4. 上色模式下回贴亮度分量并混合色度；`clamp` 后 `subplot` 拼图显示（含 α、σ 标题）。
- **⚠️ 不可运行 / Not runnable**: `plot_scattered` **未定义**；且 `data/` 下无名为 `cartoon` 的图像文件；流程需人工 `ginput` 交互。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load_image 'cartoon' -> crop -> M"] --> B["compute_edge_energy -> W"]
    B --> C["ginput 采点 -> points,f"]
    C --> D{"循环 sigma, alpha"}
    D --> E["perform_geodesic_interpolation -> A"]
    E --> F["clamp + imagesc 拼图"]
  ```

- **依赖 / Dependencies**: 2D mex；`load_image`,`crop`,`compute_edge_energy`,`perform_geodesic_interpolation`,`clamp`；**缺失** `plot_scattered`
- **See also**: [90.5 `perform_geodesic_interpolation`](05-fast-marching-solvers.md)

### `test_geodesic_vs_euclidean` — 测地距离 vs 欧氏距离
- **文件 / File**: [tests/test_geodesic_vs_euclidean.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_geodesic_vs_euclidean.m#L1-L71)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `name='chicken'`,`n=256`,`M`,`start_points`,`W`,`L`,`D0`,`D1`,`D0a`,`D1a`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `load_image`, `rescale`, `perform_fast_marching`, `convert_distance_color`, `imwrite`, `ginput`
- **功能 / Purpose**: `EN: Compare the Euclidean distance field with the geodesic distance inside a shape, using a constraint map to forbid propagation outside, and render both in color.` / 中文：在形状内用约束图禁止外扩，对比欧氏距离场与测地距离场，并彩色渲染两图。
- **变量流动 / Variable flow**: `name='chicken', n=256 →(load_image,rescale,>0.5)→ M 二值 →(ginput)→ start_points →(W=ones,L=-Inf/Inf 约束) →(perform_fast_marching 无约束→D0；带 L→D1) →(convert_distance_color→D0a,D1a) →(imagesc 左右并排 + imwrite)`
- **算法步骤 / Algorithm**:
  1. 读形状图并二值化（保证边界像素为 0）。
  2. `ginput` 选起点；第一次传播为无约束欧氏距离 `D0`。
  3. 设 `options.constraint_map=L`（形状外为 `-Inf`），第二次传播得测地距离 `D1`；把形状外置 `Inf`。
  4. 用 `convert_distance_color` 生成彩色图，标出中心点，左右并排显示并写 PNG。
- **可运行性 / Runnability**: 需 **2D mex**；`name='chicken'` 在 `data/` 存在（`chicken.gif`）；须人工 `ginput`。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load_image 'chicken' -> M 二值"] --> B["ginput -> start_points; W=ones, L 约束"]
    B --> C["FM 无约束 -> D0"]
    C --> D["FM 带 constraint_map -> D1"]
    D --> E["convert_distance_color -> 并排显示 / imwrite"]
  ```

- **依赖 / Dependencies**: 2D mex；`load_image`,`convert_distance_color`
- **See also**: [90.5 `perform_fast_marching`](05-fast-marching-solvers.md)、`test_propagation_shape`

### `test_heuristic_mesh` — 网格上的启发式传播
- **文件 / File**: [tests/test_heuristic_mesh.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_heuristic_mesh.m#L1-L58)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `name='skull.off'`,`vertex`,`faces`,`start_points`,`end_points`,`H`,`D`,`Q`,`options`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `read_mesh`, `perform_fast_marching_mesh`, `plot_mesh`, `plot3`（内建）
- **功能 / Purpose**: `EN: Demonstrate heuristic-driven FMM on a triangle mesh: compute the exact distance H to the goal, then use λ·H as heuristic for a forward propagation from the start, and color the mesh by distance.` / 中文：演示网格上的启发式 FMM：先算到目标的精确距离 `H`，再以 `λ·H` 为启发式从起点传播，并按距离着色网格。
- **变量流动 / Variable flow**: `'../toolbox_graph/off/skull.off' →(read_mesh)→ vertex,faces →(perform_fast_marching_mesh 到 end_points)→ H →(options.heuristic=λ·H；perform_fast_marching_mesh 从 start_points)→ D →(options.face_vertex_color=D；plot_mesh)`
- **算法步骤 / Algorithm**:
  1. 把 `../toolbox_graph/off` 加入路径并读入 `.off` 网格（脚本最终取 `skull.off`）。
  2. 计算从 `end_points` 出发的距离场 `H`（精确启发式）。
  3. 以 `options.heuristic=1*H(:)` 从 `start_points` 传播得 `D`。
  4. 用 `D` 作为顶点颜色 `plot_mesh`，标出起/终点（红色/蓝色大点），加 `camlight`。
- **⚠️ 不可运行 / Not runnable**: 依赖 `../toolbox_graph/off/*.off`（`nefertiti/beetle/fandisk/david50kf/skull`），**仓库无该目录与文件**；且需 mesh mex。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["read_mesh skull.off -> vertex,faces"] --> B["perform_fast_marching_mesh(end) -> H"]
    B --> C["options.heuristic=λH; FM_mesh(start) -> D"]
    C --> D["face_vertex_color=D -> plot_mesh + camlight"]
  ```

- **依赖 / Dependencies**: mesh mex；**缺失数据** `../toolbox_graph/off/*.off`；`read_mesh`,`plot_mesh`
- **See also**: [90.5 `perform_fast_marching_mesh`](05-fast-marching-solvers.md)、`test_propagation_mesh`

### `test_influence` — FMM 影响域的父指针回溯
- **文件 / File**: [tests/test_influence.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_influence.m#L1-L37)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=30`,`W`,`start_points`,`end_points`,`nb_iter_max`,`H`,`D`,`S`,`father`,`P`,`M`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `perform_front_propagation_2d_slow`（**纯 MATLAB 回退**）, `meshgrid`,`sub2ind`/`ind2sub`（内建）
- **功能 / Purpose**: `EN: Run the pure-MATLAB 2D fast marching, obtain the per-node father pointers, then walk them to reconstruct the set of nodes influencing a target.` / 中文：运行纯 MATLAB 版 2D 快速行进，取每点的父指针（father），据此回溯重建影响目标点的节点集合。
- **变量流动 / Variable flow**: `n=30,W=ones,start_points=[3;3],end_points=[n-3;7] →(perform_front_propagation_2d_slow)→ D,S,father →(从 end_points 出发沿 father 回溯，标记 M)→ M（影响域二值图）`
- **算法步骤 / Algorithm**:
  1. 构造 30×30 常量权重 `W` 与起/终点，另建网格坐标 `X,Y` 及到终点的欧氏距离 `H`（未使用于传播）。
  2. 调用 `perform_front_propagation_2d_slow`（无 mex 也可）得到 `D,S,father`。
  3. 从 `end_points` 起，用栈 `P` 沿 `father` 双向回溯，把访问点标入 `M`。
  4. **脚本本身不绘图**：仅计算影响域 `M`（末行即结束）。
- **可运行性 / Runnability**: **可运行**（不需要 mex，走纯 MATLAB 回退）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=30, W=ones, start/end"] --> B["perform_front_propagation_2d_slow -> D,S,father"]
    B --> C["从 end_points 沿 father 回溯 (栈 P)"]
    C --> D["M = 影响域二值图 (无绘图)"]
  ```

- **依赖 / Dependencies**: 无（纯 MATLAB）；`perform_front_propagation_2d_slow`
- **See also**: [90.5 `perform_front_propagation_2d_slow`](05-fast-marching-solvers.md)

### `test_landmark_error` — 采样策略对地标近似误差的影响
- **文件 / File**: [tests/test_landmark_error.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_landmark_error.m#L1-L247)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=100`,`M`,`W`,`start_points`,`end_points`,`base_points`,`DL_base`,`landmark`,`DL_landmark`,`err_list`,`options`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `load_potential_map`, `pick_start_end_point`, `perform_fast_marching`, `compute_geodesic`, `plot_fast_marching_2d`, `perform_farthest_point_sampling`, `perform_farthest_landmark_sampling`, `perform_farthest_point_sampling_boundary`, `compute_distance_landmark`, `perform_fmstar_2d`, `num2string_fixeddigit`, `get_color_from_index`（**缺失**）
- **功能 / Purpose**: `EN: Compare landmark-sampling strategies (rand / farthest / farthestboundary) and distance-combination methods (proxy / align) by how fast their approximate-distance error decays with the number of landmarks.` / 中文：比较地标采样策略（rand/farthest/farthestboundary）与距离合成方式（proxy/align）下，近似距离误差随地标数下降的快慢。
- **变量流动 / Variable flow**: `n=100 →(load_potential_map 'mountain')→ M,W →(起点/终点 + 200 base_points) →(每 base_point: perform_fast_marching→DL_base) →(循环 method×init: 采样地标→perform_fast_marching→DL_landmark；compute_distance_landmark→D1，与真实比→err) →(plot log-log 误差曲线)`
- **算法步骤 / Algorithm**:
  1. 生成 100² 势场，选定起/终点（部分地图用硬编码 `A`）。
  2. 随机选 200 个基准点，逐个算真实距离场 `DL_base`。
  3. 对每种“地标策略×合成方式”组合，按地标数递增采样并算 `DL_landmark`。
  4. `compute_distance_landmark` 得近似距离 `D1`，与真实距离求均方误差并累计。
  5. 部分设置下用 `perform_fmstar_2d` 出路径图；最后画误差-地标数双对数曲线与图例。
- **⚠️ 不可运行 / Not runnable**: `get_color_from_index` **未定义**（绘图阶段报错）；`compute_distance_landmark` 内部调用未定义的 `matrix_sampling_get`；`perform_fmstar_2d` 存在 4 参调用 bug。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load_potential_map -> M,W; 200 base_points"] --> B["逐个 perform_fast_marching -> DL_base"]
    B --> C{"循环 method x init"}
    C --> D["采样地标 -> FM -> DL_landmark"]
    D --> E["compute_distance_landmark -> D1 vs 真实 -> err"]
    E --> F["plot 误差曲线 (get_color_from_index 缺失)"]
  ```

- **依赖 / Dependencies**: 2D mex；`perform_farthest_point_sampling`/`_boundary`/`perform_farthest_landmark_sampling`,`compute_distance_landmark`,`num2string_fixeddigit`；**缺失** `get_color_from_index`,`matrix_sampling_get`
- **See also**: [90.5 采样族](05-fast-marching-solvers.md)、`test_landmark`,`test_fmstar_landmark`

### `test_landmark` — 地标近似距离场
- **文件 / File**: [tests/test_landmark.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_landmark.m#L1-L164)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=300`,`M`,`W`,`start_points`,`landmark`,`DL`,`D1`,`Z`,`options`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `load_potential_map`, `pick_start_end_point`, `perform_fast_marching`, `perform_farthest_point_sampling`, `perform_farthest_point_sampling_boundary`, `compute_heuristic_landmark`, `plot_scattered`（**缺失**）
- **功能 / Purpose**: `EN: Compute exact distance to a start point, build landmark distance maps, and compare the landmark-based approximate distance field against the exact one.` / 中文：算到起点的精确距离，构造地标距离场，对比“地标近似距离场”与精确距离场。
- **变量流动 / Variable flow**: `n=300 →(load_potential_map 'constant')→ M,W →(pick_start_end_point)→ start_points →(perform_fast_marching→D) →(farthest 采样→landmark；循环 perform_fast_marching→DL) →(compute_heuristic_landmark→D1) →(imagesc(W/D/D1) 三联 + plot_scattered)`
- **算法步骤 / Algorithm**:
  1. 生成 300² 势场并选起点；传播得真实距离 `D`。
  2. 依 `landmark_init`（rand/farthest/farthestboundary）选 10 个地标，逐个算距离场 `DL`。
  3. `compute_heuristic_landmark(DL,start_points)` 得近似距离 `D1`（含最近地标 `Z`）。
  4. 画三联图：势场+地标、真实距离 `D`、近似距离 `D1`。
  5. 末段 `return` 后为注释/动画代码（含 `compute_distance_landmark`、`plot_scattered`），实际不执行。
- **⚠️ 不可运行 / Not runnable**: `plot_scattered` **未定义**（第 71/129 行绘图即错）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load_potential_map -> M,W; 选起点"] --> B["perform_fast_marching -> D (真实)"]
    B --> C["farthest 采样 -> landmark; 逐个 FM -> DL"]
    C --> D["compute_heuristic_landmark -> D1 (近似)"]
    D --> E["imagesc 三联 + plot_scattered (缺失)"]
  ```

- **依赖 / Dependencies**: 2D mex；`perform_farthest_point_sampling`/`_boundary`,`compute_heuristic_landmark`；**缺失** `plot_scattered`
- **See also**: [90.5 采样族](05-fast-marching-solvers.md)、`test_landmark_error`

### `test_multiple_paths_2d` — 点到线 vs 逐点多路径
- **文件 / File**: [tests/test_multiple_paths_2d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_multiple_paths_2d.m#L1-L75)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=256`,`sigma`,`W`,`start_points`,`end_points`,`N=20`,`D`,`S`,`paths1`,`paths2`,`options`
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）—— 证据：[#L3](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_multiple_paths_2d.m#L3) `% Copyright (c) 2005 Gabriel Peyre`
- **被驱动的函数 / Functions exercised**: `perform_fast_marching`, `compute_geodesic`, `plot_fast_marching_2d`
- **功能 / Purpose**: `EN: Contrast two ways to extract many geodesics: one line-seed propagation (point-to-line) versus N separate point-to-point propagations.` / 中文：对比两种多测地线提取方式：一次“线状种子”传播（点到线）vs N 次独立的点点传播。
- **变量流动 / Variable flow**: `n=256 →(高斯 bump→W) →(起点为整条左边界线 start_points=[1:n;ones]) →(perform_fast_marching(线)→D) →(compute_geodesic(D,end_points)→paths1) →(逐点: perform_fast_marching→compute_geodesic→paths2) →(subplot 左右并排 plot_fast_marching_2d)`
- **算法步骤 / Algorithm**:
  1. 构造高斯 bump 权重 `W`（越大越慢），起点为左边界整条线。
  2. 一次传播得 `D`，用 N 个右侧终点批量 `compute_geodesic` 得 `paths1`（点到线）。
  3. 另起：把各终点当作独立起点、逐点单独传播并取路径 `paths2`（点点）。
  4. 左右并排显示两种结果对比。
- **可运行性 / Runnability**: 需 **2D mex**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["高斯 bump -> W; 起点=左边界线"] --> B["FM -> D -> compute_geodesic -> paths1"]
    A --> C["逐点 FM -> compute_geodesic -> paths2"]
    B --> D["subplot 并排 plot_fast_marching_2d"]
    C --> D
  ```

- **依赖 / Dependencies**: 2D mex；`compute_geodesic`,`plot_fast_marching_2d`
- **See also**: [90.5 `perform_fast_marching`](05-fast-marching-solvers.md)、`test_multiple_paths_3d`

### `test_multiple_paths_3d` — 3D 曲面多路径
- **文件 / File**: [tests/test_multiple_paths_3d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_multiple_paths_3d.m#L1-L82)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=61`,`X,Y,Z`,`D`,`R`,`M`,`W`,`start_points`,`end_points`,`paths1`,`paths2`,`options`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `plot_volumetric_data`, `perform_fast_marching`（3D）, `compute_geodesic`, `plot_fast_marching_3d`
- **功能 / Purpose**: `EN: Build a "vase" implicit surface in 3D, then extract multiple geodesics by one point-to-line propagation vs many point-to-point propagations.` / 中文：在 3D 生成“花瓶”隐式曲面，对比一次点到线传播与多次点点传播提取的多条测地线。
- **变量流动 / Variable flow**: `n=61 →(meshgrid)→ X,Y,Z →(D=√(X²+Y²), R=a+b·cos(πZ)) →(W=|D-R|+0.1 →取倒) →W →(表面圆环采样→ start_points/end_points) →(perform_fast_marching 3D→D,compute_geodesic→paths1) →(逐点→paths2) →(subplot plot_fast_marching_3d)`
- **算法步骤 / Algorithm**:
  1. 构造花瓶半径场 `R(z)`、到轴距离 `D`，权重 `W=1/(|D-R|+0.1)`（贴表面处慢）。
  2. `plot_volumetric_data(W)` 预览体数据。
  3. 在 z=±0.9 两处按圆周采样起点/终点并去重。
  4. 一次以全部起点传播、`compute_geodesic` 批量取路径 `paths1`；另逐点传播得 `paths2`。
  5. 左右并排 3D 可视化对比，并 `tic/toc` 报时间。
- **可运行性 / Runnability**: 需 **3D mex**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["n=61 meshgrid -> W=1/(abs(D-R)+0.1)"] --> B["plot_volumetric_data(W)"]
    B --> C["圆周采样 start/end_points"]
    C --> D["FM 3D -> compute_geodesic -> paths1"]
    C --> E["逐点 FM -> paths2"]
    D --> F["subplot plot_fast_marching_3d"]
    E --> F
  ```

- **依赖 / Dependencies**: 3D mex；`plot_volumetric_data`,`compute_geodesic`,`plot_fast_marching_3d`
- **See also**: [90.5 `perform_fast_marching`](05-fast-marching-solvers.md)、`test_multiple_paths_2d`

### `test_path_planing` — 带方向约束的 3D 路径规划（简版）
- **文件 / File**: [tests/test_path_planing.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_path_planing.m#L1-L37)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `map_name`,`M`,`n`,`ntheta=21`,`obj`,`M1`,`W`,`start_points`,`end_points`,`D`,`S`,`options`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `imread`, `rescale`, `generate_constrained_map`, `pick_start_end_point`, `perform_fast_marching`（3D）, `compute_geodesic`, `plot_constrained_path_planing`
- **功能 / Purpose**: `EN: Minimal constrained path planning: expand a floor map with an orientation axis, propagate a 3D front with the room as obstacle, and extract/plot the shortest path.` / 中文：最简版约束路径规划：带朝向轴扩展平面图，以房间为障碍做 3D 前向传播，提取并绘制最短路径。
- **变量流动 / Variable flow**: `data/room.png →(imread,rescale)→ M →(generate_constrained_map,ntheta=21,obj)→ M1 →(W=1-M1+0.001) →W →(pick_start_end_point + 随机方向角→ start/end_points 3×1) →(perform_fast_marching→D；compute_geodesic→path；plot_constrained_path_planing)`
- **算法步骤 / Algorithm**:
  1. 读 `data/room.png` 并反相归一 → 障碍图 `M`。
  2. `generate_constrained_map` 生成朝向约束体 `M1`，`W=1-M1+0.001`。
  3. 选起/终点并附加随机方向角（`start_theta/end_theta`）。
  4. 3D 前向传播得 `D`，`compute_geodesic` 取路径，把第 3 维由索引换算为角度。
  5. `plot_constrained_path_planing` 渲染规划结果。
- **可运行性 / Runnability**: 需 **3D mex**；`data/room.png` 存在。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["room.png -> M"] --> B["generate_constrained_map -> M1 -> W"]
    B --> C["pick_start_end_point + 方向角"]
    C --> D["perform_fast_marching 3D -> D -> compute_geodesic -> path"]
    D --> E["plot_constrained_path_planing"]
  ```

- **依赖 / Dependencies**: **3D mex**；`generate_constrained_map`,`plot_constrained_path_planing`
- **See also**: [90.5 `perform_fast_marching`](05-fast-marching-solvers.md)、`test_fmstar_path_planing`

### `test_propagation_2d` — 2D 传播动画
- **文件 / File**: [tests/test_propagation_2d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_propagation_2d.m#L1-L51)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=100`,`M`,`W`,`start_points`,`end_points`,`klist`,`options`,`D`,`S`,`U`,`v`
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching (BSD) —— 证据：[#L3](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_propagation_2d.m#L3) `% Copyright (c) 2005 Gabriel Peyre`
- **被驱动的函数 / Functions exercised**: `load_potential_map`, `rescale`, `pick_start_end_point`, `perform_fast_marching`, `imwrite`, `num2string_fixeddigit`, `progressbar`
- **功能 / Purpose**: `EN: Render the 2D front propagation as a sequence of snapshots obtained by capping the FM iteration count, coloring the distance field.` / 中文：通过限制 FMM 迭代数得到传播快照序列，按距离着色渲染 2D 前缘推进动画。
- **变量流动 / Variable flow**: `n=100 →(load_potential_map 'constant')→ M,W →(rescale(W,0.01,1)) →(pick_start_end_point(W)→起点/终点) →(klist) →(循环: perform_fast_marching(W,start,opt)→D,S；jet 着色→v) →(subplot/imwrite 逐帧)`
- **算法步骤 / Algorithm**:
  1. 生成 100² 势场并把 `W` 重标到 `[0.01,1]`，挑起/终点。
  2. 生成迭代数序列 `klist`（脚本最终把 `klist=Inf`，即只出一帧满传播）。
  3. 循环：带 `options.nb_iter_max=klist(i)` 传播得 `D,S`，`jet` 着色未到达区用原图填充。
  4. `subplot` 网格排布并用 `imwrite` 输出每帧 PNG。
- **可运行性 / Runnability**: 需 **2D mex**；`klist=Inf` 使循环仅一次。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load_potential_map -> M,W"] --> B["pick_start_end_point"]
    B --> C{"循环 klist (最终=Inf)"}
    C --> D["perform_fast_marching(nb_iter_max=k)->D,S -> jet 着色"]
    D --> E["subplot / imwrite 帧图"]
  ```

- **依赖 / Dependencies**: 2D mex；`load_potential_map`,`pick_start_end_point`,`progressbar`,`num2string_fixeddigit`
- **See also**: [90.5 `perform_fast_marching`](05-fast-marching-solvers.md)、`test_fast_marching_2d`（[90.7a](07a-fast-marching-tests-a.md)）

### `test_propagation_mesh` — 网格传播与多测地线
- **文件 / File**: [tests/test_propagation_mesh.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_propagation_mesh.m#L1-L74)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `name='hand'`,`vertex`,`faces`,`start_points`,`end_points`,`D`,`S`,`Q`,`paths`,`options`
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching (BSD) —— 证据：[#L3](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_propagation_mesh.m#L3) `% Copyright (c) 2007 Gabriel Peyre`
- **被驱动的函数 / Functions exercised**: `read_mesh`, `plot_mesh`, `perform_fast_marching_mesh`, `compute_vertex_ring`, `compute_edge_face_ring`, `compute_geodesic_mesh`, `plot_fast_marching_mesh`
- **功能 / Purpose**: `EN: Propagate a geodesic front over a triangle mesh from seed vertices and extract many continuous geodesics toward far vertices.` / 中文：在三角网格上从种子顶点传播测地前缘，并向远端顶点批量提取连续测地线。
- **变量流动 / Variable flow**: `'../toolbox_graph_data/off/hand' →(read_mesh)→ vertex,faces →(plot_mesh) →(perform_fast_marching_mesh(start_points)→D,S,Q) →(按 D 选远端 end_points) →(compute_geodesic_mesh(method='continuous')→paths) →(plot_fast_marching_mesh)`
- **算法步骤 / Algorithm**:
  1. 读网格（脚本最终取 `hand`），`plot_mesh` 出网格图。
  2. 选种子顶点（部分模型用硬编码索引）做前向传播得 `D,S,Q`。
  3. 取 `D` 最大的远端顶点作为终点集 `end_points`。
  4. 预计算 `options.v2v=compute_vertex_ring`、`options.e2f=compute_edge_face_ring`，以 `method='continuous'` 调 `compute_geodesic_mesh` 提取路径。
  5. `plot_fast_marching_mesh` 以 `colorfx='equalize'` 渲染距离与路径并保存 PNG。
- **⚠️ 不可运行 / Not runnable**: 依赖 `../toolbox_graph_data/off/*` 网格（`hand` 等），**仓库无此目录/数据**；且需 mesh mex。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["read_mesh hand -> vertex,faces"] --> B["perform_fast_marching_mesh(start) -> D,S,Q"]
    B --> C["按 D 取远端 end_points"]
    C --> D["compute_geodesic_mesh(continuous) -> paths"]
    D --> E["plot_fast_marching_mesh -> saveas"]
  ```

- **依赖 / Dependencies**: mesh mex；**缺失数据** `../toolbox_graph_data/off/*`；`read_mesh`,`plot_mesh`,`compute_vertex_ring`,`compute_edge_face_ring`,`compute_geodesic_mesh`,`plot_fast_marching_mesh`
- **See also**: [90.5 `perform_fast_marching_mesh`](05-fast-marching-solvers.md)、`test_heuristic_mesh`

### `test_propagation_shape` — 平面形状内受约束传播
- **文件 / File**: [tests/test_propagation_shape.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_propagation_shape.m#L1-L76)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `name='giraffe'`,`n=128`,`M`,`L`,`start_points`,`bound`,`end_points`,`D`,`paths`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `load_image`, `perform_blurring`, `perform_fast_marching`, `compute_shape_boundary`, `compute_geodesic`, `convert_distance_color`, `imageplot`
- **功能 / Purpose**: `EN: On a binary 2D shape, propagate a constrained geodesic front from a clicked point and extract geodesics to boundary sample points.` / 中文：在二值平面形状内，从手选点做受约束测地传播，并向边界采样点批量提取测地线。
- **变量流动 / Variable flow**: `name='giraffe', n=128 →(load_image,perform_blurring,>0.5)→ M 二值 →(constraint_map=L，形状外 -Inf) →(ginput 起点→ perform_fast_marching→D,S,Q) →(compute_shape_boundary→bound→end_points) →(compute_geodesic→paths) →(convert_distance_color→imageplot + 叠加路径)`
- **算法步骤 / Algorithm**:
  1. 读形状图、模糊后二值化（保证边界为 0），写 `-shape.png`。
  2. `ginput` 选形状内起点；设 `options.constraint_map=L` 限制在形状内传播。
  3. 传播得距离场 `D`；`compute_shape_boundary` 求边界并等距采样为终点集。
  4. 形状外置 `1e9` 后逐终点 `compute_geodesic` 取路径。
  5. `convert_distance_color` + `imageplot` 显示距离场，叠加黑色路径与起/终点标记。
- **可运行性 / Runnability**: 需 **2D mex**；`name='giraffe'` 在 `data/` 存在（`giraffe.gif`）；须人工 `ginput`。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load_image 'giraffe' -> M 二值"] --> B["constraint_map=L; ginput 起点"]
    B --> C["perform_fast_marching -> D"]
    C --> D["compute_shape_boundary -> end_points"]
    D --> E["compute_geodesic -> paths -> imageplot"]
  ```

- **依赖 / Dependencies**: 2D mex；`load_image`,`perform_blurring`,`compute_shape_boundary`,`convert_distance_color`,`imageplot`
- **See also**: [90.5 `perform_fast_marching`](05-fast-marching-solvers.md)、`test_geodesic_vs_euclidean`

### `test_redistancing` — 符号距离重新初始化
- **文件 / File**: [tests/test_redistancing.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_redistancing.m#L1-L34)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=256`,`name`,`D0`,`D`,`D1`,`D2`,`c0`,`c1`,`c2`,`options`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `compute_levelset_shape`, `perform_redistancing`, `perform_contour_extraction`
- **功能 / Purpose**: `EN: Take a shape level set, distort it, then redistribute to a signed distance with the same zero level set; compare interpolation vs no-interpolation and overlay the extracted contours.` / 中文：取形状水平集并加畸变，重新初始化成保持同一零集的红距离；对比插值/非插值两种模式并叠加提取的等值线。
- **变量流动 / Variable flow**: `n=256 →(compute_levelset_shape 'circlerect2'...)→ D0 →(D=(D0.^3).*(X+n/3) 畸变)→ D →(perform_redistancing(use_interpolation=1)→D1；=0→D2) →(perform_contour_extraction(D0/D1/D2,0)→ c0,c1,c2) →(plot 三色等值线叠加)`
- **算法步骤 / Algorithm**:
  1. `compute_levelset_shape` 生成基础形状水平集 `D0`。
  2. 构造畸变距离 `D`（保持零集不变）。
  3. 分别以 `use_interpolation=1/0` 调 `perform_redistancing` 得 `D1/D2`。
  4. 对 `D0/D1/D2` 各做 `perform_contour_extraction` 取零等值线。
  5. 三色叠加对比（Original / Interpolation / No interpolation）。
- **⚠️ 不可运行 / Not runnable**: [#L13](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_redistancing.m#L13) 使用变量 `X`，但 `compute_levelset_shape` 只返回 `D`（`X` 是其**内部**局部变量），脚本中 `X` **未定义** → 报错。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["compute_levelset_shape -> D0"] --> B["D=(D0^3)*(X+n/3)"]
    B --> C["perform_redistancing(interp=1) -> D1"]
    B --> D["perform_redistancing(interp=0) -> D2"]
    C --> E["perform_contour_extraction -> c1"]
    D --> F["perform_contour_extraction -> c2"]
    E --> G["三色等值线叠加"]
    F --> G
  ```

- **依赖 / Dependencies**: `compute_levelset_shape`,`perform_redistancing`,`perform_contour_extraction`；**未定义变量** `X`
- **See also**: [90.5 `perform_redistancing`](05-fast-marching-solvers.md)

### `test_segmentation` — 前向传播分割
- **文件 / File**: [tests/test_segmentation.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_segmentation.m#L1-L34)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=200`,`map_name`,`W`,`start_points`,`options`,`D`,`S`,`T`
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching (BSD) —— 证据：[#L3](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_segmentation.m#L3) `% Copyright (c) 2006 Gabriel Peyre`
- **被驱动的函数 / Functions exercised**: `imread`, `rescale`, `pick_start_end_point`, `perform_fast_marching`
- **功能 / Purpose**: `EN: Segment an image by thresholding the arrival time of a front propagated with speed derived from image intensity.` / 中文：以图像强度导出的速度场做前向传播，对到达时间取阈值实现图像分割。
- **变量流动 / Variable flow**: `data/cavern.png →(imread,rescale,^0.2)→ W →(pick_start_end_point(W)→ start_points) →(perform_fast_marching→D,S) →(T=sum(size(W))；imagesc(D>T) 分割)`
- **算法步骤 / Algorithm**:
  1. 读 `data/room.png` | `cavern.png`（脚本最终取 `cavern.png`），归一化并 `^0.2` 增强对比，背景加极小值。
  2. `pick_start_end_point` 选起点。
  3. `perform_fast_marching` 得距离场 `D`。
  4. 取阈值 `T=sum(size(W))`，`D>T` 即分割结果，左右并排显示原图与分割图。
- **可运行性 / Runnability**: 需 **2D mex**；`data/cavern.png` 存在；须人工 `ginput`。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["cavern.png -> W (rescale,^0.2)"] --> B["pick_start_end_point"]
    B --> C["perform_fast_marching -> D"]
    C --> D["imagesc(D>T) 分割"]
  ```

- **依赖 / Dependencies**: 2D mex；`pick_start_end_point`
- **See also**: [90.5 `perform_fast_marching`](05-fast-marching-solvers.md)

### `test_skeleton` — 骨架距离变换
- **文件 / File**: [tests/test_skeleton.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_skeleton.m#L1-L49)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=200`,`name='toto'`,`Ma`,`M`,`mask`,`skg`,`rad`,`sk1`,`sk2`,`Dsk1`,`Dsk2`,`Qsk`,`CM`,`options`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `load_image`, `skeleton`（**需编译 mex**）, `pick_curves`, `eucdist2`（**需编译 mex**）, `perform_fast_marching`
- **功能 / Purpose**: `EN: Extract a shape skeleton (thresholded and curve versions) and compare Euclidean distance transform to the skeleton against a constrained geodesic distance inside the shape.` / 中文：提取形状骨架（阈值版与曲线版），对比“到骨架的欧氏距离变换”与“形状内受约束的测地距离”。
- **变量流动 / Variable flow**: `name='toto', n=200 →(load_image→ sum→ 置入 n×n)→ mask →(skeleton(mask)→skg,rad；阈值→sk1；pick_curves→sk2) →(循环两种骨架: eucdist2→Dsk1；perform_fast_marching(constraint_map=CM)→Dsk2) →(subplot 三联: 骨架/欧氏/测地)`
- **算法步骤 / Algorithm**:
  1. 读 `data/toto` 形状图，置入 200×200 并取补得掩膜 `mask`。
  2. `skeleton(mask)` 得骨架与半径，阈值 `skg>20` 得 `sk1`；`pick_curves` 得 `sk2`。
  3. 对每种骨架：`eucdist2` 求欧氏距离变换 `Dsk1`；以骨架点为种子、形状外 `-Inf` 约束做 FMM 求测地距离 `Dsk2`。
  4. 每行三图展示骨架、欧氏距离、测地距离对比。
- **⚠️ 不可运行 / Not runnable**: `name` 最终为 `toto`，而 `data/` 中**无 `toto` 图像**（仅有 `mm.png`,`cavern.png`）；`skeleton` 需由 [compile_mex.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compile_mex.m#L12) 编译（仓库仅有旧 `skeleton.dll` 与 `skeleton.cpp`）；`eucdist2` 亦需 mex（有 `.dll`/`.mexglx` 与 `.m`）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load_image 'toto' -> mask"] --> B["skeleton(mask) -> sk1; pick_curves -> sk2"]
    B --> C["eucdist2 -> Dsk1 (欧氏)"]
    B --> D["perform_fast_marching(约束) -> Dsk2 (测地)"]
    C --> E["subplot 三联对比"]
    D --> E
  ```

- **依赖 / Dependencies**: `skeleton`（**mex**）、`eucdist2`（**mex**）、2D mex；**缺失数据** `data/toto`；`pick_curves`
- **See also**: [90.5 `perform_fast_marching`](05-fast-marching-solvers.md)、`test_eucldist`（[90.7a](07a-fast-marching-tests-a.md)）

### `test_vol3d` — 体数据可视化 + 3D 传播
- **文件 / File**: [tests/test_vol3d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_vol3d.m#L1-L95)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `M`,`rep`,`ilist`,`n=100`,`h`,`options`,`c`,`sigma`,`W`,`D`,`S`,`start_point`,`end_point`,`path`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `vol3d`, `crop`, `rescale`, `compute_alpha_map`, `imageplot`, `perform_fast_marching`（3D）, `compute_discrete_geodesic`（**仅子函数**）, `plot_fast_marching_3d`
- **功能 / Purpose**: `EN: Visualize a brain volume with vol3d, build an intensity-similarity speed field, run 3D fast marching, then render the front interface and the discrete geodesic path.` / 中文：用 `vol3d` 可视化脑体数据，构造基于灰度相似度的速度场，跑 3D 快速行进，再渲染前缘界面与离散测地路径。
- **变量流动 / Variable flow**: `brain1-crop-256.mat →(load)→ M →(crop(100),rescale；vol3d 渲染切片) →(ginput 起点→ 以 W=exp(-(M-c)²/2σ²) 建速度) →(perform_fast_marching 3D→D,S) →(选终点；compute_discrete_geodesic→path) →(plot_fast_marching_3d + vol3d)`
- **算法步骤 / Algorithm**:
  1. 载入 `brain1-crop-256.mat` 体数据，导出若干切片 PNG，裁剪到 `100³` 并归一化。
  2. `vol3d('cdata',M,'texture','2D')` 渲染体数据；`compute_alpha_map` 设透明度。
  3. `ginput` 选起点，以灰度相似度 `W=exp(-(M-c)²/(2σ²))`（重标 `[0.001,1]`）为速度。
  4. `perform_fast_marching` 做 3D 传播得 `D`；用 `vol3d` 显示前缘界面。
  5. 取远处最小 `D` 点为终点，`compute_discrete_geodesic` 回溯路径，`plot_fast_marching_3d` 绘制。
- **⚠️ 不可运行 / Not runnable**: [#L5](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_vol3d.m#L5) `load brain1-crop-256.mat` —— 仓库**无此 `.mat` 文件**（仅 `data/brain.png`）；且 [#L88](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_vol3d.m#L88) 调用 `compute_discrete_geodesic`，它是 [compute_geodesic.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_geodesic.m#L55) 的**子函数**，脚本**无法直接调用**。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load brain1-crop-256.mat (缺失) -> M"] --> B["crop/rescale -> vol3d 渲染"]
    B --> C["ginput 起点 -> W=exp(-(M-c)^2/2s^2)"]
    C --> D["perform_fast_marching 3D -> D,S"]
    D --> E["compute_discrete_geodesic (子函数,不可调) -> path"]
    E --> F["plot_fast_marching_3d + vol3d"]
  ```

- **依赖 / Dependencies**: 3D mex；`vol3d`,`crop`,`compute_alpha_map`,`imageplot`,`plot_fast_marching_3d`；**缺失数据** `brain1-crop-256.mat`；**不可访问** `compute_discrete_geodesic`
- **See also**: [90.5 `perform_fast_marching`](05-fast-marching-solvers.md)、`test_vol3d` 依赖 [90.3 环境与绘图辅助](03-matlab-environment-and-plotting.md)

### `test_voronoi_segmentation` — 网格 Voronoi + Lloyd 松弛
- **文件 / File**: [tests/test_voronoi_segmentation.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_voronoi_segmentation.m#L1-L69)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `test='lloyd'`,`name='elephant-50kv'`,`vertex`,`faces`,`n`,`nstart`,`niter`,`nadd`,`nmax`,`start_points`,`cm`,`Q`,`DQ`,`edges_id`,`lambda`,`options`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `read_mesh`, `compute_voronoi_mesh`（**缺失**）, `plot_fast_marching_mesh`, `perform_lloyd_mesh`, `num2string_fixeddigit`, `rgb2hsv`/`hsv2rgb`（内建）
- **功能 / Purpose**: `EN: Compute precise Voronoi cells on a mesh and drive either Lloyd relaxation or progressive seeding, visualizing the cell coloring at each iteration.` / 中文：在网格上计算精确 Voronoi 单元，驱动 Lloyd 松弛或渐进播种，并在每轮迭代可视化单元着色。
- **变量流动 / Variable flow**: `name='elephant-50kv' →(read_mesh)→ vertex,faces →(nstart 个随机种子) →(循环 niter: compute_voronoi_mesh→Q,DQ,ve,edges_id,lambda；plot_fast_marching_mesh) →(lloyd: perform_lloyd_mesh 更新种子；voronoi: 追加新点)`
- **算法步骤 / Algorithm**:
  1. 读网格（脚本最终取 `elephant-50kv`），随机初始化 `nstart` 个种子，生成配色 `cm`。
  2. 循环 `niter` 次：`compute_voronoi_mesh` 得单元 `Q`、Voronoi 边 `ve`、边集合 `edges_id` 与 `lambda`。
  3. 以 `Q(:,1)` 着色、叠加 Voronoi 边 `plot_fast_marching_mesh`，保存 PNG。
  4. `lloyd` 模式：`perform_lloyd_mesh` 更新种子位置；`voronoi` 模式：每轮追加 `nadd` 个随机点（渐增至 `nmax`）。
- **⚠️ 不可运行 / Not runnable**: `compute_voronoi_mesh` 在仓库中**未定义**（仅有 `compute_voronoi_triangulation_mesh`）；且依赖 `../toolbox_graph_data/off/*` 网格（`elephant-50kv` 等），**仓库无此数据**；需 mesh mex。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["read_mesh elephant-50kv -> vertex,faces"] --> B["随机 nstart 种子"]
    B --> C{"循环 niter"}
    C --> D["compute_voronoi_mesh (缺失) -> Q, edges_id, lambda"]
    D --> E["plot_fast_marching_mesh 着色 + Voronoi 边"]
    E --> F["lloyd: perform_lloyd_mesh 更新 / voronoi: 加点"]
    F --> C
  ```

- **依赖 / Dependencies**: mesh mex；**缺失** `compute_voronoi_mesh`；**缺失数据** `../toolbox_graph_data/off/*`；`read_mesh`,`plot_fast_marching_mesh`,`perform_lloyd_mesh`
- **See also**: [90.5 `perform_lloyd_mesh`](05-fast-marching-solvers.md)、`test_voronoi_triangulation`

### `test_voronoi_triangulation` — 2D Voronoi 三角化
- **文件 / File**: [tests/test_voronoi_triangulation.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_voronoi_triangulation.m#L1-L34)
- **类型 / Type**: 脚本（无输入/输出参数）/ script — 创建 `n=300`,`name='constant'`,`p=80`,`start_points`,`M`,`W`,`D`,`Z`,`Q`,`faces`,`edges`
- **署名 / Attribution**: 无文件头署名；属 Toolbox Fast Marching 测试套件
- **被驱动的函数 / Functions exercised**: `load_potential_map`, `perform_fast_marching`, `compute_voronoi_triangulation`, `compute_edges`, `plot_edges`, `imagesc`/`plot`（内建）
- **功能 / Purpose**: `EN: Run a multi-seed 2D fast marching to get the Voronoi decomposition Q, then build its Voronoi triangulation and draw the cell edges.` / 中文：多种子 2D 快速行进得到 Voronoi 分解 `Q`，据此构建 Voronoi 三角化并绘制单元边。
- **变量流动 / Variable flow**: `n=300,name='constant',p=80 →(随机 80 种子) →(load_potential_map→M,W) →(perform_fast_marching(W,start_points)→D,Z,Q) →(compute_voronoi_triangulation(Q,vertex)→faces；compute_edges→edges；plot_edges)`
- **算法步骤 / Algorithm**:
  1. 生成 300² 势场 `W` 与 80 个随机种子。
  2. `perform_fast_marching` 多种子传播，得距离场 `D` 与最近种子索引 `Q`。
  3. 左图显示 `Q`（Voronoi 单元）并标种子，右图显示距离场 `D`。
  4. `compute_voronoi_triangulation(Q, vertex)` 构建三角化 `faces`，`compute_edges` 取边，`plot_edges` 叠加显示。
- **⚠️ 不可运行 / Not runnable**: [#L28](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/tests/test_voronoi_triangulation.m#L28) 使用变量 `vertex`，但脚本中**从未定义** `vertex` → 报错。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["load_potential_map 'constant' -> M,W; 80 种子"] --> B["perform_fast_marching -> D,Z,Q"]
    B --> C["compute_voronoi_triangulation(Q, vertex=未定义)"]
    C --> D["compute_edges -> plot_edges"]
  ```

- **依赖 / Dependencies**: 2D mex；`load_potential_map`,`compute_voronoi_triangulation`,`compute_edges`,`plot_edges`；**未定义变量** `vertex`
- **See also**: [90.5 `perform_fast_marching`](05-fast-marching-solvers.md)、`test_voronoi_segmentation`

---

## 函数清单核对表 / Coverage Checklist

| # | 脚本 / Script | 覆盖 | 可运行性 / Runnability |
|---|---|---|---|
| 24 | `test_fmstar_error` | ✔ | ✘ 缺 `norme` + 4 参调用 |
| 25 | `test_fmstar_landmark` | ✔ | ✘ `perform_fmstar_2d` 参数不匹配 |
| 26 | `test_fmstar_path_planing` | ✔ | △ 需 3D mex（reduc_factor=1） |
| 27 | `test_fmstar_weight_2d` | ✔ | ✘ 缺 `plot_curve`/`compute_hausdorff_distance` |
| 28 | `test_geodesic_interpolation` | ✔ | ✘ 缺 `plot_scattered` + 缺图 `cartoon` |
| 29 | `test_geodesic_vs_euclidean` | ✔ | △ 需 2D mex + `ginput` |
| 30 | `test_heuristic_mesh` | ✔ | ✘ 缺网格数据 `toolbox_graph/off/*` |
| 31 | `test_influence` | ✔ | ✔ 纯 MATLAB（`_slow`） |
| 32 | `test_landmark_error` | ✔ | ✘ 缺 `get_color_from_index`/`matrix_sampling_get` |
| 33 | `test_landmark` | ✔ | ✘ 缺 `plot_scattered` |
| 34 | `test_multiple_paths_2d` | ✔ | △ 需 2D mex |
| 35 | `test_multiple_paths_3d` | ✔ | △ 需 3D mex |
| 36 | `test_path_planing` | ✔ | △ 需 3D mex |
| 37 | `test_propagation_2d` | ✔ | △ 需 2D mex |
| 38 | `test_propagation_mesh` | ✔ | ✘ 缺网格数据 `toolbox_graph_data/off/*` |
| 39 | `test_propagation_shape` | ✔ | △ 需 2D mex + `ginput` |
| 40 | `test_redistancing` | ✔ | ✘ 未定义变量 `X` |
| 41 | `test_segmentation` | ✔ | △ 需 2D mex + `ginput` |
| 42 | `test_skeleton` | ✔ | ✘ 缺数据 `data/toto` + 需 `skeleton`/`eucdist2` mex |
| 43 | `test_vol3d` | ✔ | ✘ 缺 `brain1-crop-256.mat` + 不可访问 `compute_discrete_geodesic` |
| 44 | `test_voronoi_segmentation` | ✔ | ✘ 缺 `compute_voronoi_mesh` + 缺网格数据 |
| 45 | `test_voronoi_triangulation` | ✔ | ✘ 未定义变量 `vertex` |

**统计 / Tally**：22 个脚本全部覆盖（24–45）；可直接运行（含 `_slow` 回退）**1** 个；仅需编译 mex 即可运行 **7** 个（△）；因**缺失函数/变量/数据**而不可运行 **14** 个（✘）。

**mex 编译前提 / mex prerequisite**：[compile_mex.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compile_mex.m#L1-L40) 负责编译 `perform_front_propagation_2d`/`_3d`、`perform_circular_front_propagation_2d`、`skeleton`、`eucdist2` 与 GW 版 `perform_front_propagation_mesh`；**未编译**时 2D 退化到纯 MATLAB 回退、3D 与网格直接报错。各向异性 mex 在源码中被注释为“不存在”。仓库另存有旧平台二进制 `skeleton.dll`、`eucdist2.dll`/`.mexglx`，跨平台/位数不匹配时仍应重新编译。

**缺失依赖汇总 / Missing-symbol summary**（在仓库中未定义、被本章测试引用）：`norme`、`plot_curve`、`plot_scattered`、`compute_hausdorff_distance`、`get_color_from_index`、`compute_voronoi_mesh`、`matrix_sampling_get`（经 `compute_distance_landmark` 间接命中）；另 `image_3d_resize`（被 `perform_fmstar_3d` 引用，`reduc_factor≠1` 时命中）、`image_resize`（`load_image.m` 内为子函数、`perform_image_resize.m` 文件名不匹配），以及 `compute_discrete_geodesic`（仅为 `compute_geodesic.m` 的子函数，脚本不可直接调用）。
