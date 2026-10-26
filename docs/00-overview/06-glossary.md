# 术语表 / Glossary

> 本文档给出 `gptoolbox-1` 涉及的**几何处理与实现术语中英对照**，供重建者在阅读源码时
> 快速理解语义。每条术语给出：**English → 中文 → 一句话解释 →（如相关）对应的 gptoolbox 函数名**。
> 函数名均已在仓库中确认存在（不确定的一律不写），可点击链接指向真实文件。
>
> This glossary maps **geometry-processing and implementation terminology (EN ⇄ ZH)** used by
> `gptoolbox-1`. Each entry gives **English → Chinese → one-line explanation → (if any) the
> gptoolbox function**. Function names are verified to exist in this repo; clickable links point
> to real files.

- 语言 / Language：中英对照（中文为主）
- 相关章节 / See also：[01-project-overview.md](01-project-overview.md)、
  [02-architecture.md](02-architecture.md)、[05-dependency-map.md](05-dependency-map.md)
- 符号约定 / Notation：`V` 顶点、`F` 面、`T` 四面体、`Q` 四边形、`E` 边、`C` 角标/控制点、
  `W` 权重、`L` 拉普拉斯、`M` 质量矩阵（详见 [docs/README.md](../README.md) §2）。

---

## 1. 数据结构与索引 / Data Structures & Indexing

| English | 中文 | 一句话解释 | gptoolbox 函数 / 文件 |
|---|---|---|---|
| Vertex positions `V` | 顶点坐标矩阵 | `#V by 3`（2D 时 `#V by 2`）的坐标表 | [`readOBJ`](file:///c:/VSWork/gptoolbox-1/mesh/readOBJ.m)、[`readOFF`](file:///c:/VSWork/gptoolbox-1/mesh/readOFF.m) |
| Triangle indices `F` | 三角面索引矩阵 | `#F by 3`，每行是一个三角形的三个顶点索引 | [`edges`](file:///c:/VSWork/gptoolbox-1/mesh/edges.m)、[`outline`](file:///c:/VSWork/gptoolbox-1/mesh/outline.m) |
| Tetrahedron indices `T` | 四面体索引矩阵 | `#T by 4`，每行一个四面体 | [`tetgen`](file:///c:/VSWork/gptoolbox-1/wrappers/tetgen.m)、[`regular_tetrahedral_mesh`](file:///c:/VSWork/gptoolbox-1/mesh/regular_tetrahedral_mesh.m) |
| Quad indices `Q` | 四边形索引矩阵 | `#Q by 4`，每行一个四边形 | [`readOBJ`](file:///c:/VSWork/gptoolbox-1/mesh/readOBJ.m) |
| Edge indices `E` | 边索引矩阵 | `#E by 2`，无向边端点对 | [`edges`](file:///c:/VSWork/gptoolbox-1/mesh/edges.m) |
| Corner `C` / control `C` | 角标 / 控制点 | 三角形的“顶点-角”索引（每面三个角）；在曲线/MVC 语境下也指控制点 | [`cotangent`](file:///c:/VSWork/gptoolbox-1/mesh/cotangent.m)、[`mvc`](file:///c:/VSWork/gptoolbox-1/mesh/mvc.m) |
| Weights `W` | 权重矩阵 | `#V by #handles`，每顶点在各控制元上的权重 | [`biharmonic_bounded`](file:///c:/VSWork/gptoolbox-1/mesh/biharmonic_bounded.m)、[`bone_heat`](file:///c:/VSWork/gptoolbox-1/mesh/bone_heat.m) |
| Laplacian `L` | 拉普拉斯矩阵 | `#V by #V` 稀疏算子，度量局部平坦程度 | [`cotmatrix`](file:///c:/VSWork/gptoolbox-1/mesh/cotmatrix.m) |
| Mass matrix `M` | 质量矩阵 | `#V by #V` 稀疏内积矩阵（面积/体积权重） | [`massmatrix`](file:///c:/VSWork/gptoolbox-1/mesh/massmatrix.m) |
| Simplex | 单纯形 | 三角形/四面体等的统称；“simplex size” = `size(F,2)` | [`cotmatrix`](file:///c:/VSWork/gptoolbox-1/mesh/cotmatrix.m) 以 `size(F,2)` 分支 |
| Row-major indexing | 行主序索引 | “`#F by 3`” 即“每行一个面”，`F(i,:)` 为第 i 个面 | 全库约定 |
| 1-based indexing | 1 起始索引 | MATLAB 索引从 1 开始（对应 C/C++ 的 0-based 需换算） | [`tetgen`](file:///c:/VSWork/gptoolbox-1/wrappers/tetgen.m) 读入后显式处理 |
| Sparse matrix | 稀疏矩阵 | 算子在 MATLAB 中一律用 `sparse` 存储，支撑上万顶点隐式求解 | [`sparse_eigs`](file:///c:/VSWork/gptoolbox-1/matrix/sparse_eigs.m) |
| `#V by 3` notation | `#V by 3` 尺寸记法 | 源码注释里表示“行数 by 列数”的统一写法 | 见 [CLAUDE.md](file:///c:/VSWork/gptoolbox-1/CLAUDE.md) |
| Adjacency matrix | 邻接矩阵 | `#V by #V` 稀疏 0/1 或加权图矩阵 | [`adjacency_matrix`](file:///c:/VSWork/gptoolbox-1/mesh/adjacency_matrix.m) |
| Half-edge | 半边 | 有向半边数据结构；**gptoolbox 未实现显式半边结构**，改用“边/有向边列表 + 索引”表示 | [`outline`](file:///c:/VSWork/gptoolbox-1/mesh/outline.m)、[`edges`](file:///c:/VSWork/gptoolbox-1/mesh/edges.m) |
| Directed edge | 有向边 | 带方向的边列表，可用于提取边界环路 | [`outline`](file:///c:/VSWork/gptoolbox-1/mesh/outline.m) |
| Boundary edge | 边界边 | 只被一个面共享的边 | [`outline`](file:///c:/VSWork/gptoolbox-1/mesh/outline.m)、[`on_boundary`](file:///c:/VSWork/gptoolbox-1/mesh/on_boundary.m) |
| Non-manifold edge | 非流形边 | 被 ≥3 个面共享的边（破坏流形性） | [`split_nonmanifold`](file:///c:/VSWork/gptoolbox-1/mex/split_nonmanifold.m) `[MEX]` |

---

## 2. 离散微分几何 / Discrete Differential Geometry (DDG)

| English | 中文 | 一句话解释 | gptoolbox 函数 / 文件 |
|---|---|---|---|
| Cotangent Laplacian | 余切拉普拉斯算子 | 用余切权重构造的离散 Laplace–Beltrami 算子 | [`cotmatrix`](file:///c:/VSWork/gptoolbox-1/mesh/cotmatrix.m) |
| Cotangent weight | 余切权重 | 每个角对应的 `cot(θ)`，拉普拉斯边的权重 | [`cotangent`](file:///c:/VSWork/gptoolbox-1/mesh/cotangent.m) |
| Mass matrix | 质量矩阵 | 有限元意义下的内积矩阵（Voronoi/barycentric 两种） | [`massmatrix`](file:///c:/VSWork/gptoolbox-1/mesh/massmatrix.m) |
| Voronoi area | Voronoi 面积权重 | 以顶点周围 Voronoi 区域面积为权 | [`massmatrix`](file:///c:/VSWork/gptoolbox-1/mesh/massmatrix.m)（`'voronoi'`） |
| Barycentric area | 重心面积权重 | 以相邻三角形重心划分面积为权 | [`massmatrix`](file:///c:/VSWork/gptoolbox-1/mesh/massmatrix.m)（`'barycentric'`） |
| Gradient operator | 梯度算子 | 面片常值梯度矩阵 `G`，`G*u` 得逐面梯度 | [`grad`](file:///c:/VSWork/gptoolbox-1/mesh/grad.m) |
| Divergence operator | 散度算子 | 梯度的（负）转置，把逐面向量场映射回逐顶点 | [`div`](file:///c:/VSWork/gptoolbox-1/mesh/div.m) |
| Laplace–Beltrami operator | Laplace–Beltrami 算子 | 曲面上的拉普拉斯推广，`cotmatrix` 即其离散近似 | [`cotmatrix`](file:///c:/VSWork/gptoolbox-1/mesh/cotmatrix.m) |
| Intrinsic Delaunay | 内在 Delaunay（三角剖分） | 仅用边长信息、通过翻边达到 Delaunay 的剖分 | [`intrinsic_delaunay_cotmatrix`](file:///c:/VSWork/gptoolbox-1/mesh/intrinsic_delaunay_cotmatrix.m)、[`is_intrinsic_delaunay`](file:///c:/VSWork/gptoolbox-1/mesh/is_intrinsic_delaunay.m)、[`flip_edges`](file:///c:/VSWork/gptoolbox-1/mesh/flip_edges.m) |
| Intrinsic Laplacian | 内在拉普拉斯 | 由边长（非坐标）构造的拉普拉斯 | [`cotmatrix_intrinsic`](file:///c:/VSWork/gptoolbox-1/mesh/cotmatrix_intrinsic.m)、[`massmatrix_intrinsic`](file:///c:/VSWork/gptoolbox-1/mesh/massmatrix_intrinsic.m) |
| Crouzeix–Raviart | Crouzeix–Raviart 元 | 自由度在边中点上的有限元，用于（双）调和约束 | [`crouzeix_raviart_cotmatrix`](file:///c:/VSWork/gptoolbox-1/mesh/crouzeix_raviart_cotmatrix.m)、[`crouzeix_raviart_massmatrix`](file:///c:/VSWork/gptoolbox-1/mesh/crouzeix_raviart_massmatrix.m) |
| Mean curvature | 平均曲率 | 主曲率均值，`(κ1+κ2)/2` | [`discrete_mean_curvature`](file:///c:/VSWork/gptoolbox-1/mesh/discrete_mean_curvature.m)、[`curvature`](file:///c:/VSWork/gptoolbox-1/mesh/curvature.m) |
| Gaussian curvature | 高斯曲率 | 主曲率之积 `κ1·κ2`，离散用角亏度量 | [`discrete_gaussian_curvature`](file:///c:/VSWork/gptoolbox-1/mesh/discrete_gaussian_curvature.m) |
| Principal curvature | 主曲率 | 曲面上两个正交方向的最大/最小法曲率 | [`principal_curvature`](file:///c:/VSWork/gptoolbox-1/mex/principal_curvature.m) `[MEX]` |
| Dihedral angle | 二面角 | 相邻两面间的夹角 | [`dihedral_angles`](file:///c:/VSWork/gptoolbox-1/mesh/dihedral_angles.m) |
| Internal angle | 内角 | 三角形在某一顶点处的内角 | [`internalangles`](file:///c:/VSWork/gptoolbox-1/mesh/internalangles.m) |
| Edge length | 边长 | 三角形各边长度（内在几何基础量） | [`edge_lengths`](file:///c:/VSWork/gptoolbox-1/mesh/edge_lengths.m) |
| Double area / volume | 二倍面积 / （四面体）体积 | 三角形二倍面积（或四面体 6 倍体积），避免开根号 | [`doublearea`](file:///c:/VSWork/gptoolbox-1/mesh/doublearea.m) |
| Harmonic function | 调和函数 | 满足 `Δu=0` 的函数（线性插值解） | [`harmonic`](file:///c:/VSWork/gptoolbox-1/mesh/harmonic.m) |
| Biharmonic | 双调和 | 满足 `Δ²u=0`；用于光滑权重与薄板能量 | [`biharmonic_bounded`](file:///c:/VSWork/gptoolbox-1/mesh/biharmonic_bounded.m) |
| Dirichlet / Neumann / Robin BC | 第一/第二/第三类边界条件 | 固定值 / 固定通量 / 两者线性组合的边界条件 | [`heat_geodesic`](file:///c:/VSWork/gptoolbox-1/mesh/heat_geodesic.m)（`'BoundaryConditions'`） |
| Poisson equation | 泊松方程 | `Δφ = ∇·X`，热方法最后一步的线性方程 | [`heat_geodesic`](file:///c:/VSWork/gptoolbox-1/mesh/heat_geodesic.m) |
| Solenoidal / tangential | 无散 / 切向场 | 与散度/切向相关的向量场概念（投影/分解用） | [`hessian_squared`](file:///c:/VSWork/gptoolbox-1/mesh/hessian_squared.m) 等 |

---

## 3. 网格处理 / Mesh Processing

| English | 中文 | 一句话解释 | gptoolbox 函数 / 文件 |
|---|---|---|---|
| Parameterization | 参数化 | 把曲面映射到平面（`U` 为 `#V by 2`） | [`lscm`](file:///c:/VSWork/gptoolbox-1/mesh/lscm.m)、[`harmonic`](file:///c:/VSWork/gptoolbox-1/mesh/harmonic.m) |
| LSCM | 最小二乘共形映射 | Least-Squares Conformal Map，共形（保角）参数化 | [`lscm`](file:///c:/VSWork/gptoolbox-1/mesh/lscm.m) |
| Conformal | 共形 / 保角 | 局部保角（可差缩放）的映射 | [`lscm`](file:///c:/VSWork/gptoolbox-1/mesh/lscm.m) |
| Harmonic map | 调和映射 | 以调和能量最小的参数化 | [`harmonic`](file:///c:/VSWork/gptoolbox-1/mesh/harmonic.m)、[`kharmonic`](file:///c:/VSWork/gptoolbox-1/mesh/kharmonic.m) |
| k-harmonic coordinates | k-调和坐标 | 广义调和坐标（k=1 调和、k=2 双调和） | [`kharmonic`](file:///c:/VSWork/gptoolbox-1/mesh/kharmonic.m) |
| ARAP | 尽可能刚性变形 | As-Rigid-As-Possible：局部拟合旋转 + 全局解泊松的迭代变形 | [`arap`](file:///c:/VSWork/gptoolbox-1/mesh/arap.m)、[`arap_linear_block`](file:///c:/VSWork/gptoolbox-1/mesh/arap_linear_block.m)、[`arap_rhs`](file:///c:/VSWork/gptoolbox-1/mesh/arap_rhs.m) |
| ASAP | 尽可能仿射变形 | As-Soon/As-Affine-As-Possible 参数化（ARAP 的简化版） | [`takeo_asap`](file:///c:/VSWork/gptoolbox-1/mesh/takeo_asap.m) |
| Local–global (alternating) | 局部-全局（交替）迭代 | 局部步求旋转、全局步解线性系统，交替收敛 | [`arap`](file:///c:/VSWork/gptoolbox-1/mesh/arap.m) |
| Skinning | 蒙皮 | 由骨架/控制元权重驱动网格变形 | [`lbs`](file:///c:/VSWork/gptoolbox-1/mesh/lbs.m) |
| LBS | 线性混合蒙皮 | Linear Blend Skinning：`U = W·T·V` 的加权线性混合 | [`lbs`](file:///c:/VSWork/gptoolbox-1/mesh/lbs.m) |
| DQS | 对偶四元数蒙皮 | Dual-Quaternion Skinning，用对偶四元数混合刚体变换，避免“糖纸”现象 | [`dualquatlbs`](file:///c:/VSWork/gptoolbox-1/mesh/dualquatlbs.m) |
| Bone heat | 骨热扩散权重 | 用热扩散解自动生成骨骼蒙皮权重 | [`bone_heat`](file:///c:/VSWork/gptoolbox-1/mesh/bone_heat.m) |
| Bounded biharmonic weights | 有界双调和权重 | 双调和能量 + 箱约束 + 单位分解，常用自动蒙皮 | [`biharmonic_bounded`](file:///c:/VSWork/gptoolbox-1/mesh/biharmonic_bounded.m) |
| Biharmonic coordinates | 双调和坐标 | 笼/控制元的双调和插值坐标 | [`biharmonic_coordinates`](file:///c:/VSWork/gptoolbox-1/mesh/biharmonic_coordinates.m) |
| MVC | 均值坐标 | Mean Value Coordinates，多边形/网格上的重心型插值坐标 | [`mvc`](file:///c:/VSWork/gptoolbox-1/mesh/mvc.m) |
| Cage deformation | 笼变形 | 用低模“笼”控制高模的变形 | [`lazy_cage`](file:///c:/VSWork/gptoolbox-1/mesh/lazy_cage.m)、[`encage`](file:///c:/VSWork/gptoolbox-1/mesh/encage.m) |
| Laplacian editing | 拉普拉斯编辑 | 保拉普拉斯坐标的交互式编辑 | [`iterative_laplacian_mesh_editing`](file:///c:/VSWork/gptoolbox-1/mesh/iterative_laplacian_mesh_editing.m)、[`laplacian_editing_system`](file:///c:/VSWork/gptoolbox-1/mesh/laplacian_editing_system.m) |
| Mesh smoothing | 网格光顺 | 迭代平滑顶点位置（拉普拉斯/曲率流） | [`laplacian_smooth`](file:///c:/VSWork/gptoolbox-1/mesh/laplacian_smooth.m)、[`curve_smooth`](file:///c:/VSWork/gptoolbox-1/mesh/curve_smooth.m) |
| Remeshing | 重网格化 | 重新分布顶点/改变连接以改善质量 | [`remesh_level_set`](file:///c:/VSWork/gptoolbox-1/mesh/remesh_level_set.m)、[`remesh_planar_patches`](file:///c:/VSWork/gptoolbox-1/mesh/remesh_planar_patches.m) |
| Subdivision | 细分 | 迭代插入顶点细化网格 | [`loop`](file:///c:/VSWork/gptoolbox-1/mesh/loop.m)、[`sqrt3`](file:///c:/VSWork/gptoolbox-1/mesh/sqrt3.m)、[`upsample`](file:///c:/VSWork/gptoolbox-1/mesh/upsample.m) |
| Decimation | 简化/抽稀 | 减少面片数（边坍缩） | [`decimate_libigl`](file:///c:/VSWork/gptoolbox-1/mex/decimate_libigl.m) `[MEX]`、[`qslim`](file:///c:/VSWork/gptoolbox-1/wrappers/qslim.m) |

---

## 4. 拓扑与修复 / Topology & Repair

| English | 中文 | 一句话解释 | gptoolbox 函数 / 文件 |
|---|---|---|---|
| Boundary | 边界 | 只被一个面共享的边/面集合 | [`on_boundary`](file:///c:/VSWork/gptoolbox-1/mesh/on_boundary.m)、[`boundary_faces`](file:///c:/VSWork/gptoolbox-1/mesh/boundary_faces.m) |
| Outline | 轮廓边 | 有向边界边列表（按环绕方向） | [`outline`](file:///c:/VSWork/gptoolbox-1/mesh/outline.m) |
| Connected components | 连通分量 | 把网格分解为互不相连的块 | [`connected_components`](file:///c:/VSWork/gptoolbox-1/mesh/connected_components.m) |
| Watertight | 封闭 / 无边界 | 无边界边的闭合曲面（可求内外） | [`is_watertight`](file:///c:/VSWork/gptoolbox-1/mesh/is_watertight.m) |
| Self-intersection | 自交 | 面片彼此相交 | [`selfintersect`](file:///c:/VSWork/gptoolbox-1/mex/selfintersect.m) `[MEX]` |
| Winding number | 环绕数 | 点相对闭合曲面的卷绕次数（判内外/布尔） | [`winding_number`](file:///c:/VSWork/gptoolbox-1/mex/winding_number.m) `[MEX]`、[`solid_angle`](file:///c:/VSWork/gptoolbox-1/mesh/solid_angle.m) |
| Duplicate vertices | 几何重复顶点 | 坐标重合的多余顶点 | [`remove_duplicate_vertices`](file:///c:/VSWork/gptoolbox-1/mesh/remove_duplicate_vertices.m) |
| Unreferenced vertices | 未引用顶点 | 不被任何面使用的顶点 | [`remove_unreferenced`](file:///c:/VSWork/gptoolbox-1/mesh/remove_unreferenced.m) |
| Degenerate face | 退化面 | 零面积或含重复索引的面 | [`remove_degenerate_faces`](file:///c:/VSWork/gptoolbox-1/mesh/remove_degenerate_faces.m)、[`collapse_small_triangles`](file:///c:/VSWork/gptoolbox-1/mex/collapse_small_triangles.m) `[MEX]` |
| Fill holes | 补洞 | 封闭网格上的孔洞 | [`fill_holes`](file:///c:/VSWork/gptoolbox-1/mesh/fill_holes.m) |
| Mesh clean-up | 网格清理 | 去重、去退化、剖分自交的综合管线 | [`clean_mesh`](file:///c:/VSWork/gptoolbox-1/mesh/clean_mesh.m) |
| Boolean (CSG) | 布尔运算 | 网格并/交/差 | [`mesh_boolean`](file:///c:/VSWork/gptoolbox-1/mex/mesh_boolean.m) `[MEX]`、[`mesh_boolean_winding_number`](file:///c:/VSWork/gptoolbox-1/mesh/mesh_boolean_winding_number.m) |
| Convex hull | 凸包 | 包含点集的最小凸体 | [`outer_hull`](file:///c:/VSWork/gptoolbox-1/mex/outer_hull.m) `[MEX]` |
| Reorient facets | 面朝向一致化 | 让所有面法向朝外/一致 | [`reorient_facets`](file:///c:/VSWork/gptoolbox-1/mex/reorient_facets.m) `[MEX]` |
| Snap rounding | 吸附取整 | 把近重合点吸附到格点以提升鲁棒性 | [`snap_rounding`](file:///c:/VSWork/gptoolbox-1/mex/snap_rounding.m) `[MEX]` |
| Non-manifold split | 非流形分裂 | 把非流形结构拆成若干流形片 | [`split_nonmanifold`](file:///c:/VSWork/gptoolbox-1/mex/split_nonmanifold.m) `[MEX]` |

---

## 5. 几何查询与算法 / Geometry Queries & Algorithms

| English | 中文 | 一句话解释 | gptoolbox 函数 / 文件 |
|---|---|---|---|
| Closest point / squared distance | 最近点 / 平方距离 | 点到（面/边/体）的最近点与距离平方 | [`point_mesh_squared_distance`](file:///c:/VSWork/gptoolbox-1/mex/point_mesh_squared_distance.m) `[MEX]` |
| Point location | 点定位 | 判断点落在哪个（四面体）单元内 | [`in_element`](file:///c:/VSWork/gptoolbox-1/mesh/in_element.m)、[`in_element_aabb`](file:///c:/VSWork/gptoolbox-1/mex/in_element_aabb.m) `[MEX]` |
| Signed distance field (SDF) | 符号距离场 | 到曲面带符号的距离（内负外正） | [`signed_distance`](file:///c:/VSWork/gptoolbox-1/mex/signed_distance.m) `[MEX]`、[`df_build`](file:///c:/VSWork/gptoolbox-1/mesh/df_build.m)、[`df_query`](file:///c:/VSWork/gptoolbox-1/mesh/df_query.m) |
| Geodesic distance | 测地距离 | 曲面上的最短路径长度 | [`heat_geodesic`](file:///c:/VSWork/gptoolbox-1/mesh/heat_geodesic.m)、[`exact_geodesic`](file:///c:/VSWork/gptoolbox-1/mex/exact_geodesic.m) `[MEX]` |
| Heat method | 热方法 | 三步（热扩散→归一化梯度→泊松）求测地距离 | [`heat_geodesic`](file:///c:/VSWork/gptoolbox-1/mesh/heat_geodesic.m) |
| Fast marching | 快速行进法 | 三角网格上的近似测地推进算法 | `external/toolbox_fast_marching/perform_fast_marching_mesh`（第三方） |
| Farthest point sampling | 最远点采样 | 迭代选取距已选集合最远的点 | [`farthest_points`](file:///c:/VSWork/gptoolbox-1/mesh/farthest_points.m) |
| Ray casting | 光线投射 | 射线与网格求交，用于可见性/遮挡 | [`ray_mesh_intersect`](file:///c:/VSWork/gptoolbox-1/mex/ray_mesh_intersect.m) `[MEX]`、[`bone_visible`](file:///c:/VSWork/gptoolbox-1/mesh/bone_visible.m) |
| Ambient occlusion | 环境光遮蔽 | 每点被周围几何遮挡的程度 | [`ambient_occlusion`](file:///c:/VSWork/gptoolbox-1/mex/ambient_occlusion.m) `[MEX]`、[`apply_ambient_occlusion`](file:///c:/VSWork/gptoolbox-1/mesh/apply_ambient_occlusion.m) |
| Inside / outside test | 内外判定 | 判断点是否在某闭合区域/多边形内 | [`point_inside_polygon`](file:///c:/VSWork/gptoolbox-1/mesh/point_inside_polygon.m) |
| Hausdorff distance | Hausdorff 距离 | 两网格/点集间的最大最小距离 | [`hausdorff`](file:///c:/VSWork/gptoolbox-1/mesh/hausdorff.m) |
| Bounding box | 包围盒 | 轴对齐最小包围长方体 | [`bounding_box`](file:///c:/VSWork/gptoolbox-1/mesh/bounding_box.m) |
| Delaunay triangulation | Delaunay 三角剖分 | 满足空外接圆性质的三角剖分 | [`delaunayize`](file:///c:/VSWork/gptoolbox-1/mesh/delaunayize.m)、[`triangle`](file:///c:/VSWork/gptoolbox-1/wrappers/triangle.m) |
| Constrained triangulation | 约束三角剖分 | 带边界/约束的三角剖分（供 2D 网格化） | [`triangle`](file:///c:/VSWork/gptoolbox-1/wrappers/triangle.m)、[`triangulate`](file:///c:/VSWork/gptoolbox-1/mex/triangulate.m) `[MEX]` |
| Tetrahedralization | 四面体化 | 把体域剖分为四面体 | [`tetgen`](file:///c:/VSWork/gptoolbox-1/wrappers/tetgen.m)、[`tetrahedralize`](file:///c:/VSWork/gptoolbox-1/mex/tetrahedralize.m) `[MEX]` |
| Voxelization | 体素化 | 把几何采样为规则体素 | [`voxelize`](file:///c:/VSWork/gptoolbox-1/mesh/voxelize.m)、[`readVOX`](file:///c:/VSWork/gptoolbox-1/mesh/readVOX.m) |
| Marching / isosurface | 等值面提取 | 从标量场抽取等值面 | [`polygonize`](file:///c:/VSWork/gptoolbox-1/mesh/polygonize.m)、[`signed_distance_isosurface`](file:///c:/VSWork/gptoolbox-1/mex/signed_distance_isosurface.m) `[MEX]` |

---

## 6. 几何基元与曲线 / Geometric Primitives & Curves

| English | 中文 | 一句话解释 | gptoolbox 函数 / 文件 |
|---|---|---|---|
| Primitive | 基元 | 基本几何体（立方体、球、胶囊…） | [`cube`](file:///c:/VSWork/gptoolbox-1/mesh/cube.m)、[`capsule_mesh`](file:///c:/VSWork/gptoolbox-1/mesh/capsule_mesh.m) |
| Sphere | 球面 | 细分/随机生成的球面网格 | [`subdivided_sphere`](file:///c:/VSWork/gptoolbox-1/mesh/subdivided_sphere.m)、[`randsphere`](file:///c:/VSWork/gptoolbox-1/mesh/randsphere.m) |
| Ball | 球体 | 球内随机/规则采样点 | [`randball`](file:///c:/VSWork/gptoolbox-1/mesh/randball.m) |
| Regular grid | 规则网格 | 规则的体素/晶格网格 | [`create_regular_grid`](file:///c:/VSWork/gptoolbox-1/mesh/create_regular_grid.m)、[`regular_tetrahedral_mesh`](file:///c:/VSWork/gptoolbox-1/mesh/regular_tetrahedral_mesh.m) |
| Barycentric coordinates | 重心坐标 | 点相对单纯形的重心权重 | [`barycentric_coordinates`](file:///c:/VSWork/gptoolbox-1/mesh/barycentric_coordinates.m) |
| Polyline / curve | 折线 / 曲线 | 有序点连线，常用于绘制轮廓 | [`triangulate_curves`](file:///c:/VSWork/gptoolbox-1/mesh/triangulate_curves.m)、[`cubic_uniformly_sample`](file:///c:/VSWork/gptoolbox-1/mesh/cubic_uniformly_sample.m) |
| Cubic Bézier | 三次 Bézier 曲线 | 四点确定的三次曲线 | [`fit_cubic_bezier`](file:///c:/VSWork/gptoolbox-1/mex/fit_cubic_bezier.m) `[MEX]` |
| B-spline / spline | B 样条 | 分段多项式曲线/曲面 | [`spline_closing`](file:///c:/VSWork/gptoolbox-1/mesh/spline_closing.m)、[`spline_uniformly_sample`](file:///c:/VSWork/gptoolbox-1/mesh/spline_uniformly_sample.m) |
| Solid angle | 立体角 | 面上一点张开的立体角（用于环绕数/可见性） | [`solid_angle`](file:///c:/VSWork/gptoolbox-1/mesh/solid_angle.m) |
| Polygon | 多边形 | 2D 闭合折线区域 | [`png2poly`](file:///c:/VSWork/gptoolbox-1/mesh/png2poly.m)、[`polygonize`](file:///c:/VSWork/gptoolbox-1/mesh/polygonize.m) |

---

## 7. 文件格式 / File Formats

| 格式 / Format | 说明 | 读 / 写函数 |
|---|---|---|
| OBJ | 通用三角/多边形网格（Wavefront） | [`readOBJ`](file:///c:/VSWork/gptoolbox-1/mesh/readOBJ.m) / [`writeOBJ`](file:///c:/VSWork/gptoolbox-1/mesh/writeOBJ.m) |
| OFF | Object File Format（点面表） | [`readOFF`](file:///c:/VSWork/gptoolbox-1/mesh/readOFF.m) / [`writeOFF`](file:///c:/VSWork/gptoolbox-1/mesh/writeOFF.m) |
| PLY | Stanford 多边形格式（含属性） | [`readPLY`](file:///c:/VSWork/gptoolbox-1/mesh/readPLY.m) / [`writePLY`](file:///c:/VSWork/gptoolbox-1/mesh/writePLY.m)（经 `external/read_ply`、`external/write_ply`） |
| STL | 三角面片（3D 打印常用） | [`readSTL`](file:///c:/VSWork/gptoolbox-1/mesh/readSTL.m) / [`writeSTL`](file:///c:/VSWork/gptoolbox-1/mesh/writeSTL.m) |
| WRL / VRML | 虚拟现实建模语言 | [`readWRL`](file:///c:/VSWork/gptoolbox-1/mesh/readWRL.m)（写出经 `external/vrml.m`） |
| MESH / BB | MEDIT / GAMMA 格式 | [`readMESH`](file:///c:/VSWork/gptoolbox-1/mesh/readMESH.m) / [`writeMESH`](file:///c:/VSWork/gptoolbox-1/mesh/writeMESH.m)（封装见 [`medit`](file:///c:/VSWork/gptoolbox-1/wrappers/medit.m)） |
| SMF | QSlim 简化器输入格式 | [`readSMF`](file:///c:/VSWork/gptoolbox-1/mesh/readSMF.m) / [`writeSMF`](file:///c:/VSWork/gptoolbox-1/mesh/writeSMF.m) |
| `.poly/.node/.ele/.face` | TetGen / Triangle 输入输出 | [`writePOLY_tetgen`](file:///c:/VSWork/gptoolbox-1/mesh/writePOLY_tetgen.m)、[`writePOLY_triangle`](file:///c:/VSWork/gptoolbox-1/mesh/writePOLY_triangle.m)、[`readNODE`](file:///c:/VSWork/gptoolbox-1/mesh/readNODE.m)、[`readELE`](file:///c:/VSWork/gptoolbox-1/mesh/readELE.m)、[`readFACE`](file:///c:/VSWork/gptoolbox-1/mesh/readFACE.m)、[`readEDGE`](file:///c:/VSWork/gptoolbox-1/mesh/readEDGE.m) |
| VOX | 体素栅格格式 | [`readVOX`](file:///c:/VSWork/gptoolbox-1/mesh/readVOX.m) |
| DMAT | libigl 稠密矩阵二进制格式 | [`readDMAT`](file:///c:/VSWork/gptoolbox-1/matrix/readDMAT.m) / [`writeDMAT`](file:///c:/VSWork/gptoolbox-1/matrix/writeDMAT.m) |
| MSH | Gmsh 网格格式 | [`readMSH`](file:///c:/VSWork/gptoolbox-1/mex/readMSH.m) `[MEX]` |
| PNG | 位图图像（用于从图像建网格/编码） | [`png2mesh`](file:///c:/VSWork/gptoolbox-1/mesh/png2mesh.m)、[`bwmesh`](file:///c:/VSWork/gptoolbox-1/mesh/bwmesh.m)、[`packpng`](file:///c:/VSWork/gptoolbox-1/imageprocessing/packpng.m) / [`unpackpng`](file:///c:/VSWork/gptoolbox-1/imageprocessing/unpackpng.m) |

> `[MEX]` = 编译型 C++ MEX（`.m` 为注释桩，实现在 `.cpp`）；详见
> [60-mex](../60-mex/README.md)。

---

## 8. 缩写表 / Abbreviations

| 缩写 | 全称 (EN) | 中文 | 说明 / 相关函数 |
|---|---|---|---|
| DDG | Discrete Differential Geometry | 离散微分几何 | 研究网格上离散化算子（`cotmatrix`/`massmatrix`/`grad`/`div`） |
| LSCM | Least-Squares Conformal Map | 最小二乘共形映射 | [`lscm`](file:///c:/VSWork/gptoolbox-1/mesh/lscm.m) |
| ARAP | As-Rigid-As-Possible | 尽可能刚性 | [`arap`](file:///c:/VSWork/gptoolbox-1/mesh/arap.m) |
| ASAP | As-Soon/As-Affine-As-Possible | 尽可能仿射 | [`takeo_asap`](file:///c:/VSWork/gptoolbox-1/mesh/takeo_asap.m) |
| MVC | Mean Value Coordinates | 均值坐标 | [`mvc`](file:///c:/VSWork/gptoolbox-1/mesh/mvc.m) |
| LBS | Linear Blend Skinning | 线性混合蒙皮 | [`lbs`](file:///c:/VSWork/gptoolbox-1/mesh/lbs.m) |
| DQS | Dual-Quaternion Skinning | 对偶四元数蒙皮 | [`dualquatlbs`](file:///c:/VSWork/gptoolbox-1/mesh/dualquatlbs.m) |
| UDQ | Ultimate Deformation Quality | 极限变形质量（蒙皮法） | **库内未实现**（仅有 LBS/DQS，检索 `udq` 无匹配） |
| BVH | Bounding Volume Hierarchy | 包围体层次 | 由 Embree 提供，服务于 [`ray_mesh_intersect`](file:///c:/VSWork/gptoolbox-1/mex/ray_mesh_intersect.m)、`bone_visible_embree` |
| AABB | Axis-Aligned Bounding Box | 轴对齐包围盒 | [`in_element_aabb`](file:///c:/VSWork/gptoolbox-1/mex/in_element_aabb.m) `[MEX]`、[`bounding_box`](file:///c:/VSWork/gptoolbox-1/mesh/bounding_box.m) |
| CSG | Constructive Solid Geometry | 构造实体几何 | [`mesh_boolean`](file:///c:/VSWork/gptoolbox-1/mex/mesh_boolean.m) `[MEX]` |
| SDF | Signed Distance Field | 符号距离场 | [`signed_distance`](file:///c:/VSWork/gptoolbox-1/mex/signed_distance.m) `[MEX]`、[`df_build`](file:///c:/VSWork/gptoolbox-1/mesh/df_build.m) |
| PDE | Partial Differential Equation | 偏微分方程 | 热方程/泊松方程等，[`heat_geodesic`](file:///c:/VSWork/gptoolbox-1/mesh/heat_geodesic.m) |
| KKT | Karush–Kuhn–Tucker | KKT（约束最优性）条件 | [`min_quad_with_fixed`](file:///c:/VSWork/gptoolbox-1/matrix/min_quad_with_fixed.m) 的鞍点系统；[`kharmonic`](file:///c:/VSWork/gptoolbox-1/mesh/kharmonic.m) 的 `Condensed=false` 分支 |
| PSD | Positive Semi-Definite | 半正定 | 二次型 `A` 通常 PSD；[`min_quad_with_fixed`](file:///c:/VSWork/gptoolbox-1/matrix/min_quad_with_fixed.m) 据性质选 Cholesky/LDL/LU |
| CGAL | Computational Geometry Algorithms Library | 计算几何算法库 | 用于 MEX 的精确谓词/布尔（`mex/mesh_boolean.cpp` 等，编译期依赖） |
| MEX | MATLAB Executable | MATLAB 可执行（C/C++ 扩展） | `mex/` 目录；`.m` 为注释桩，实现在 `.cpp`，构建见 [60-mex](../60-mex/README.md) |

---

## 9. 术语 → 函数速查 / Term → Function Quick Index

| 你想做的事 / Task | 首选函数 / Primary function |
|---|---|
| 读/写网格 | `readOBJ`/`writeOBJ`、`readOFF`/`writeOFF`、`readPLY`/`writePLY` |
| 去重、去未引用 | `remove_duplicate_vertices`、`remove_unreferenced` |
| 建 DDG 算子 | `cotmatrix`、`massmatrix`、`grad`、`div`、`doublearea` |
| 解约束二次型 | `min_quad_with_fixed`、`min_quad_with_fixed_active_set` |
| 参数化（共形/调和） | `lscm`、`harmonic`、`kharmonic` |
| 变形（刚性/交互） | `arap`、`deform`、`fast_mass_springs` |
| 自动蒙皮权重 | `bone_heat`、`biharmonic_bounded` |
| 蒙皮应用 | `lbs`、`dualquatlbs` |
| 测地距离 | `heat_geodesic`、`exact_geodesic` `[MEX]` |
| 布尔 / CSG | `mesh_boolean` `[MEX]`、`mesh_boolean_winding_number` |
| 距离场 / 内外 | `signed_distance` `[MEX]`、`df_build`/`df_query` |
| 2D 三角剖分 | `triangle`（wrapper）、`triangulate` `[MEX]` |
| 四面体化 | `tetgen`（wrapper）、`tetrahedralize` `[MEX]` |
| 网格清理 | `clean_mesh`、`selfintersect` `[MEX]` |

---

## 10. 验证说明 / Verification Notes

- 本文所有“函数名”均在仓库中确认存在（用 Glob 按文件名核对，如
  [mesh/lscm.m](file:///c:/VSWork/gptoolbox-1/mesh/lscm.m)、
  [matrix/min_quad_with_fixed.m](file:///c:/VSWork/gptoolbox-1/matrix/min_quad_with_fixed.m)、
  [wrappers/tetgen.m](file:///c:/VSWork/gptoolbox-1/wrappers/tetgen.m)）。
- 缩写表中 `UDQ` 经全库检索（`\budq\b`）**无匹配**，故标注“库内未实现”。
- 本表未列出的术语一律**不虚构**函数名；如需精确行号锚点，请转至对应模块文档
  （`10-mesh`…`60-mex`）的逐函数条目。

> 上一篇：[05-dependency-map.md](05-dependency-map.md)；项目总览请回到
> [01-project-overview.md](01-project-overview.md)。
