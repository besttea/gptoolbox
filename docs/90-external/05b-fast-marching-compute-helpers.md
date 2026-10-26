# 90.5b Toolbox Fast Marching — 派生量计算（第三方） / Fast Marching Derived Quantities (Vendored)

> 本章覆盖 `external/toolbox_fast_marching/`（Gabriel Peyre 的 *Toolbox Fast Marching*，BSD，
> 约 2007 年）中**核心求解器之上的一层派生量与辅助函数**：由距离场回溯测地路径、构造启发式/
> 势场/约束图、形状与拓扑分析、Voronoi 分割转三角化、距离场配色，以及若干微分/距离小算子。
> 这些函数**不含** Fast Marching 的主循环本身（主循环见本章姊妹篇 `05-fast-marching-solvers.md`），
> 而是"调用求解器 → 后处理"的胶水层。
>
> This chapter covers the **derived-quantity and helper layer** shipped in `external/toolbox_fast_marching/`
> (Gabriel Peyre's *Toolbox Fast Marching*, BSD, ~2007): geodesic path back-tracing from a distance
> map, heuristic/potential/constraint-map construction, shape/topology analysis, Voronoi-to-triangulation
> conversion, distance-field colouring, plus a few differential/distance mini-operators. The Fast Marching
> main loop itself lives in the sibling chapter `05-fast-marching-solvers.md`; this chapter is the glue.

- 语言 / Language：中英对照（中文为主）
- 相关章节 / See also：[90.5 核心求解器](05-fast-marching-solvers.md)、`05c-*`（绘图/拾取/显示/批处理/元数据）、
  `06-*`（`toolbox/` 子目录）、`07-*`（`tests/` 子目录）、[90.3 MATLAB 环境与绘图](03-matlab-environment-and-plotting.md)、
  [appendices/03-doc-template.md](../appendices/03-doc-template.md)
- 命名约定 / Notation：`D`/`DL` 为距离场（`n by n` 或 `n by n by p`），`W` 为速度/势场，`Q` 为
  Voronoi 索引图，`vertex`/`vertex1` 为顶点坐标（`dim by #V`，Peyre 约定列优先），`faces` 为面索引。
- 重要声明 / Disclaimer：本目录为**第三方 vendored 代码**，署名/许可由**文件头**决定（本节各条目
  "署名"一栏给出文件头证据）。发现**遮蔽 MATLAB 内置**或**与本库其它模块重叠**时逐条标注。
  部分辅助函数依赖 **Image Processing Toolbox**（`imdilate`、`smooth3`）或**预编译 MEX**（`eucdist2`）。

---

## 本章概览 / Chapter Overview

`external/toolbox_fast_marching/` 的派生量层可按用途分为四类：**路径回溯**（`compute_geodesic*`）、
**启发式/势场/约束构造**（`compute_potential` 系列的消费者如 `load_potential_map`、`compute_edge_energy`、
`compute_heuristic_*`、`generate_constrained_map`、`compute_distance_landmark`）、**形状与分割后处理**
（`compute_eccentricity_transform`、`compute_bending_invariant`、`compute_levelset_shape`、`compute_saddle_points`、
`compute_shape_boundary`、`compute_voronoi_triangulation*`）、以及**小工具/可视化**（`compute_alpha_map`、
`convert_distance_color`、`divgrad`、`eucdist2`、`callback_active_contour`）。

### 函数总表 / Function overview

| 函数 / Function | 类别 / Category | 一句话功能 / One-line purpose |
|---|---|---|
| `compute_distance_landmark` | 启发式/近似距离 | 用路标点距离场近似任意两点的测地距离（landmark heuristic） |
| `compute_geodesic` | 路径回溯 | 在 2D/3D 距离场上回溯最短测地路径 |
| `compute_geodesic_mesh` | 路径回溯 | 在三角网格距离场上回溯最短测地路径 |
| `compute_eccentricity_transform` | 形状分析 | 计算形状相对一组采样点的偏心距（eccentricity） |
| `compute_bending_invariant` | 嵌入/不变 | 用 Isomap/MDS 式脉冲距离构造弯曲不变量嵌入 |
| `compute_edge_energy` | 势场构造 | 由图像梯度构造 Fast Marching 速度场（势场） |
| `compute_heuristic_landmark` | 启发式 | 由路标距离场给出对齐式启发式地图 |
| `compute_heuristic_multiresolution` | 启发式 | 在低分辨率上解 FM 再插值成启发式地图 |
| `compute_alpha_map` | 显示辅助 | 生成 alpha 透明度图及其配色映射 |
| `compute_levelset_shape` | 合成测试形状 | 生成圆/矩形/方/多圆盘等解析水平集形状 |
| `compute_saddle_points` | 拓扑/分割 | 提取 Voronoi 分割的鞍点（双点/三点） |
| `compute_shape_boundary` | 边界提取 | 提取二值形状的 8 连通有序边界 |
| `compute_voronoi_triangulation` | 三角化 | 由 2D/3D Voronoi 索引图构造三角化 |
| `compute_voronoi_triangulation_mesh` | 三角化 | 由网格 Voronoi 索引图构造网格三角化 |
| `generate_constrained_map` | 约束/障碍图 | 由 2D 形状按方向膨胀出方向相关约束图 |
| `load_potential_map` | 数据/势场加载 | 生成或加载内置势场/高度场（含 data/ 图像） |
| `convert_distance_color` | 可视化转换 | 把距离场转成带背景的彩色图 |
| `divgrad` | 微分算子 | 计算梯度或散度（中心差分，周期/对称边界） |
| `eucdist2` | 距离变换 [MEX] | 2-D 欧氏距离变换（MATLAB 旧版 MEX，需预编译） |
| `callback_active_contour` | 迭代回调 | `perform_active_contour` 每次迭代的共轭梯度回调 |

---

## 逐函数详解 / Per-function reference

### `compute_distance_landmark` — 路标点近似距离（landmark heuristic）
- **文件 / File**: [compute_distance_landmark.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_distance_landmark.m#L1-L114)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2005 Gabriel Peyr?`。
- **签名 / Signature**: `[D1,Z] = compute_distance_landmark(start_points, DL, landmark, landmark_method);`
- **功能 / Purpose**: `EN: Approximate the geodesic distance from a start point to every pixel using precomputed distance maps to a set of landmark points (landmark-based lower-bound heuristic).` / 中文：利用到一组路标点的预计算距离图 `DL`，近似任意起点到全图的测地距离（用于 A*/FM* 的启发式下界）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `start_points` | `2 by 1` | 起点像素坐标 `[row;col]` / start pixel coordinates |
  | `DL` | `n by n by p` | 到第 `i` 个路标的距离图 / distance maps to landmark `i` |
  | `landmark` | `2 by p` 或 `p by 2` | 路标像素坐标（列优先） / landmark coordinates |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `D1` | `n by n` | 到起点的近似距离 / approximate distance to start |
  | `Z` | `n by n` | 决定该近似值的路标/配对索引 / index of the landmark |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `landmark_method` | `'align'`/`'proxy'`/`'multiproxy<k>'` | `'align'` | 近似策略；`'multiproxy0'` 被归一为 `'align'` |

- **变量流动 / Variable flow**: `start_points,DL,landmark,landmark_method →(nargin<4? 'align')→(landmark 转 2 by p)→(multiproxy<k>? 预处理 DD 与 Top-k 的 DI,I → 处理 D1=max(Da),Z=i+(j-1)*order)→(否则 switch: 'align' → T=repmat(DL(start),...) → [D1,Z]=max(abs(T-DL),[],3); 'proxy' → [V,I]=min(DL,[],3)、DD/DD1 对称化 → D1=max(DD-Dx-V,0))`
- **算法步骤 / Algorithm**:
  1. 缺省补齐 `landmark_method='align'`；把 `landmark` 统一为 `2 by p`。
  2. `align`：把起点处的距离向量广播成全图张量，逐点取 `max` 绝对差作为下界近似，并记录取到最大差的路标索引（L75–L77）。
  3. `multiproxy<k>`：先算路标两两距离 `DD`，再取前 `order` 小的距离及其索引，按三角不等式式 `D1=max(D1, DD-DI(start)-DI)` 迭代（L28–L71）。
  4. `proxy`：先做最近路标的 Voronoi 量化，按 `d(x,y)≈d(φ(x),φ(y))-d(φ(x),x)-d(φ(y),y)` 估计并取非负（L79–L99）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 start_points,DL,landmark,method"] --> B{"method 是否 multiproxy?"}
    B -- 是 --> C["DD 路标两两距离; 取前 order 距离"]
    C --> D{"i <= order ?"}
    D -- 是 --> E["D1=max(D1,Da); Z 记录配对"]
    E --> D
    D -- 否 --> Z
    B -- 否 --> F{"lower(method)"}
    F -- "'align'" --> G["D1,Z = max(abs(T-DL),[],3)"]
    F -- "'proxy'" --> H["Voronoi 量化 → D1=max(DD-Dx-V,0)"]
    G --> Z["输出 D1,Z"]
    H --> Z
  ```

- **依赖 / Dependencies**: `matrix_sampling_get`（在 `'proxy'` 分支调用，**全库未见此函数**，见下表标注）；内置 `max`/`min`/`repmat`/`reshape`/`str2num`/`find`/`zeros`/`size`。
- **See also**: `compute_heuristic_landmark`, `perform_fmstar_2d`, `perform_fmstar_landmark`
- **⚠️ 依赖缺口 / Missing dep**: `'proxy'` 分支使用的 `matrix_sampling_get` **在本仓库中不存在**，故该分支不可用；`'align'`/`'multiproxy*'` 分支不依赖它。**不遮蔽内置**（无同名内置）。

### `compute_geodesic` — 2D/3D 距离场上的测地路径回溯
- **文件 / File**: [compute_geodesic.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_geodesic.m#L1-L363)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2007 Gabriel Peyre`（`extract_path_2d/3d` 为 `Copyright (c) 2004 Gabriel Peyr?`）。
- **签名 / Signature**: `path = compute_geodesic(D, x, options)`
- **功能 / Purpose**: `EN: Extract the shortest discrete/continuous geodesic path from a point x back to the source, by gradient descent on a Fast-Marching distance map D in 2D or 3D.` / 中文：在 2D 或 3D 距离场 `D` 上，从点 `x` 沿负梯度回溯到源点（`D(y)=0`），得到最短测地路径。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `D` | `n by n` 或 `n by n by n` | 距离场 / distance map |
  | `x` | `2 by k` 或 `3 by k` | 起点（可多列 → 多条路径） / start point(s) |
  | `options` | struct | 选项（见下） / options struct |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `path` | `#P by dim` 或 `1 by k` cell | 路径点序列（多起点时为 cell） / shortest path(s) |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `options.method` | `'continuous'`/`'discrete'` | `'continuous'` | 连续梯度下降或纯离散下降 |
  | `options.trim_path` | `logical` | `1` | 是否按到源点距离阈值裁剪路径 |
  | `options.stepsize` | `double` | `0.1` | `stream2` 步长 |
  | `options.maxverts` | `int` | `10000` | `stream2` 最大顶点数 |
  | `options.start_points` | 点集 | 路径终点 | 源点集合（裁剪用） |
  | `options.Tmax` | `double` | `sum(size(A))` | 旧 ODE 分支的时间上限（当前代码未启用） |

- **变量流动 / Variable flow**: `D,x,options → method=getoptions(...,'continuous') →(method=='discrete'? path=compute_discrete_geodesic(D,x))→(x 为多列? 逐列递归得 cell)→(size(D,3)>1? extract_path_3d : extract_path_2d)→ 内部: A1(A==Inf 置 max) → grad=compute_grad(A1) 取负并归一 → stream2/stream3 追迹 → 距离阈值裁剪 → 追加 compute_discrete_geodesic 修正 → path`
- **算法步骤 / Algorithm**:
  1. 取 `method`；若为 `'discrete'` 直接走局部子函数 `compute_discrete_geodesic`（逐邻格下降，更鲁棒但更粗）（L21–L24, L55–L110）。
  2. 若 `x` 有多列（多起点），逐列递归并返回 cell（L26–L36）。
  3. 2D：把 `A==Inf` 置为有限最大值，`grad=compute_grad(A1)` 后取负并 `perform_vf_normalization` 归一；用 `stream2` 从终点顺流追迹（L156–L171）。
  4. 3D：用内置 `gradient` 求 `gy,gx,gz` 并归一，用 `stream3` 追迹（L306–L343）。
  5. 裁剪：以到 `start_points` 的距离阈值 `T=mmax(d)/300^2` 截去冗长前段（L180–L192）。
  6. 再用 `compute_discrete_geodesic` 补足离散尾段，保证端点精确落在源点（L198）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 D,x,options"] --> B{"method == 'discrete' ?"}
    B -- 是 --> B1["compute_discrete_geodesic: 邻格最速下降"]
    B1 --> Z
    B -- 否 --> C{"x 含多列 ?"}
    C -- 是 --> C1["for i: 递归 compute_geodesic → path{i}"]
    C1 --> Z
    C -- 否 --> D{"size(D,3) > 1 ?"}
    D -- 是 --> E["extract_path_3d: gradient 归一 → stream3"]
    D -- 否 --> F["extract_path_2d: compute_grad 归一 → stream2"]
    E --> G["按 start_points 距离阈值裁剪 trim_path"]
    F --> G
    G --> H["追加 compute_discrete_geodesic 尾段"]
    H --> Z["输出 path"]
  ```

- **依赖 / Dependencies**: 局部子函数 `compute_discrete_geodesic`/`extract_path_2d`/`extract_path_3d`/`get_gradient`/`event_callback`；[toolbox/compute_grad.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/compute_grad.m#L1)、[toolbox/perform_vf_normalization.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/perform_vf_normalization.m#L1)、[toolbox/mmax.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/mmax.m#L1)、[toolbox/clamp.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/clamp.m#L1)、[toolbox/getoptions.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/getoptions.m#L1)、`compute_distance_to_points`；内置 `stream2`/`stream3`/`gradient`/`find`/`repmat`。
- **See also**: `compute_geodesic_mesh`, `perform_fast_marching`, `heat_geodesic`, `dijkstra_shepard`, `exact_geodesic`
- **⚠️ 重叠 / 冲突标注**: 与 gptoolbox 自有测地实现 [`mesh/heat_geodesic`](file:///c:/VSWork/gptoolbox-1/mesh/heat_geodesic.m#L1)、[`mesh/dijkstra_shepard`](file:///c:/VSWork/gptoolbox-1/mesh/dijkstra_shepard.m#L1)、[`mex/exact_geodesic`](file:///c:/VSWork/gptoolbox-1/mex/exact_geodesic.m#L1) **用途重叠**（网格情形应优先用 `compute_geodesic_mesh`）。**不遮蔽内置**（无内置 `compute_geodesic`）。

### `compute_geodesic_mesh` — 三角网格距离场上的测地路径回溯
- **文件 / File**: [compute_geodesic_mesh.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_geodesic_mesh.m#L1-L187)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2007 Gabriel Peyre`。
- **签名 / Signature**: `[path,vlist,plist] = compute_geodesic_mesh(D, vertex, face, x, options)`
- **功能 / Purpose**: `EN: Extract a discrete geodesic on a triangle mesh from vertex x back to the source of the Fast-Marching distance D, crossing triangle interiors via local gradient descent.` / 中文：在网格距离场 `D` 上从顶点 `x` 回溯到源点，路径可穿过三角形内部（局部梯度下降求与边的交点）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `D` | `#V by 1` | 顶点距离场 / per-vertex distance |
  | `vertex` | `dim by #V` 或 `#V by dim` | 顶点坐标（自动转置） / vertex positions |
  | `face` | `3 by #F` 或 `#F by 3` | 面索引（自动转置） / face indices |
  | `x` | 标量或向量 | 起点顶点索引（多值 → 多条路径） / start vertex(es) |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `path` | `3 by #P` 或 cell | 路径三维坐标 / path positions |
  | `vlist` | `2 by #P` 或 cell | 沿途顶点对（每步跨过的边） / traversed vertex pairs |
  | `plist` | `1 by #P` 或 cell | 交点在端点间的重心参数 / barycentric parameter |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `options.method` | `'continuous'`/`'discrete'` | `'continuous'` | 内部交点下降或纯边下降 |
  | `options.verb` | `logical` | `1` | 多起点时是否显示进度条 |
  | `options.v2v` | cell | 就地计算 | 预计算的顶点邻接表 |
  | `options.e2f` | 矩阵 | 就地计算 | 预计算的边-面邻接表 |

- **变量流动 / Variable flow**: `D,vertex,face,x,options →(length(x)>1? 逐点递归 → cell; 含 progressbar)→ check_face_vertex 转置规范化 →(method=='discrete'? vlist=沿 vring 最速下降 : vertex_stepping 初始化 → while: 三角内梯度 gr=A\grad → 求交 s/t → 选 CASE1/2/3 推进 f,vlist,plist)→ path = v1.*plist + v2.*(1-plist)`
- **算法步骤 / Algorithm**:
  1. 多起点时逐点递归并（多于 5 条时）显示进度条（L18–L29）。
  2. `method=='discrete'`：以 `compute_vertex_ring` 的顶点邻接做纯边最速下降，`path=vertex(:,vlist)`（L35–L58）。
  3. 连续模式：用 `vertex_stepping` 取邻顶点中距离最小者确定初始边 `[x;w]` 与面 `f`（L75, L142–L160）。
  4. 在三角形内把距离线性化为 `gr=[a-c;b-c]` 并解 `A=( [u v]'[u v] )^{-1}` 得局部梯度，求梯度方向与边的交点参数 `etas/etat`（L82–L103）。
  5. 按 CASE1（跨 `i,k` 边到 `f1`）、CASE2（跨 `j,k` 边到 `f2`）、CASE3（无有效交点则退化为沿边 `vertex_stepping`）推进（L104–L126）。
  6. 终止条件 `Dnew==0` 或距离不再下降；最终按重心参数插值出 `path`（L127–L137）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 D,vertex,face,x,options"] --> B{"length(x) > 1 ?"}
    B -- 是 --> B1["for i: 递归 compute_geodesic_mesh → cell"]
    B1 --> Z
    B -- 否 --> C["check_face_vertex 规范化"]
    C --> D{"method == 'discrete' ?"}
    D -- 是 --> D1["沿 vring 最速下降 → vlist"]
    D1 --> Z
    D -- 否 --> E["vertex_stepping 初始化边与面 f"]
    E --> F{"循环: 求三角内梯度交点"}
    F -- "CASE1" --> F1["跨边(i,k) → f=f1"]
    F -- "CASE2" --> F2["跨边(j,k) → f=f2"]
    F -- "CASE3" --> F3["vertex_stepping 沿边推进"]
    F1 --> G{"Dnew==0 或距离不再下降 ?"}
    F2 --> G
    F3 --> G
    G -- 否 --> F
    G -- 是 --> H["path = v1.*plist + v2.*(1-plist)"]
    H --> Z["输出 path,vlist,plist"]
  ```

- **依赖 / Dependencies**: 局部子函数 `vertex_stepping`/`get_vertex_face`/`get_face_face`；[toolbox/check_face_vertex.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/check_face_vertex.m#L1)、[toolbox/compute_vertex_ring.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/compute_vertex_ring.m#L1)、[toolbox/compute_edge_face_ring.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/compute_edge_face_ring.m#L1)、[toolbox/getoptions.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/getoptions.m#L1)、[toolbox/progressbar.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/progressbar.m#L1)；内置 `setdiff`/`min`/`repmat`/`zeros`。
- **See also**: `compute_geodesic`, `perform_fast_marching_mesh`, `compute_voronoi_triangulation_mesh`
- **⚠️ 重叠 / 冲突标注**: 与 [`mesh/heat_geodesic`](file:///c:/VSWork/gptoolbox-1/mesh/heat_geodesic.m#L1)、[`mesh/dijkstra_shepard`](file:///c:/VSWork/gptoolbox-1/mesh/dijkstra_shepard.m#L1)、[`mex/exact_geodesic`](file:///c:/VSWork/gptoolbox-1/mex/exact_geodesic.m#L1) 的**网格测地路径**功能重叠（此为 FM 距离场的后处理）。**不遮蔽内置**。

### `compute_eccentricity_transform` — 形状偏心距变换
- **文件 / File**: [compute_eccentricity_transform.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_eccentricity_transform.m#L1-L127)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2006 Gabriel Peyr?`。
- **签名 / Signature**: `[E,Ind] = compute_eccentricity_transform(M, options)`
- **功能 / Purpose**: `EN: Compute the s-eccentricity of a binary shape: for each pixel, aggregate the (geodesic or Euclidean) distances to a sampled set of shape points via a power mean.` / 中文：对二值形状每个像素，聚合到一组采样点的（测地或欧氏）距离，得到 s-偏心距 `E(i)=(1/p Σ d(i,j)^s)^{1/s}`（`s=Inf` 时取最大距离）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `M` | `n by n` 或 `n by n by k` | 二值形状（内 1 外 0） / binary shape |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `E` | `n by n`（或 `n by n by k`） | 偏心距图 / eccentricity map |
  | `Ind` | `n by n` | `s==Inf` 时达到最大距离的采样点索引 / argmax sample |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `options.nb_samples` | `int` | `50` | 采样点数（超出边界点数时自动截断） |
  | `options.s` | `double` 或 `Inf` | `Inf` | 幂平均指数；`Inf` 为最大距离 |
  | `options.metric` | `'geodesic'`/`'euclidean'` | `'geodesic'` | 形状内测地距离或欧氏距离 |

- **变量流动 / Variable flow**: `M,options →(size(M,3)>1? 逐层递归)→ 读 nb_samples/s/metric → W=epsilon+M → perform_fast_marching 取中心连通分量并剔除孤立点 → L=±Inf 作 constraint_map → W=ones → 采样点(s==Inf 取外边界, 否则取形状内部) → randperm 子采样 → for i: perform_fast_marching 或 compute_euclidean_distance → d 清理 →(s==Inf? E=max(E,d) 且 Ind 记录 : E=E+d.^s)→(s~=Inf? E=(E/nb_samples).^(1/s))→ E,Ind`
- **算法步骤 / Algorithm**:
  1. 3D 输入时逐层递归（L29–L35）。
  2. 读选项；设 `nb_iter_max=Inf`、`Tmax=2*n`，先用一次 FM 传播只保留含中心的唯一连通分量（L57–L69）。
  3. 造约束图 `L(M==1)=+Inf`、`W=ones`（L71–L76）。
  4. 采样：`s==Inf` 取形状边界（`perform_convolution` 的 5 点核），否则取形状内部点，`randperm` 下采样到 `nb_samples`（L80–L97）。
  5. 对每个采样点做 FM（或欧氏距离），清理 `Inf/过大/背景` 值后按 `s` 累加 `max` 或幂和（L100–L120）。
  6. `s~=Inf` 时取幂平均；`s==Inf` 时把 `Ind` 映射回采样点索引（L121–L126）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 M,options"] --> B{"size(M,3) > 1 ?"}
    B -- 是 --> B1["逐层递归"]
    B1 --> Z
    B -- 否 --> C["FM 取中心连通分量; L=±Inf; W=ones"]
    C --> D{"s == Inf ?"}
    D -- 是 --> D1["采样点=边界; E=max(E,d); Ind 记录"]
    D -- 否 --> D2["采样点=内部; E=E+d.^s"]
    D1 --> E{"i <= nb_samples ?"}
    D2 --> E
    E -- 是 --> E1["perform_fast_marching(start_points) → d"]
    E1 --> E
    E -- 否 --> F{"s ~= Inf ?"}
    F -- 是 --> F1["E=(E/nb_samples).^(1/s)"]
    F -- 否 --> F2["Ind=samples(Ind)"]
    F1 --> Z["输出 E,Ind"]
    F2 --> Z
  ```

- **依赖 / Dependencies**: [perform_fast_marching.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/perform_fast_marching.m#L1)、[toolbox/perform_convolution.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/perform_convolution.m#L1)、[toolbox/progressbar.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/progressbar.m#L1)、`compute_euclidean_distance`（**全库未见**）；内置 `meshgrid`/`ind2sub`/`randperm`/`linspace`/`max`。
- **See also**: `perform_farthest_point_sampling`, `compute_shape_boundary`, `display_eccentricity`
- **⚠️ 依赖缺口 / Missing dep**: `options.metric='euclidean'` 分支调用的 `compute_euclidean_distance` **在本仓库中不存在**，该分支不可用；`'geodesic'`（默认）分支正常。**不遮蔽内置**。

### `compute_bending_invariant` — 弯曲不变量（Isomap/MDS 嵌入）
- **文件 / File**: [compute_bending_invariant.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_bending_invariant.m#L1-L58)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2007 Gabriel Peyre`。
- **签名 / Signature**: `vertex1 = compute_bending_invariant(vertex,faces,options)`
- **功能 / Purpose**: `EN: Compute bending-invariant coordinates of a mesh by running an Isomap-style MDS embedding on geodesic distances to a set of landmarks (farthest-point sampling), then out-of-sample interpolating to all vertices.` / 中文：先用最远点采样取路标、算测地距离矩阵，做 Isomap 式 MDS 得到 3 维嵌入，再把嵌入插值（外推）到全部顶点，得到弯曲不变量坐标。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `vertex` | `dim by #V` 或 `#V by dim` | 顶点坐标 / vertex positions |
  | `faces` | `3 by #F` 或 `#F by 3` | 面索引 / face indices |
  | `options` | struct | 选项（见下） / options |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `vertex1` | `#V by 3` | 每个顶点的弯曲不变量嵌入坐标 / bending-invariant coordinates |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `options.nlandmarks` | `int` | `200` | 路标数（自动截断到 `#V`） |

- **变量流动 / Variable flow**: `vertex,faces,options → nlandmarks=min(200,#V) → for i=1..nlandmarks: 首个随机、其余取 Dland 最小距离最大者（最远点采样） → perform_fast_marching_mesh → Dland(:,i)=d(:) → Dland=Dland.^2 → D1=Dland(landmarks,·) 对称化 → J=I-11^T/n → K=-J D1 J/2 → [xy,val]=eigs(K,3,'LM') → 归一化 xy → for x=1..nverts: vertex1(x,:)=1/2 (xy'(deltan-deltax)')'`
- **算法步骤 / Algorithm**:
  1. `nlandmarks=min(getoptions(...,200),#V)`（L13–L15）。
  2. 最远点采样选路标：首个随机，其余取"到已选路标最小距离"最大者；对每个路标跑 `perform_fast_marching_mesh` 填 `Dland`（L18–L29）。
  3. 距离平方 → 取路标子矩阵 → 对称化 → 构中心化矩阵 `J` 与内积矩阵 `K=-1/2 J D1 J`（L31–L36）。
  4. 对 `K` 做 `eigs(K,3,'LM')` 得 3 维嵌入 `xy`，按 `1/sqrt(eig)` 归一（L38–L40）。
  5. 外推插值：以 `1/2·xy'·(deltan-deltax)'` 把路标嵌入线性外推到全部顶点 `vertex1`（L43–L48）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 vertex,faces,options"] --> B["nlandmarks = min(200,#V)"]
    B --> C{"i <= nlandmarks ?"}
    C -- 是 --> C1["最远点采样取路标; perform_fast_marching_mesh → Dland(:,i)"]
    C1 --> C
    C -- 否 --> D["Dland=Dland.^2; D1=路标子矩阵对称化"]
    D --> E["K = -1/2 J D1 J"]
    E --> F["eigs(K,3,'LM') → xy 归一"]
    F --> G{"x <= #V ?"}
    G -- 是 --> G1["vertex1(x,:)=1/2 (xy'(deltan-deltax)')'"]
    G1 --> G
    G -- 否 --> Z["输出 vertex1"]
  ```

- **依赖 / Dependencies**: [perform_fast_marching_mesh.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/perform_fast_marching_mesh.m#L1)、[toolbox/getoptions.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/getoptions.m#L1)、[toolbox/progressbar.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/progressbar.m#L1)；内置 `eigs`（稀疏特征值）/`rand`/`eye`/`ones`/`mean`/`repmat`。Eigen 分解需要 `eigs`（属于 MATLAB 基础库）。
- **See also**: `perform_fast_marching_mesh`, `compute_eccentricity_transform`, `test_bending_invariants`
- **备注 / Note**: 文件末尾 `clf` 之后的绘图代码在 `return` 之后，为**不可达的遗留代码**（L52–L58）。**不遮蔽内置**。

### `compute_edge_energy` — Fast Marching 速度/势场
- **文件 / File**: [compute_edge_energy.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_edge_energy.m#L1-L76)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2005 Gabriel Peyr?`。
- **签名 / Signature**: `W = compute_edge_energy(M,s,epsi, center_point)`
- **功能 / Purpose**: `EN: Build a Fast-Marching speed function from an image: W=1/(1/(1+|grad(M*G_s)|)+epsi), high where the image gradient is strong, with optional center-based metric scaling.` / 中文：由图像构造速度场 `W`（梯度强处速度高），公式 `1/W=1/(1+|grad(M*G_s)|)+epsi`；可选按中心点距离缩放度量以避免"收缩"。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `M` | `m by p` 或 `m by p by q` | 输入图像/势场（自动 rescale） / input image |
  | `s` | `double` | 高斯平滑宽度 / smoothing width |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `W` | 同 `M` 尺寸 | 速度场（`==1/potential`） / speed function |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `s` | `double` | `3` | 平滑宽度（`nargin<2`） |
  | `epsi` | `double` | `0.05` | 正则小量（`nargin<3`） |
  | `center_point` | `[row col (dep)]` | 缺省不启用 | 提供时按到中心的距离缩放 `W`（`nargin>3`） |

- **变量流动 / Variable flow**: `M,s,epsi,center_point →(nargin<2/3 补默认)→ M=rescale(M), s=round(s) →(ndims==2: h=compute_gaussian_filter → G=conv2(M,h,'same') → grad=compute_grad(G) → G=1/(1+sqrt(sum(grad.^2,3)))+epsi → W=1/G → 边界置 e=1/(1+epsi) →(nargin>3: W=W.*d))→(ndims==3: G=smooth3(M,'box',s) → gradient → W=1/G →(nargin>3: W=W.*d))→ W`
- **算法步骤 / Algorithm**:
  1. 补默认 `s=3`、`epsi=0.05`；`M=rescale(M)`、`s=round(s)`（L17–L25）。
  2. 2D：构造高斯核 `h=compute_gaussian_filter([15 15],s/(2*size(M,1)),size(M))`（用 `conv2` 而非 `imfilter`）；平滑后求梯度，按公式得 `G` 与 `W=1/G`（L27–L36）。
  3. 2D 边界区置常量 `e=1/(1+epsi)` 以去除边界伪影（L38–L43）。
  4. 3D：用 `smooth3(M,'box',s)` 平滑 + `gradient` 求梯度，其余同理（L54–L63）。
  5. 若给出 `center_point`：乘以到中心的距离场 `d`（中心处置 `1e-9`）避免零点附近收缩（L45–L53, L65–L73）。
  6. `ndims` 既非 2 也非 3 时报错（L74–L75）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 M,s,epsi,center_point"] --> B["补默认 s=3, epsi=0.05; M=rescale(M)"]
    B --> C{"ndims(M)"}
    C -- "2" --> D["h=compute_gaussian_filter; G=conv2; grad=compute_grad"]
    C -- "3" --> E["G=smooth3(M,'box',s); gradient"]
    C -- "其它" --> F["error: 只支持 2D/3D"]
    D --> G["G=1/(1+norm(grad))+epsi; W=1/G; 边界置 e"]
    E --> G
    G --> H{"提供 center_point ?"}
    H -- 是 --> H1["W = W .* d(到中心距离)"]
    H -- 否 --> I
    H1 --> I["输出 W"]
  ```

- **依赖 / Dependencies**: [toolbox/compute_gaussian_filter.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/compute_gaussian_filter.m#L1)、[toolbox/compute_grad.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/compute_grad.m#L1)、[toolbox/rescale.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/rescale.m#L1)；内置 `conv2`/`meshgrid`/`sqrt`/`sum`。
- **See also**: `perform_fast_marching`, `load_potential_map`, `compute_grad`
- **⚠️ 工具箱依赖 / Toolbox dep**: 3D 分支使用 `smooth3`（**Image Processing Toolbox**）。**不遮蔽内置**。

### `compute_heuristic_landmark` — 路标对齐启发式
- **文件 / File**: [compute_heuristic_landmark.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_heuristic_landmark.m#L1-L14)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2005 Gabriel Peyr?`。
- **签名 / Signature**: `[H,Z] = compute_heuristic_landmark(DL,start_points)`
- **功能 / Purpose**: `EN: Build an 'align'-style lower-bound heuristic map to start_points from the stack of landmark distance maps DL (the multi-landmark, no-ordering special case of compute_distance_landmark).` / 中文：由路标距离图堆栈 `DL` 直接给出起点处的对齐式启发式地图（`compute_distance_landmark` 的简化特例）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `DL` | `n by n by p` | 到第 `i` 个路标的距离图 / landmark distance maps |
  | `start_points` | `2 by 1` | 起点坐标 `[row;col]` / start coordinates |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `H` | `n by n` | 启发式地图 / heuristic map |
  | `Z` | `n by n` | 取到最大差的路标索引 / argmax landmark index |

- **可选参数 / Optional**: 无（函数体不足 14 行，无 `options`）。

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | — | — | — | 无 / none |

- **变量流动 / Variable flow**: `DL,start_points → n=size(DL,1) → T=repmat(DL(sx,sy,:),[n n 1]) → [H,Z]=max(abs(T-DL),[],3)`
- **算法步骤 / Algorithm**:
  1. 把起点处的距离向量广播成 `n by n by p` 张量 `T`（L13）。
  2. 逐点取 `abs(T-DL)` 沿路标维的最大值作为下界 `H`，并记录取到最大的路标索引 `Z`（L14）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 DL,start_points"] --> B["T = repmat(DL(start), [n n 1])"]
    B --> C["[H,Z] = max(abs(T-DL), [], 3)"]
    C --> D["输出 H,Z"]
  ```

- **依赖 / Dependencies**: 仅内置 `repmat`/`max`/`abs`/`size`。
- **See also**: `compute_distance_landmark`, `compute_heuristic_multiresolution`, `perform_fmstar_landmark`
- **备注 / Note**: `compute_distance_landmark(...,'align')` 与它计算等价（本函数是其无 `order` 直通版）。**不遮蔽内置**。

### `compute_heuristic_multiresolution` — 多分辨率启发式
- **文件 / File**: [compute_heuristic_multiresolution.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_heuristic_multiresolution.m#L1-L51)
- **署名 / Attribution**: 无可考署名（文件头**无** Copyright 行）；同族、同风格函数均为 Gabriel Peyre / Toolbox Fast Marching（BSD）。
- **签名 / Signature**: `H = compute_heuristic_multiresolution( W, end_points, options )`
- **功能 / Purpose**: `EN: Compute a coarse-resolution Fast-Marching heuristic to end_points, then up-sample it to the full resolution (optionally with circular propagation).` / 中文：在低分辨率上对终点做一次 Fast Marching 得到粗启发式，再插值放大到原分辨率（可选圆周传播模式）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `W` | `n by n` | 全分辨率速度/势场 / full-res speed map |
  | `end_points` | `2 by 1` | 终点坐标 / end point |
  | `options` | struct | 选项（见下） / options |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `H` | `n by n` | 全分辨率启发式地图 / up-sampled heuristic map |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `options.reduc_factor` | `double in (0,1]` | `0.3` | 粗分辨率缩放因子（`1` 则不缩放） |
  | `options.propagation_type` | `'normal'`/`'circular'` | `'normal'` | 普通或圆周前进 |
  | `options.center_point` | `2 by 1` | 缺省则回退 `'normal'` | `'circular'` 模式必需的中心点 |

- **变量流动 / Variable flow**: `W,end_points,options → 读 reduc_factor(0.3)/propagation_type('normal') →(circular 缺 center_point? 警告并回退 normal)→(reduc_factor~=1? Ws=perform_image_resize(W,·factor) : Ws=W)→ clear options; options.nb_iter_max=Inf → start_points_small=round(end_points*reduc_factor) →('normal'? perform_fast_marching : perform_circular_fast_marching_2d)→(reduc_factor~=1? H=image_resize(Hs,size(W,1),size(W,2)) : H=Hs)→ H`
- **算法步骤 / Algorithm**:
  1. 读 `reduc_factor`（默认 0.3）、`propagation_type`（默认 `'normal'`）；`'circular'` 缺 `center_point` 时警告并回退 `'normal'`（L4–L22）。
  2. 若 `reduc_factor~=1`，用 `perform_image_resize` 把 `W` 缩到粗分辨率（L27–L31）。
  3. 清空 `options` 后设 `nb_iter_max=Inf`，把终点按因子缩放到粗网格（L35–L37）。
  4. `'normal'` 用 `perform_fast_marching`；`'circular'` 用 `perform_circular_fast_marching_2d`（中心点同步缩放）（L38–L43）。
  5. 把粗启发式 `Hs` 插值回全分辨率得到 `H`（L47–L51）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 W,end_points,options"] --> B{"propagation_type == 'circular' 且无 center_point ?"}
    B -- 是 --> B1["warning; 回退 'normal'"]
    B -- 否 --> C
    B1 --> C{"reduc_factor ~= 1 ?"}
    C -- 是 --> C1["Ws = perform_image_resize(W, factor)"]
    C -- 否 --> C2["Ws = W"]
    C1 --> D
    C2 --> D["clear options; nb_iter_max=Inf; 缩放 end_points"]
    D --> E{"propagation_type"}
    E -- "'normal'" --> E1["perform_fast_marching(Ws, start_small)"]
    E -- "'circular'" --> E2["perform_circular_fast_marching_2d(...)"]
    E1 --> F{"reduc_factor ~= 1 ?"}
    E2 --> F
    F -- 是 --> F1["H = image_resize(Hs, n, n)"]
    F -- 否 --> F2["H = Hs"]
    F1 --> Z["输出 H"]
    F2 --> Z
  ```

- **依赖 / Dependencies**: [toolbox/perform_image_resize.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/perform_image_resize.m#L1)、[perform_fast_marching.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/perform_fast_marching.m#L1)、[perform_circular_fast_marching_2d.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/perform_circular_fast_marching_2d.m#L1)、`image_resize`（**全库未见**）；内置 `round`/`warning`/`clear`。
- **See also**: `perform_fmstar_2d`, `compute_heuristic_landmark`, `perform_fast_marching`
- **⚠️ 依赖缺口 / Missing dep**: 放大阶段调用了 `image_resize(Hs,...)`，该函数**在本仓库中不存在**（疑似应为 `perform_image_resize` 或图像工具箱的 `imresize`），会使 `reduc_factor~=1`（默认）路径报未定义函数。**不遮蔽内置**。

### `compute_alpha_map` — alpha 透明度图 + 配色
- **文件 / File**: [compute_alpha_map.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_alpha_map.m#L1-L44)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2007 Gabriel Peyre`。
- **签名 / Signature**: `[a,c] = compute_alpha_map(aname, options)`
- **功能 / Purpose**: `EN: Build a named alpha (transparency) ramp ('rampup'/'rampdown'/'gaussian'), push it into the current figure's ALim alphamap, and derive a matching N-color colormap for display.` / 中文：按名称生成 alpha 透明度斜坡（`rampup`/`rampdown`/`gaussian`），设置当前图的 alphamap，并派生一个与之匹配的 N 色配色。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `aname` | `char` | alpha 图类型名 / alpha-map name |
  | `options` | struct | 选项（见下） / options |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `a` | `nalpha by 1` | alpha 斜坡 / alpha ramp |
  | `c` | `#c by 3` | 关联配色 / associated colormap |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `options.nalpha` | `int` | `256` | alpha 采样点数 |
  | `options.sigma` | `double` | `.08` | gaussian 的宽度 |
  | `options.center` | `double` | `.4` | gaussian 的中心 |
  | `options.cname` | `char` | `'jet'` | 基础配色名（`eval([cname '(256)'])` 调用） |

- **变量流动 / Variable flow**: `aname,options → 读 nalpha/sigma/center → x=linspace(0,1,nalpha) →(switch aname: rampup → a=x; rampdown → a=1-x; gaussian → a=exp(-(x-c).^2/(2σ²)))→ alphamap(a) 作用于当前图 → c0=eval([cname '(256)']) → si=rescale(cumsum(a)) → for i=1:3: c(:,i)=interp1(s,c0(:,i),si)→ a,c`
- **算法步骤 / Algorithm**:
  1. 读 `nalpha`(256)/`sigma`(.08)/`center`(.4)，生成采样轴 `x`（L18–L22）。
  2. 按 `aname` 计算 alpha 斜坡：`'rampup'`、`'rampdown'`、`'gaussian'`（L24–L31）。
  3. `alphamap(a)` 把斜坡写入**当前图窗**（需要已打开图窗）（L33）。
  4. 取基础配色 `c0`，用 `a` 的累积和重参数化 `si`，对每个通道 `interp1` 得到匹配配色 `c`（L35–L44）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 aname,options"] --> B["读 nalpha,sigma,center; x=linspace(0,1,nalpha)"]
    B --> C{"lower(aname)"}
    C -- "'rampup'" --> C1["a = x"]
    C -- "'rampdown'" --> C2["a = 1-x"]
    C -- "'gaussian'" --> C3["a = exp(-(x-c).^2/(2*sigma^2))"]
    C1 --> D["alphamap(a) 写入当前图窗"]
    C2 --> D
    C3 --> D
    D --> E["c0=eval(cname(256)); si=rescale(cumsum(a))"]
    E --> F["for i=1:3: c(:,i)=interp1(s,c0(:,i),si)"]
    F --> Z["输出 a,c"]
  ```

- **依赖 / Dependencies**: 内置 `linspace`/`exp`/`cumsum`/`interp1`/`alphamap`（作用于当前图窗）/`eval`；[toolbox/rescale.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/rescale.m#L1)。
- **See also**: `convert_distance_color`, `display_segmentation`, `colormap`
- **备注 / Note**: 顶部 `if nargin<2; cname='jet'; end`（L13–L15）赋值的 `cname` 随即被 `getoptions(options,'cname','jet')` 覆盖，为**无效/冗余赋值**。**不遮蔽内置**。

### `compute_levelset_shape` — 解析水平集测试形状
- **文件 / File**: [compute_levelset_shape.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_levelset_shape.m#L1-L97)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2007 Gabriel Peyre`。
- **签名 / Signature**: `D = compute_levelset_shape(name, n, options)`
- **功能 / Purpose**: `EN: Generate analytic level-set (signed distance-like) fields for standard test shapes: circle, rectangle, square, circlerect1/2, small-disks — recursively composed from the circle/rectangle primitives.` / 中文：为圆、矩形、方、圆-矩形组合、多圆盘等标准测试形状生成解析水平集（类符号距离）场，组合形状由圆/矩形原语递归合成。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `name` | `char` | 形状名 `'circle'`/`'rectangle'`/`'square'`/`'circlerect1'`/`'circlerect2'`/`'small-disks'` / shape name |
  | `n` | `int` | 网格边长（生成 `n by n`） / grid size |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `D` | `n by n` | 水平集场（内负外正，视原语而定） / level-set field |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `options.center` | `1 by 2` | `[n n]/2`（circle）/`[n/2 n/2]`（square） | 圆心 / 方中心 |
  | `options.radius` | `double` | `n/4` | 圆半径 |
  | `options.corner1` | `1 by 2` | `[0.2 0.2]*n` | 矩形一角（对角） |
  | `options.corner2` | `1 by 2` | `[0.8 0.8]*n` | 矩形另一角 |
  | `options.width` | `double` | `0.8*n` | 正方形边长 |
  | `options.nbdisks` | `int` | `5` | `'small-disks'` 每边圆盘数 |

- **变量流动 / Variable flow**: `name,n,options → [Y,X]=meshgrid(1:n,1:n) → switch lower(name): 'circle' → D=sqrt((X-cx)²+(Y-cy)²)-r; 'rectangle' → D=max(max(c1x-X,X-c2x),max(c1y-Y,Y-c2y)); 'square' → 换算 corner → 递归 'rectangle'; 'circlerect1/2' → 递归 'circle' 与 'rectangle' 的 max/min; 'small-disks' → 循环递归 'circle' 取 min; otherwise → error`
- **算法步骤 / Algorithm**:
  1. 建坐标为 `1:n` 的网格（L15）。
  2. `circle`：解析圆 `D=‖p-c‖-r`（L19–L30）。
  3. `rectangle`：两角之差的 `max` 方块距离（L31–L42）。
  4. `circlerect1`/`circlerect2`：分别以 `max(-circle, rectangle)` / `min(circle, rectangle)` 组合出"圆+矩形"形状（L43–L64）。
  5. `square`：由中心与 `width` 换算两角后递归 `'rectangle'`（L65–L78）。
  6. `small-disks`：在 `nbdisks×nbdisks` 网格点上递归 `'circle'` 取 `min`（L80–L94）。
  7. 未知名称报错（L95–L96）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 name,n,options"] --> B["[Y,X]=meshgrid(1:n,1:n)"]
    B --> C{"lower(name)"}
    C -- "'circle'" --> C1["D=norm(p-c)-radius"]
    C -- "'rectangle'" --> C2["D=max(角差)"]
    C -- "'square'" --> C3["换算 corner → 递归 'rectangle'"]
    C -- "'circlerect1/2'" --> C4["max/min(递归 'circle','rectangle')"]
    C -- "'small-disks'" --> C5["循环递归 'circle' 取 min"]
    C -- "其它" --> C6["error: Unknown shape"]
    C1 --> Z["输出 D"]
    C2 --> Z
    C3 --> Z
    C4 --> Z
    C5 --> Z
  ```

- **依赖 / Dependencies**: 自身递归调用（`'square'`/`'circlerect*'`/`'small-disks'` 内部复用 `'circle'`/`'rectangle'`）；内置 `meshgrid`/`sqrt`/`max`/`min`/`linspace`/`ones`/`Inf`。
- **See also**: `perform_fast_marching`, `test_farthest_sampling_shape`, `test_propagation_shape`
- **备注 / Note**: 无自建辅助函数依赖，纯解析。**不遮蔽内置**。

### `compute_saddle_points` — Voronoi 分割鞍点提取
- **文件 / File**: [compute_saddle_points.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_saddle_points.m#L1-L60)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2008 Gabriel Peyre`。
- **签名 / Signature**: `[vertex,faces] = compute_saddle_points(Q,D,mask)`
- **功能 / Purpose**: `EN: Detect saddle points of a Voronoi segmentation: double points (2-cell meetings) and triple points (3-cell meetings), returning them as a small graph (vertices + faces) sorted by distance.` / 中文：在 Voronoi 索引图 `Q` 上检测鞍点——先双点（两格相接）后三点（三格相接），返回小图（顶点 + 面），并按距离排序。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `Q` | `n by n` | Voronoi 索引图 / Voronoi index map |
  | `D` | `n by n` | Voronoi 距离场 / distance map |
  | `mask` | `n by n` | 有效区域掩膜（可选） / mask (optional) |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `vertex` | `2 by #N` | 鞍点坐标 / saddle point positions |
  | `faces` | `3 by #N` | 组成鞍点的 cell 索引 / incident cell indices |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `mask` | `n by n` 逻辑/数值 | 缺省不启用 | 非零为有效；为 0 处 `Q` 置 -1（第三参且非空时） |

- **变量流动 / Variable flow**: `Q,D,mask →(nargin==3 且 mask 非空? Q(mask==0)=-1)→ Q1=padarray 式补 -1 → 取每 2x2 四角 → V=sort(V,2) → d=相邻不等计数 → I=find(d>=2) → 转坐标并 clamp → 按 D(J) 降序排序 → vertex=坐标、faces=V(1:3,I) →(空? 取 D 最大点补一个)→ 面索引重排`
- **算法步骤 / Algorithm**:
  1. 若有 `mask`，把掩膜外 `Q` 置 -1（L16–L18）。
  2. 四周补 -1 得 `Q1`，抽取每个 `2x2` 邻域四角为候选四元组并排序（L20–L27）。
  3. 计数四元组中相邻不等次数 `d`；`d>=2` 视为鞍点候选（双点 `d=2`/三点 `d=3`）（L28–L31）。
  4. 由线性索引恢复坐标并 `clamp` 到有效范围（L33–L36）。
  5. 按距离 `D(J)` 降序排序候选，输出 `vertex` 与 `faces=V(1:3,I)`（L38–L48）。
  6. 若无候选，取距离最大点作为退化结果，并重排负索引面（L50–L60）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 Q,D,mask"] --> B{"nargin==3 且 mask 非空 ?"}
    B -- 是 --> B1["Q(mask==0) = -1"]
    B -- 否 --> C
    B1 --> C["Q1 补 -1; 抽 2x2 四角 → V=sort(V,2)"]
    C --> D["d = 相邻不等计数; I=find(d>=2)"]
    D --> E["恢复坐标并 clamp → 按 D(J) 降序"]
    E --> F{"候选为空 ?"}
    F -- 是 --> F1["取 D 最大点补一个"]
    F -- 否 --> G["vertex=坐标; faces=V(1:3,I)"]
    F1 --> G
    G --> H["重排负索引面"]
    H --> Z["输出 vertex,faces"]
  ```

- **依赖 / Dependencies**: [toolbox/clamp.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/clamp.m#L1)；内置 `sort`/`unique`/`ind2sub`/`find`/`zeros`/`min`/`max`。
- **See also**: `compute_voronoi_triangulation`, `perform_lloyd_mesh`, `perform_fast_marching`
- **备注 / Note**: 与 `tests/test_voronoi_triangulation.m` 相关。**不遮蔽内置**。

### `compute_shape_boundary` — 二值形状 8 连通有序边界
- **文件 / File**: [compute_shape_boundary.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_shape_boundary.m#L1-L88)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2007 Gabriel Peyre`。
- **签名 / Signature**: `bound = compute_shape_boundary(M)`
- **功能 / Purpose**: `EN: Extract the ordered (8-connected) boundary pixels of a binary shape, both after keeping only the connected component containing the shape's centroid.` / 中文：提取二值形状的 8 连通**有序**边界点（先只保留含形状中心的连通分量）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `M` | `n by n` | 二值形状 / binary shape |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `bound` | `#N by 2` | 有序边界点坐标 / ordered boundary pixels |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | — | — | — | 无 `options`（前向兼容说明见下） |

- **变量流动 / Variable flow**: `M → W=epsilon+M → perform_fast_marching 取含中心连通分量、剔除孤立点 → perform_convolution(ones(3)) 找内部边界 A=Mh>0 且 Mh<9 且 M==1 → 取首个边界点起（A 计数）→ while sum(A(:))>0: 按 8 邻域 wx/wy 找最近未访问点推进，找不到则取全局最近点 → 记录 boundx/boundy → bound=[boundx boundy] → 按叉积 z 判向（可翻转）`
- **算法步骤 / Algorithm**:
  1. 先用 FM 把形状限制为含中心（`X²+Y²` 最小点）的唯一连通分量（L15–L26）。
  2. 用 `ones(3)` 卷积得邻域计数 `Mh`，取"内部边界"`Mh>0 && Mh<9 && M==1` 作为边界候选 `A`（L34–L39）。
  3. 从首个边界点起，按 8 邻域方向表 `wx/wy` 依次找最近未访问边界点推进（L42–L62）。
  4. 若 8 邻域都无候选，则退化为取全局最近边界点（L64–L71）。
  5. 以 `A(x,y)=A(x,y)-1` 计数避免重复，累加到 `bound`（L72–L76）。
  6. 用叉积符号判断绕向并按需翻转（`end:-1:1`），使曲线方向一致（L79–L88）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 M"] --> B["FM: 保留含中心连通分量"]
    B --> C["perform_convolution(ones(3)) → 边界 A"]
    C --> D["取首个边界点; A(x,y)=0"]
    D --> E{"sum(A(:)) > 0 ?"}
    E -- 是 --> E1["扫描 8 邻域找最近未访问点; 找不到则取全局最近点"]
    E1 --> E2["A(x,y)=A(x,y)-1; 记录 bound"]
    E2 --> E
    E -- 否 --> F["按叉积符号判向, 必要时翻转"]
    F --> Z["输出 bound"]
  ```

- **依赖 / Dependencies**: [perform_fast_marching.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/perform_fast_marching.m#L1)、[toolbox/perform_convolution.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/perform_convolution.m#L1)；内置 `meshgrid`/`ind2sub`/`mean`/`cat`/`sum`/`min`/`max`/`find`。
- **See also**: `compute_eccentricity_transform`, `perform_farthest_point_sampling_boundary`, `perform_contour_extraction`
- **⚠️ 备注 / Note**: 末尾绕向判断写为 `if sum(z<0)>sum(z<0)`，两侧恒等，**该翻转分支永不触发**（L85–L88，源码疑似笔误，应为 `sum(z>0)>sum(z<0)`）。**不遮蔽内置**。

### `compute_voronoi_triangulation` — 2D/3D Voronoi → 三角化
- **文件 / File**: [compute_voronoi_triangulation.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_voronoi_triangulation.m#L1-L124)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2008 Gabriel Peyre`。
- **签名 / Signature**: `faces = compute_voronoi_triangulation(Q, vertex)`
- **功能 / Purpose**: `EN: Convert a Fast-Marching Voronoi partition map into a triangulation: split 4-fold (2D) / degenerate (3D) Voronoi cells into triangles around their centroid, keeping non-degenerate triangles.` / 中文：把 Fast Marching 得到的 Voronoi 划分图转成三角化：对 4 重（2D）/退化（3D）单元按质心拆分三角形，保留非退化三角形。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `Q` | `n by n` 或 `n by n by n` | Voronoi 索引图 / Voronoi index map |
  | `vertex` | `dim by #V` | 种子点坐标（用以定拆序） / vertex positions |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `faces` | `3 by #F`（2D）/ `dim+1 by #F`（3D） | 三角/四面单元索引 / element indices |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | — | — | — | 无 `options`（维度由 `nb_dims(Q)` 决定） |

- **变量流动 / Variable flow**: `Q,vertex → d=nb_dims(Q) →(d==2? compute_voronoi_triangulation_2d; d==3? ..._3d; 否则 error)→ 2D: 抽每 2x2 四角 → unique prod>0 → d=不等计数 → d==3 单元按质心 atan2 排序拆 2 三角、d==2 保留单三角 → faces 转置；3D: 抽每 2x2x2 八角 → unique → d==4 五重按质心拆 2 三角、d==3 唯一化保留 → faces`
- **算法步骤 / Algorithm**:
  1. 由 `nb_dims(Q)` 判断 2D/3D，否则报错（L15–L23）。
  2. 2D：抽取每个 `2x2` 邻域四角，`sort` 后 `unique`，去掉含 0 的退化（`prod>0`）（L84–L91）。
  3. 2D：`d==3` 的 4 重单元以质心为原点按 `atan2` 排序拆成两个三角；`d==2` 的保留为单三角；`d==3` 多于 1 个时警告（L94–L121）。
  4. 3D：抽取每个 `2x2x2` 八角，`unique` 后按相似方式处理 `d==4`（五重）与 `d==3`（保留），返回四面体索引（L35–L74）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 Q,vertex"] --> B["d = nb_dims(Q)"]
    B --> C{"d"}
    C -- "2" --> D["compute_voronoi_triangulation_2d"]
    C -- "3" --> E["compute_voronoi_triangulation_3d"]
    C -- "其它" --> F["error: 仅支持 2D/3D"]
    D --> G["抽取邻域角 → unique → 按 d 拆分/保留"]
    E --> H["抽取邻域角 → unique → 按 d 拆分/保留"]
    G --> Z["输出 faces"]
    H --> Z
  ```

- **依赖 / Dependencies**: [toolbox/nb_dims.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/nb_dims.m#L1)；局部子函数 `compute_voronoi_triangulation_2d`/`_3d`；内置 `meshgrid`/`sort`/`unique`/`atan2`/`prod`/`warning`。
- **See also**: `perform_fast_marching`, `compute_voronoi_triangulation_mesh`, `compute_saddle_points`
- **备注 / Note**: `vertex` 仅用于拆分时的绕向排序。**不遮蔽内置**。

### `compute_voronoi_triangulation_mesh` — 网格 Voronoi → 三角化
- **文件 / File**: [compute_voronoi_triangulation_mesh.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_voronoi_triangulation_mesh.m#L1-L34)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2006 Gabriel Peyre`。
- **签名 / Signature**: `[vertex_voronoi,faces_voronoi] = compute_voronoi_triangulation_mesh(Q, vertex, faces)`
- **功能 / Purpose**: `EN: Given a per-vertex Voronoi label map Q on a mesh, build the coarser 'Voronoi mesh' whose vertices are the seeds actually touched and whose faces are the original triangles spanning exactly two labels.` / 中文：由网格上的逐顶点 Voronoi 标签图 `Q`，构造一个更粗的"Voronoi 网格"：顶点为被触及的种子，面为恰好跨越两种标签的原始三角形。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `Q` | `#V by 1` | 逐顶点 Voronoi 标签 / per-vertex Voronoi label |
  | `vertex` | `dim by #V` 或 `#V by dim` | 顶点坐标（自动转置） / vertex positions |
  | `faces` | `3 by #F` 或 `#F by 3` | 面索引（自动转置） / face indices |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `vertex_voronoi` | `dim by #S` | 被触及的种子顶点坐标 / touched-seed positions |
  | `faces_voronoi` | `3 by #Fv` | 跨两标签的三角形（索引重映射到 `vertex_voronoi`） / reindexed faces |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | — | — | — | 无 `options` |

- **变量流动 / Variable flow**: `Q,vertex,faces →(转置统一为 dim by #V / 3 by #F)→ V=Q(faces)（3 by #F 标签）→ V=sort(V,1) → V=unique(V','rows')' → d=(V(1,:)~=V(2,:))+(V(2,:)~=V(3,:)) → I=find(d==2)（跨两标签的面）→ w=sort(unique(V(:,I)))、z 做索引重映射 → faces_voronoi=z(V(:,I)); vertex_voronoi=vertex(:,w)`
- **算法步骤 / Algorithm**:
  1. 统一 `vertex`/`faces` 方向（`dim by #V` / `3 by #F`）（L12–L17）。
  2. 取每面三顶点的 Voronoi 标签并排序，`unique` 去重（L19–L21）。
  3. `d` 统计三标签中相邻不等的次数；`d==2` 即"跨恰好两种标签"的三角形（L24）。
  4. 收集这些面触及的种子索引 `w`，建立旧→新索引映射 `z`（L26–L31）。
  5. 输出重映射后的 `faces_voronoi` 与对应 `vertex_voronoi`（L33–L34）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 Q,vertex,faces"] --> B["统一方向; V=Q(faces)"]
    B --> C["V=sort(V,1); V=unique(V','rows')'"]
    C --> D["d = 相邻标签不等计数; I=find(d==2)"]
    D --> E["w=被触及种子; z 建索引映射"]
    E --> F["faces_voronoi=z(V(:,I)); vertex_voronoi=vertex(:,w)"]
    F --> Z["输出 vertex_voronoi,faces_voronoi"]
  ```

- **依赖 / Dependencies**: 仅内置 `sort`/`unique`/`find`/`zeros`/`size`。
- **See also**: `perform_fast_marching_mesh`, `compute_voronoi_triangulation`, `perform_lloyd_mesh`
- **备注 / Note**: 与 `tests/test_voronoi_triangulation.m` 相关；用于 Lloyd 迭代的重网格化。**不遮蔽内置**。

### `generate_constrained_map` — 方向相关约束/障碍图
- **文件 / File**: [generate_constrained_map.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/generate_constrained_map.m#L1-L43)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2004 Gabriel Peyr?`。
- **签名 / Signature**: `M1 = generate_constrained_map(M,nb_theta,obj,theta_max)`
- **功能 / Purpose**: `EN: Build a 3-D constraint map from a 2-D binary map by dilating the obstacles with a rotating rectangle kernel, one slice per orientation theta.` / 中文：把 2D 二值地图按一系列旋转角度，用矩形结构元膨胀障碍，得到 3D 约束图（每层对应一个朝向 `theta`）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `M` | `n by p` | 2D 二值地图（0 背景 / 1 物体） / binary map |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `M1` | `n by p by nb_theta` | 每层对应朝向 `theta=(t-1)*2*pi/nb_theta` 的约束图 / per-orientation maps |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `nb_theta` | `int` | `10` | 朝向数（`nargin<2`） |
  | `obj` | `1 by 2` | `[0.2,0.05]` | 运动矩形尺寸（`nargin<3`） |
  | `theta_max` | `double` | `2*pi` | 最大角度（`nargin<4`） |

- **变量流动 / Variable flow**: `M,nb_theta,obj,theta_max →(nargin<2/3/4 补默认)→ M1=zeros(n,p,nb_theta) → nw=round((norm(obj)*n+4-1)/2)*2+1; x=linspace → [Y,X]=meshgrid(x,x) → for t=1:nb_theta: theta=theta_max*(t-1)/nb_theta → 旋转 Xr,Yr → SE=(|Xr|<obj1/2).*(|Yr|<obj2/2) → M1(:,:,t)=imdilate(M,SE)`
- **算法步骤 / Algorithm**:
  1. 补默认 `nb_theta=10`、`obj=[0.2,0.05]`、`theta_max=2*pi`（L15–L25）。
  2. 由 `obj` 的范数确定结构元尺寸 `nw`（取奇整数），生成其坐标网格 `X,Y`（L30–L35）。
  3. 对每个朝向 `t`：把网格旋转 `theta`，用矩形指示函数 `SE=(|Xr|<obj(1)/2).*(|Yr|<obj(2)/2)` 作为旋转矩形结构元（L36–L41）。
  4. 用 `imdilate(M,SE)` 膨胀障碍并写入 `M1(:,:,t)`（L42）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 M,nb_theta,obj,theta_max"] --> B["补默认; M1=zeros(n,p,nb_theta)"]
    B --> C["nw 定结构元尺寸; [Y,X]=meshgrid"]
    C --> D{"t <= nb_theta ?"}
    D -- 是 --> E["theta=theta_max*(t-1)/nb_theta; 旋转 → SE"]
    E --> F["M1(:,:,t) = imdilate(M,SE)"]
    F --> D
    D -- 否 --> Z["输出 M1"]
  ```

- **依赖 / Dependencies**: 内置 `zeros`/`meshgrid`/`linspace`/`norm`/`cos`/`sin`/`round`/`max`/`abs`。
- **See also**: `perform_fmstar_2d`, `test_constrained_map`, `perform_fast_marching`
- **⚠️ 工具箱依赖 / Toolbox dep**: `imdilate` 属 **Image Processing Toolbox**（无该工具箱则报错）。**不遮蔽内置**。

### `load_potential_map` — 生成/加载内置势场
- **文件 / File**: [load_potential_map.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/load_potential_map.m#L1-L69)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2004 Gabriel Peyr?`。
- **签名 / Signature**: `[M,W] = load_potential_map(name, n, options)`
- **功能 / Purpose**: `EN: Produce a display image M and a Fast-Marching potential/speed map W for a named test scene (gaussian, constant, peaks, mountain, road2, cavern, binary), reading bundled PNGs from data/ when needed, then resizing to n-by-n.` / 中文：为具名测试场景（`gaussian`/`constant`/`peaks`/`mountain`/`road2`/`cavern`/`binary`）生成显示图 `M` 与势场 `W`，需要时读取 `data/` 下的 PNG，最后缩放到 `n` 见方。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `name` | `char` | 场景名 / scene name |
  | `n` | `int` | 目标边长 / target size |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `M` | `n by n` | 显示用图像 / display image |
  | `W` | `n by n` | 势场/速度图 / potential map |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `options` | struct | — | **声明但未使用**（仅 `options.null=0`） / declared but unused |

- **变量流动 / Variable flow**: `name,n,options → file_name=['data/',name,'.png'] → switch name: gaussian/constant/peaks/mountain/road2/cavern/binary 各构造 M,W; otherwise → M=rescale(imread(file_name)); W=compute_edge_energy(M,5,0.05) → dr=[min(W(:)) max(W(:))] → m=n/max(size(M)) → M/W = perform_image_resize(...,m) → W=rescale(W,dr(1),dr(2)) → M,W`
- **算法步骤 / Algorithm**:
  1. 组装数据文件路径 `data/<name>.png`（L14）。
  2. 按 `name` 分支构造 `M`/`W`：`'gaussian'` 解析高斯、`'constant'`/`'peaks'`/`'binary'` 解析常量场、`'mountain'`/`'cavern'`/`'road2'` 读 PNG 后按阈值/边缘处理（L16–L58）。
  3. `otherwise`：读图后直接用 `compute_edge_energy(M,5,0.05)` 作为势场（L55–L57）。
  4. 记录动态范围 `dr`，把 `M`/`W` 等比缩放到 `n` 见方，再按 `dr` 重标定 `W`（L60–L68）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 name,n"] --> B["file_name = data/name.png"]
    B --> C{"switch name"}
    C -- "gaussian/constant/peaks/binary" --> C1["解析构造 M,W"]
    C -- "mountain/road2/cavern" --> C2["imread(file_name) 后处理"]
    C -- "otherwise" --> C3["imread; W=compute_edge_energy(M,5,0.05)"]
    C1 --> D["记 dr; 缩放 M,W 到 n by n; rescale W"]
    C2 --> D
    C3 --> D
    D --> Z["输出 M,W"]
  ```

- **依赖 / Dependencies**: [toolbox/perform_image_resize.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/perform_image_resize.m#L1)、[toolbox/rescale.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/rescale.m#L1)、`compute_edge_energy`；内置 `peaks`/`imread`/`meshgrid`/`exp`/`ones`/`min`/`max`；数据 `data/*.png`。
- **See also**: `compute_edge_energy`, `perform_fast_marching`, `test_propagation_2d`
- **备注 / Note**: 需 `data/` 目录存在（随库附带）。`peaks` 分支在 `switch` 中重复出现（后一个覆盖，无实际影响）。**不遮蔽内置**。

### `convert_distance_color` — 距离场转彩色图
- **文件 / File**: [convert_distance_color.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/convert_distance_color.m#L1-L25)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2007 Gabriel Peyre`。
- **签名 / Signature**: `v = convert_distance_color(D,M)`
- **功能 / Purpose**: `EN: Render a distance map into an RGB image using jet, blending the background image M into pixels where the distance is Inf.` / 中文：把距离场用 `jet` 配色渲成 RGB 图，并在 `D==Inf` 处混入背景图 `M`。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `D` | `n by n` | 距离场 / distance map |
  | `M` | `n by n` | 背景图（可选） / background (optional) |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `v` | `n by n by 3` | 彩色距离图 / RGB image |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `M` | `n by n` | `ones(n)` | 背景灰度图（`nargin<2` 时全白） |

- **变量流动 / Variable flow**: `D,M → n=size(D,1) →(nargin<2? M=ones(n))→ c=jet(256) → U=D; U(U==Inf)=min(U(U~=Inf)) → I=floor(255*rescale(U))+1 → v=c(I(:),:); v=reshape([n n 3]) → A=repmat(rescale(M),[1 1 3]); B=repmat(D,[1 1 3]) → v(B==Inf)=A(B==Inf) → v`
- **算法步骤 / Algorithm**:
  1. `nargin<2` 时背景置全 1（L15–L17）。
  2. 取 `jet(256)` 配色；把 `Inf` 距离替换为最小有限距离以便索引（L19–L20）。
  3. 归一化后量化到 1..256 的色表索引，映射并重排为 `n by n by 3`（L21–L22）。
  4. 构造背景 `A` 与掩膜 `B`，在 `D==Inf` 处用背景色替换（L23–L25）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 D,M"] --> B{"nargin < 2 ?"}
    B -- 是 --> B1["M = ones(n)"]
    B -- 否 --> C
    B1 --> C["c = jet(256); U(u==Inf)=min finite; I 量化"]
    C --> D["v = c(I(:),:) 重排为 n by n by 3"]
    D --> E["A=背景; B=D; v(B==Inf)=A(B==Inf)"]
    E --> Z["输出 v"]
  ```

- **依赖 / Dependencies**: 内置 `jet`/`repmat`/`floor`/`reshape`；[toolbox/rescale.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/rescale.m#L1)。
- **See also**: `plot_fast_marching_2d`, `compute_alpha_map`, `display_segmentation`, `turbo`, `viridis`
- **⚠️ 重叠 / 冲突标注**: 配色职责与 gptoolbox 的 [`utility/turbo`](file:///c:/VSWork/gptoolbox-1/utility/turbo.m#L1) 及 [external/viridis.m](file:///c:/VSWork/gptoolbox-1/external/viridis.m#L1) 重叠（本函数固定用旧式 `jet`，不感知新版感知均匀配色）。**不遮蔽内置**（无内置 `convert_distance_color`）。

### `divgrad` — 梯度/散度算子（中心差分）
- **文件 / File**: [divgrad.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/divgrad.m#L1-L68)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copyright (c) 2007 Gabriel Peyre`。
- **签名 / Signature**: `G = divgrad(M,options)`
- **功能 / Purpose**: `EN: One entry point for both operators: divergence of a 2-channel vector field, or gradient of a scalar field, using centered finite differences with symmetric or periodic boundary.` / 中文：统一入口计算两种算子——对 2 通道向量场求散度、对标量场求梯度，采用中心差分与对称/周期边界。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `M` | `n by n` 或 `n by n by 2` | 标量场（求梯度）或 2 通道向量场（求散度） / scalar or 2-channel field |
  | `options` | struct | 选项（见下） / options |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `G` | 输入为标量 → `n by n by 2`；输入为向量场 → `n by n` | 梯度或散度 / gradient or divergence |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `options.bound` | `'sym'`/其它 | `'sym'` | 对称（前向/后向）或周期边界 |

- **变量流动 / Variable flow**: `M,options →(size(M,3)==2? G=mydiv(M,options) : G=mygrad(M,options))→ mydiv: 读 bound →(sym: 内部中心差分 + 边界前向/后向 : 周期索引 sel1/sel2 差分)→ mygrad: 读 bound →(sym: g(:,:,1),g(:,:,2) 中心差分+边界 : 周期差分 cat(3))→ G`
- **算法步骤 / Algorithm**:
  1. `size(M,3)==2`（即 2 通道向量场）→ 走 `mydiv` 求散度；否则走 `mygrad` 求梯度（L14–L18）。
  2. `mydiv`：`'sym'` 时内部用 `(g(3:end)-g(1:end-2))/2`，边界用单边差分；非 `'sym'` 用周期移位 `sel1/sel2`（L24–L43）。
  3. `mygrad`：`'sym'` 时对 `x`/`y` 两方向中心差分并处理首末行；非 `'sym'` 用周期移位后 `cat(3,...)`（L48–L68）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 M,options"] --> B{"size(M,3) == 2 ?"}
    B -- 是 --> C["mydiv: 求散度"]
    B -- 否 --> D["mygrad: 求梯度"]
    C --> E{"bound == 'sym' ?"}
    E -- 是 --> E1["内部中心差分 + 边界单边"]
    E -- 否 --> E2["周期移位差分"]
    D --> F{"bound == 'sym' ?"}
    F -- 是 --> F1["x/y 中心差分 + 边界处理"]
    F -- 否 --> F2["周期移位 cat(3)"]
    E1 --> Z["输出 G"]
    E2 --> Z
    F1 --> Z
    F2 --> Z
  ```

- **依赖 / Dependencies**: 局部子函数 `mydiv`/`mygrad`；[toolbox/getoptions.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/getoptions.m#L1)；内置 `zeros`/`cat`/`size`。
- **See also**: `compute_grad`, `perform_active_contour`, `callback_active_contour`, `gradient`
- **⚠️ 遮蔽与文档偏差 / Shadowing & doc mismatch**: 与 MATLAB 内置 `gradient` 用途相近但**非同名遮蔽**；函数头注释称"2D 数组求梯度、3D 数组求散度"，而**代码实际按 `size(M,3)==2` 判为散度**（`mydiv`），即以**通道数**而非数组维度分派，注释与实现**不一致**（使用时以代码为准）。**不遮蔽内置**（无内置 `divgrad`）。

### `eucdist2` — 2-D 欧氏距离变换 [MEX]
- **文件 / File**: [eucdist2.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/eucdist2.m#L1-L18)
- **署名 / Attribution**: The MathWorks, Inc.（文件头证据：`%   Copyright 1993-2003 The MathWorks, Inc.`，`$Revision: 1.6.4.2$`）。**非 Peyre 代码**，属 MATLAB 旧版自带函数的 `.m` 注释桩。
- **签名 / Signature**: `varargout = eucdist2(varargin)`（桩）；实际接口 `D = eucdist2(BW)`、`[D,L] = eucdist2(BW)`
- **功能 / Purpose**: `EN: Compute the 2-D Euclidean distance transform of a binary image (distance to nearest nonzero pixel), optionally returning the nearest-nonzero linear index map; implemented by the precompiled MEX, this .m is only a loader stub that errors when the MEX is missing.` / 中文：对二值图做 2-D 欧氏距离变换（到最近非零像素的距离），可选返回最近非零像素的线性索引；实现在预编译 MEX 中，本 `.m` 仅为加载桩，MEX 缺失即报错。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `BW` | 2-D 逻辑/数值 | 二值图（仅支持 2-D） / binary image |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `D` | 2-D double | 到最近非零像素的欧氏距离 / Euclidean distance transform |
  | `L` | 2-D double | 最近非零像素的线性索引 / nearest-nonzero linear index |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | — | — | — | 桩无 `varargin` 解析；语义由 MEX 接口决定 |

- **变量流动 / Variable flow**: `varargin →(MEX 存在: eucdist2.mex*/eucdist2.dll 被调用)→ D[,L]`；桩路径：`varargin → error('Images:eucdist2:missingMEXFile', ...)`
- **算法步骤 / Algorithm**:
  1. 本 `.m` 不含算法，仅 `%#mex` 标注与 `error` 桩（L16–L18）。
  2. 运行期由同目录预编译二进制 `eucdist2.dll`（Windows）/`eucdist2.mexglx`（Linux 旧版）接管；源码 `mex/eucdist2.c` 随库附带。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 BW (varargin)"] --> B{"预编译 MEX 是否可用 ?"}
    B -- 是 --> C["调用 eucdist2.mex*/dll → D[,L]"]
    B -- 否 --> D["error: Missing MEX-file"]
    C --> Z["输出 D[,L]"]
  ```

- **依赖 / Dependencies**: 预编译二进制 [eucdist2.dll](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/eucdist2.dll)、`eucdist2.mexglx`；C 源码 [mex/eucdist2.c](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/mex/eucdist2.c#L1)。
- **See also**: `bwdist`, `pdist2`, `compute_euclidean_distance`
- **⚠️ MEX 要求 + 遮蔽/重叠 / MEX requirement & overlap**: **[MEX]** 需在目标平台**预编译** `eucdist2`（Windows 已含 `eucdist2.dll`；其它平台需自行 `mex eucdist2.c`）。功能上等价于现在的内置 `bwdist`（距离变换）与 `pdist2`（成对距离）**重叠**，属旧版 MathWorks 遗留函数。**不遮蔽现代内置**（`eucdist2` 已非当前 MATLAB 自带函数）。

### `callback_active_contour` — 主动轮廓的共轭梯度回调
- **文件 / File**: [callback_active_contour.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/callback_active_contour.m#L1-L19)
- **署名 / Attribution**: Gabriel Peyre / Toolbox Fast Marching（BSD）。文件头证据：`%   Copytight (c) 2007 Gabriel Peyre`（原文拼写为 `Copytight`）。
- **签名 / Signature**: `y = callback_active_contour(x, options)`
- **功能 / Purpose**: `EN: Per-iteration callback for the active-contour solver: returns one explicit-Euler step y = x - dt * weighted_divgrad(x) (optionally weighted by an extra field E), used by perform_active_contour's conjugate-gradient loop.` / 中文：主动轮廓求解器的**逐迭代回调**：返回一步显式欧拉更新 `y = x - dt·(加权 divgrad)(x)`（若有 `E` 则再乘 `E`），供 `perform_active_contour` 的共轭梯度循坏调用。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `x` | `n*n by 1` 或 `n by n` | 当前轮廓（重排为 `n by n`） / current field |
  | `options` | struct | 见下 / options |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `y` | `n*n by 1` | 一步更新后的场（列向量） / updated field |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `options.dt` | `double` | 必需 | 时间步长 |
  | `options.n` | `int` | 必需 | 网格边长 |
  | `options.d` | `n by n` | 必需 | 权重/度量场 |
  | `options.E` | `n by n` 或 `[]` | 空则不加权 | 额外乘子场 |

- **变量流动 / Variable flow**: `x,options → dt=options.dt; n=options.n; d=options.d → x=reshape(x,n,n) → y=divgrad( divgrad(x,options)./repmat(d,[1 1 2]), options ) →(isempty(options.E)? y=x(:)-dt*d(:).*y(:) : y=x(:)-dt*E(:).*d(:).*y(:))→ y`
- **算法步骤 / Algorithm**:
  1. 取出 `dt/n/d`，把 `x` 重排成 `n by n`（L10–L13）。
  2. 计算"加权拉普拉斯式"算子 `y = divgrad( divgrad(x)./d )`（用 `divgrad` 的梯度再散度，并按 `d` 加权）（L14）。
  3. 无 `E` 时 `y = x(:) - dt * d(:).*y(:)`；否则再乘 `E`（L15–L19）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 x,options"] --> B["读 dt,n,d; x=reshape(x,n,n)"]
    B --> C["y = divgrad( divgrad(x,options)./repmat(d,[1 1 2]), options )"]
    C --> D{"options.E 是否为空 ?"}
    D -- 是 --> D1["y = x(:) - dt * d(:).*y(:)"]
    D -- 否 --> D2["y = x(:) - dt * E(:).*d(:).*y(:)"]
    D1 --> Z["输出 y"]
    D2 --> Z
  ```

- **依赖 / Dependencies**: [divgrad.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/divgrad.m#L1)；内置 `reshape`/`repmat`/`isempty`。
- **See also**: `perform_active_contour`, `divgrad`, `test_active_contour`
- **备注 / Note**: 强耦合于 `divgrad` 的"标量→梯度、2 通道→散度"约定（此处先用 `size(M,3)==1` 求梯度，再对 2 通道结果求散度）。**不遮蔽内置**。

---

## 函数清单核对表 / Coverage Checklist

> 20 个文件逐一对应 20 个条目；锚点均可点击跳转至源码。

| # | 函数 / File | 锚点 / Anchor | 署名 / Attribution | 一句话用途 / One-line purpose | 需 MEX / 工具箱 |
|---|---|---|---|---|---|
| 1 | `compute_distance_landmark` | [compute_distance_landmark.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_distance_landmark.m#L1-L114) | Gabriel Peyre (2005) | 路标点近似距离 | 否（`'proxy'` 分支缺 `matrix_sampling_get`） |
| 2 | `compute_geodesic` | [compute_geodesic.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_geodesic.m#L1-L363) | Gabriel Peyre (2007) | 2D/3D 测地路径回溯 | 否 |
| 3 | `compute_geodesic_mesh` | [compute_geodesic_mesh.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_geodesic_mesh.m#L1-L187) | Gabriel Peyre (2007) | 网格测地路径回溯 | 否 |
| 4 | `compute_eccentricity_transform` | [compute_eccentricity_transform.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_eccentricity_transform.m#L1-L127) | Gabriel Peyre (2006) | 形状偏心距 | 否（`'euclidean'` 分支缺 `compute_euclidean_distance`） |
| 5 | `compute_bending_invariant` | [compute_bending_invariant.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_bending_invariant.m#L1-L58) | Gabriel Peyre (2007) | 弯曲不变量（MDS 嵌入） | 否（需 `eigs`） |
| 6 | `compute_edge_energy` | [compute_edge_energy.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_edge_energy.m#L1-L76) | Gabriel Peyre (2005) | 速度/势场构造 | 3D 需 `smooth3`（Image Processing Toolbox） |
| 7 | `compute_heuristic_landmark` | [compute_heuristic_landmark.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_heuristic_landmark.m#L1-L14) | Gabriel Peyre (2005) | 路标对齐启发式 | 否 |
| 8 | `compute_heuristic_multiresolution` | [compute_heuristic_multiresolution.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_heuristic_multiresolution.m#L1-L51) | 无署名 / uncredited | 多分辨率启发式 | 否（放大用 `image_resize`，**全库缺失**） |
| 9 | `compute_alpha_map` | [compute_alpha_map.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_alpha_map.m#L1-L44) | Gabriel Peyre (2007) | alpha 图与配色 | 否（需已开图窗） |
| 10 | `compute_levelset_shape` | [compute_levelset_shape.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_levelset_shape.m#L1-L97) | Gabriel Peyre (2007) | 解析水平集测试形状 | 否 |
| 11 | `compute_saddle_points` | [compute_saddle_points.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_saddle_points.m#L1-L60) | Gabriel Peyre (2008) | Voronoi 鞍点提取 | 否 |
| 12 | `compute_shape_boundary` | [compute_shape_boundary.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_shape_boundary.m#L1-L88) | Gabriel Peyre (2007) | 8 连通有序边界 | 否 |
| 13 | `compute_voronoi_triangulation` | [compute_voronoi_triangulation.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_voronoi_triangulation.m#L1-L124) | Gabriel Peyre (2008) | 2D/3D Voronoi → 三角化 | 否 |
| 14 | `compute_voronoi_triangulation_mesh` | [compute_voronoi_triangulation_mesh.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/compute_voronoi_triangulation_mesh.m#L1-L34) | Gabriel Peyre (2006) | 网格 Voronoi → 三角化 | 否 |
| 15 | `generate_constrained_map` | [generate_constrained_map.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/generate_constrained_map.m#L1-L43) | Gabriel Peyre (2004) | 方向相关约束图 | 需 `imdilate`（Image Processing Toolbox） |
| 16 | `load_potential_map` | [load_potential_map.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/load_potential_map.m#L1-L69) | Gabriel Peyre (2004) | 生成/加载内置势场 | 否（需 `data/*.png`） |
| 17 | `convert_distance_color` | [convert_distance_color.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/convert_distance_color.m#L1-L25) | Gabriel Peyre (2007) | 距离场 → 彩色图 | 否 |
| 18 | `divgrad` | [divgrad.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/divgrad.m#L1-L68) | Gabriel Peyre (2007) | 梯度/散度（中心差分） | 否 |
| 19 | `eucdist2` | [eucdist2.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/eucdist2.m#L1-L18) | The MathWorks (1993-2003) | 2-D 欧氏距离变换 | **[MEX]** 需 `eucdist2.dll`/自编译 |
| 20 | `callback_active_contour` | [callback_active_contour.m](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/callback_active_contour.m#L1-L19) | Gabriel Peyre (2007) | 主动轮廓迭代回调 | 否 |

覆盖核对：清单中的 20 个文件**全部存在**并各有 `###` 条目（20/20）。其中 1 个需预编译 **MEX**（`eucdist2`），2 个需 **Image Processing Toolbox**（`compute_edge_energy` 的 3D 分支、`generate_constrained_map`）；另有 3 处**依赖缺口/异常**需注意：`compute_distance_landmark` 的 `'proxy'` 分支、`compute_eccentricity_transform` 的 `'euclidean'` 分支、`compute_heuristic_multiresolution` 的放大分支各自调用了本仓库中不存在的辅助函数。
