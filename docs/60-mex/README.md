# 60 mex 模块 / Compiled C++ MEX Module

> `mex/` 是 gptoolbox 中**唯一需要编译**的模块：57 个 `.m` 接口文件对应同名 `.cpp`
> 实现，依赖 Eigen / libigl，部分依赖 CGAL / Embree / cyCodeBase / libccd / ElTopo。
> 本目录下的 4 篇章节对 **57 个函数逐条**给出：桩与实现双锚点、签名、输入/输出/可选参数、
> 变量流动链、算法步骤、Mermaid 流程图、三方依赖（gptoolbox 函数 / 捆绑 C++ 库 / MATLAB 工具箱）
> 与 CMake 构建说明。
>
> `mex/` is the only module of gptoolbox that must be **compiled**: 57 `.m` interface files
> pair with same-named `.cpp` implementations, depending on Eigen / libigl and optionally
> CGAL / Embree / cyCodeBase / libccd / ElTopo. The four chapters in this folder document
> **all 57 functions** one by one: stub + implementation anchors, signature, inputs/outputs/
> options, variable-flow chain, algorithm steps, Mermaid flowchart, three-way dependency
> breakdown (gptoolbox / bundled C++ lib / MATLAB toolbox) and CMake build notes.

- 语言 / Language：中英对照（中文为主）
- 覆盖 / Coverage：`mex/*.m` **57 / 57**（100%）
- 粒度 / Granularity：逐函数 API + 算法 + 流程图 + 变量流动 + 构建与依赖

---

## 1. 章节结构 / Chapter layout

| 章节 | 文件 | 函数数 | 主题 |
|---|---|---|---|
| 60.1 | [01-booleans-and-csg.md](01-booleans-and-csg.md) | 9 | 布尔运算（CSG）与实体建模 / 网格修复 |
| 60.1b | [01b-tet-and-tri-meshing.md](01b-tet-and-tri-meshing.md) | 7 | 四面体与三角形网格生成、细分、简化 |
| 60.2 | [02-distance-queries-and-intersection.md](02-distance-queries-and-intersection.md) | 27 | 距离查询、卷绕数、测地线、相交测试与谓词 |
| 60.3 | [03-numerics-io-and-misc.md](03-numerics-io-and-misc.md) | 14 | 数值内核、I/O 与杂项 |
| — | **合计 / Total** | **57** | — |

---

## 2. 重要前提：`.m` 是桩 / Key premise: the `.m` files are stubs

按 [CLAUDE.md](../../CLAUDE.md) 的约定，`mex/<name>.m` **只含 `%` 注释头**，没有
`function` 行、没有代码，唯一作用是让 `help <name>` 在编译前可用；真实实现在同名
`mex/<name>.cpp`。因此本篇文档每条目都给出**两个锚点**。

**例外（`.m` 不是纯桩）/ Exceptions:**

| 文件 | 实情 |
|---|---|
| [winding_number.m](file:///c:/VSWork/gptoolbox-1/mex/winding_number.m) | **纯 MATLAB 回退实现**（含 `function` 行，调用 `solid_angle`），功能集小于 `.cpp` |
| [readMSH.m](file:///c:/VSWork/gptoolbox-1/mex/readMSH.m) | 含 `function` 行的纯 MATLAB 回退，**函数体整体被注释**，实际不返回结果 |
| [psd_project_rows.m](file:///c:/VSWork/gptoolbox-1/mex/psd_project_rows.m) | 含 `function` 行的纯 MATLAB 回退，支持 `tol`（`.cpp` 忽略 `tol`） |
| [fit_rotations_mex.m](file:///c:/VSWork/gptoolbox-1/mex/fit_rotations_mex.m) | 3 行**存在性探测**辅助函数（返回 `false`），非接口桩 |
| [gptoolbox_mexopts.m](file:///c:/VSWork/gptoolbox-1/mex/gptoolbox_mexopts.m) | **普通 MATLAB 函数**（生成编译选项），非 MEX |

---

## 3. 全量函数索引 / Complete function index

锚点列指向对应章节条目；库依赖列为该 `.cpp` 实际 `#include` / CMake 目标所依赖的捆绑 C++ 库。

### 3.1 布尔运算与实体建模（60.1，9 个）/ Booleans and CSG

| 函数 | 库依赖 | 一句话功能 |
|---|---|---|
| [`mesh_boolean`](file:///c:/VSWork/gptoolbox-1/mex/mesh_boolean.m) | libigl `copyleft/cgal`（CGAL Epeck） | 两个网格的并/交/差/对称差布尔运算 |
| [`trim_with_solid`](file:///c:/VSWork/gptoolbox-1/mex/trim_with_solid.m) | libigl `copyleft/cgal` | 用实体边界网格裁剪给定网格 |
| [`outer_hull`](file:///c:/VSWork/gptoolbox-1/mex/outer_hull.m) | libigl `copyleft/cgal` | 计算网格的"外 Hull"并标注朝向 |
| [`upper_envelope`](file:///c:/VSWork/gptoolbox-1/mex/upper_envelope.m) | CGAL Epeck + libigl | m 个分段线性函数的上包络 |
| [`selfintersect`](file:///c:/VSWork/gptoolbox-1/mex/selfintersect.m) | libigl `copyleft/cgal` | 自交检测与自交重网格 |
| [`split_nonmanifold`](file:///c:/VSWork/gptoolbox-1/mex/split_nonmanifold.m) | libigl core | 非流形网格按连通片拆分 |
| [`reorient_facets`](file:///c:/VSWork/gptoolbox-1/mex/reorient_facets.m) | Embree（`reorient_facets_raycast`） | 射线投票统一面片朝向 |
| [`snap_rounding`](file:///c:/VSWork/gptoolbox-1/mex/snap_rounding.m) | libigl `copyleft/cgal` + CGAL | 线段集合的整数格点舍入 |
| [`segment_graph`](file:///c:/VSWork/gptoolbox-1/mex/segment_graph.m) | 内联 Felzenszwalb–Huttenlocher + libigl + Eigen | 基于图的贪心图像分割 |

### 3.2 四面体与三角形网格化（60.1b，7 个）/ Tet & Tri meshing

| 函数 | 库依赖 | 一句话功能 |
|---|---|---|
| [`tetrahedralize`](file:///c:/VSWork/gptoolbox-1/mex/tetrahedralize.m) | **TetGen** + libigl + Eigen | 四面体网格化（Delaunay / 受限） |
| [`triangulate`](file:///c:/VSWork/gptoolbox-1/mex/triangulate.m) | **Triangle** + **CGAL** + libigl + Eigen | 受限 Delaunay 三角化 |
| [`refine_tetrahedralization`](file:///c:/VSWork/gptoolbox-1/mex/refine_tetrahedralization.m) | **TetGen** + libigl | 四面体网格细分（TetGen 重建） |
| [`refine_triangulation`](file:///c:/VSWork/gptoolbox-1/mex/refine_triangulation.m) | **Triangle** + libigl | 三角剖分细分（Triangle 重建） |
| [`marching_tets`](file:///c:/VSWork/gptoolbox-1/mex/marching_tets.m) | libigl + Eigen | 四面体等值面提取（Marching Tetrahedra） |
| [`decimate_libigl`](file:///c:/VSWork/gptoolbox-1/mex/decimate_libigl.m) | libigl + Eigen | 网格简化（decimate / qslim / progressive hulls） |
| [`collapse_small_triangles`](file:///c:/VSWork/gptoolbox-1/mex/collapse_small_triangles.m) | libigl + Eigen | 折叠删除小三角形 |

### 3.3 距离查询与相交（60.2，27 个）/ Distance queries & intersection

| 函数 | 库依赖 | 一句话功能 |
|---|---|---|
| [`signed_distance`](file:///c:/VSWork/gptoolbox-1/mex/signed_distance.m) | libigl + Eigen（Embree/CGAL 可选） | 点到网格的有符号距离 |
| [`signed_distance_isosurface`](file:///c:/VSWork/gptoolbox-1/mex/signed_distance_isosurface.m) | libigl `copyleft/cgal` + CGAL | 有符号距离场的等值面提取 |
| [`point_mesh_squared_distance`](file:///c:/VSWork/gptoolbox-1/mex/point_mesh_squared_distance.m) | libigl `copyleft/cgal` + CGAL | 点到网格的平方距离（AABB 树） |
| [`point_cubic_squared_distance`](file:///c:/VSWork/gptoolbox-1/mex/point_cubic_squared_distance.m) | cyCodeBase | 点到单条三次 Bézier 的平方距离 |
| [`point_spline_squared_distance`](file:///c:/VSWork/gptoolbox-1/mex/point_spline_squared_distance.m) | cyCodeBase | 点到样条集合的平方距离 |
| [`point_spline_signed_distance`](file:///c:/VSWork/gptoolbox-1/mex/point_spline_signed_distance.m) | cyCodeBase + libigl predicates | 点到样条集合的有符号距离 |
| [`simplex_simplex_squared_distance`](file:///c:/VSWork/gptoolbox-1/mex/simplex_simplex_squared_distance.m) | libigl | 两单纯形之间的平方距离 |
| [`winding_number`](file:///c:/VSWork/gptoolbox-1/mex/winding_number.m) | libigl + Embree（CGAL 可选） | 点相对网格的广义卷绕数 |
| [`cubic_winding_number`](file:///c:/VSWork/gptoolbox-1/mex/cubic_winding_number.m) | libigl predicates | 单条三次 Bézier 曲线的卷绕数 |
| [`spline_winding_number`](file:///c:/VSWork/gptoolbox-1/mex/spline_winding_number.m) | cyCodeBase + libigl predicates | 三次 Bézier 样条集合的广义卷绕数 |
| [`exact_geodesic`](file:///c:/VSWork/gptoolbox-1/mex/exact_geodesic.m) | libigl | 三角网格上的精确测地距离（MMP） |
| [`ray_mesh_intersect`](file:///c:/VSWork/gptoolbox-1/mex/ray_mesh_intersect.m) | Embree | 射线-网格首个命中 |
| [`ray_mesh_intersect_all`](file:///c:/VSWork/gptoolbox-1/mex/ray_mesh_intersect_all.m) | Embree | 射线-网格全部命中 |
| [`box_intersect`](file:///c:/VSWork/gptoolbox-1/mex/box_intersect.m) | CGAL | 包围盒两两/自相交检测 |
| [`gjk_intersect`](file:///c:/VSWork/gptoolbox-1/mex/gjk_intersect.m) | libccd | GJK 凸包相交测试 |
| [`gjk_penetration`](file:///c:/VSWork/gptoolbox-1/mex/gjk_penetration.m) | libccd | GJK+EPA 穿透深度/分离方向/接触点 |
| [`mpr_intersect`](file:///c:/VSWork/gptoolbox-1/mex/mpr_intersect.m) | libccd | MPR 凸包相交测试 |
| [`mpr_penetration`](file:///c:/VSWork/gptoolbox-1/mex/mpr_penetration.m) | libccd | MPR 穿透深度/分离方向/接触点 |
| [`orient2d`](file:///c:/VSWork/gptoolbox-1/mex/orient2d.m) | libigl predicates | 2D 精确定向谓词（Shewchuk） |
| [`orient3d`](file:///c:/VSWork/gptoolbox-1/mex/orient3d.m) | libigl predicates | 3D 精确定向谓词（Shewchuk） |
| [`in_element_aabb`](file:///c:/VSWork/gptoolbox-1/mex/in_element_aabb.m) | libigl | AABB 树点-单元包含判定 |
| [`intersect_other`](file:///c:/VSWork/gptoolbox-1/mex/intersect_other.m) | libigl `copyleft/cgal` + CGAL | 两网格之间的相交面对检测与重建 |
| [`ambient_occlusion`](file:///c:/VSWork/gptoolbox-1/mex/ambient_occlusion.m) | Embree | 逐点环境光遮蔽 |
| [`form_factor`](file:///c:/VSWork/gptoolbox-1/mex/form_factor.m) | CGAL（Epick） | 辐射度形状因子计算 |
| [`isolines`](file:///c:/VSWork/gptoolbox-1/mex/isolines.m) | libigl | 三角网格标量场等值线 |
| [`isolines_intrinsic`](file:///c:/VSWork/gptoolbox-1/mex/isolines_intrinsic.m) | libigl | 固有（无坐标）等值线 |
| [`wire_mesh`](file:///c:/VSWork/gptoolbox-1/mex/wire_mesh.m) | libigl `copyleft/cgal` + CGAL | 由线网生成线框/支柱曲面 |

### 3.4 数值内核、I/O 与杂项（60.3，14 个）/ Numeric kernels, I/O & misc

| 函数 | 库依赖 | 一句话功能 |
|---|---|---|
| [`fast_roots`](file:///c:/VSWork/gptoolbox-1/mex/fast_roots.m) | cyCodeBase + Eigen + libigl | 向量化多项式求根（可按区间过滤） |
| [`fast_sparse`](file:///c:/VSWork/gptoolbox-1/mex/fast_sparse.m) | Eigen(Sparse) + libigl | 更快的稀疏矩阵构造 |
| [`psd_project_rows`](file:///c:/VSWork/gptoolbox-1/mex/psd_project_rows.m) | Eigen | 逐行半正定投影 |
| [`fit_rotations_mex`](file:///c:/VSWork/gptoolbox-1/mex/fit_rotations_mex.m) | libigl | 堆叠旋转块的最小二乘拟合 |
| [`fit_cubic_bezier`](file:///c:/VSWork/gptoolbox-1/mex/fit_cubic_bezier.m) | libigl | G1 连续三次 Bézier 拟合 |
| [`principal_curvature`](file:///c:/VSWork/gptoolbox-1/mex/principal_curvature.m) | libigl | 顶点主曲率（二次曲面拟合） |
| [`slim`](file:///c:/VSWork/gptoolbox-1/mex/slim.m) | libigl | SLIM 局部单射参数化 |
| [`dual_laplacian`](file:///c:/VSWork/gptoolbox-1/mex/dual_laplacian.m) | Eigen | 四面体网格对偶拉普拉斯 |
| [`eltopo`](file:///c:/VSWork/gptoolbox-1/mex/eltopo.m) | ElTopo + BLAS + libigl | El Topo 碰撞感知重网格 |
| [`readMSH`](file:///c:/VSWork/gptoolbox-1/mex/readMSH.m) | libigl | 读取 gmsh `.msh` 四面体网格 |
| [`read_mesh_from_xml`](file:///c:/VSWork/gptoolbox-1/mex/read_mesh_from_xml.m) | libigl(xml) + CGAL + tinyxml2 | 读取 XML 中序列化的网格 |
| [`impaste`](file:///c:/VSWork/gptoolbox-1/mex/impaste.m) | macOS Foundation/AppKit（`paste.mm`） | 从剪贴板粘贴图像 |
| [`blue_noise`](file:///c:/VSWork/gptoolbox-1/mex/blue_noise.m) | libigl | 蓝噪声（Poisson 盘）采样 |
| [`gptoolbox_mexopts`](file:///c:/VSWork/gptoolbox-1/mex/gptoolbox_mexopts.m) | —（普通 MATLAB 函数） | 生成 `mex` 编译选项 |

---

## 4. 依赖分布 / Dependency distribution

| C++ 库 | 出现次数 | 涉及函数（示例） |
|---|---|---|
| Eigen | 多处 | `fast_sparse`、`psd_project_rows`、`dual_laplacian`、`fast_roots` |
| libigl（core） | 多数 | `simplex_simplex_squared_distance`、`marching_tets`、`isolines`、`slim` |
| libigl `copyleft/cgal` + CGAL | 11 | `mesh_boolean`、`trim_with_solid`、`outer_hull`、`selfintersect`、`snap_rounding`、`signed_distance_isosurface`、`point_mesh_squared_distance`、`intersect_other`、`wire_mesh`、`upper_envelope`、`box_intersect` |
| Embree | 6 | `reorient_facets`、`ray_mesh_intersect`、`ray_mesh_intersect_all`、`ambient_occlusion`、`winding_number`、`signed_distance`（可选） |
| cyCodeBase | 4 | `fast_roots`、`point_cubic_squared_distance`、`point_spline_*`、`spline_winding_number` |
| libccd | 4 | `gjk_intersect`、`gjk_penetration`、`mpr_intersect`、`mpr_penetration` |
| 外部网格器 | 4 | `tetrahedralize` / `refine_tetrahedralization`（TetGen）、`triangulate` / `refine_triangulation`（Triangle） |
| ElTopo | 1 | `eltopo` |
| tinyxml2 | 1 | `read_mesh_from_xml` |
| 平台 API | 1 | `impaste`（仅 macOS 构建） |

构建方式（CMake，非 `mex` 命令直编）：

```bash
cd mex
mkdir build && cd build
cmake ..
make
```

若 CMake 找不到 MATLAB，传 `-DMatlab_ROOT_DIR=<path>`；若报未知 MATLAB 版本，
更新 `mex/cmake/FindMATLAB.cmake` 中的 `MATLAB_VERSIONS_MAPPING`。
详见 [mex/README.md](file:///c:/VSWork/gptoolbox-1/mex/README.md) 与
[00-overview/03-build-and-install.md](../00-overview/03-build-and-install.md)。

---

## 5. 文档过程中发现的源码问题 / Source issues found while documenting

以下均为**对照 `.m` 桩与 `.cpp` 实现**后确认的偏差，已写入各章节条目的「备注」，此处汇总：

| 类别 | 实例 |
|---|---|
| 桩的输出数与实现不符 | `tetrahedralize`（桩 7 / 实现 8，含越界 `plhs+8`）、`outer_hull`（桩 3 / 实现 4）、`upper_envelope`（桩 4 / 实现 6）、`point_spline_squared_distance`、`point_spline_signed_distance`（桩 8 / 实现 7）、`readMSH`（桩 3 / 实现 5）、`impaste`（桩 1 / 实现 2） |
| 未文档化的选项 | `tetrahedralize` 的 `'Holes'`、`triangulate` 的 `'Method'`、`slim` 的 `'AlignGuess'`、`blue_noise` 的 2D 分支选项、`wire_mesh` 的 `'Solid'`、`mesh_boolean` 的 `'WindingNumberFilter'`/`'Debug'` |
| 死代码 / 未实现分支 | `mesh_boolean` 的 Cork 分支（`WITH_CORK` 从未定义）、`eltopo` 的 `G`（实为 `G = F`）、`fit_rotations_mex` 的 `'AllowFlips'` |
| 潜在缺陷 | `triangulate` 的 `case CGAL_EPECK` 缺 `break`（贯穿覆盖结果）、`snap_rounding` 无 `nrhs` 校验、`segment_graph` 未知参数不报错 + `Threshold` 存 `int` 截断、`decimate_libigl` 的 `V.cols()==3` 校验被注释 |
| 构建脚本不一致 | `cubic_winding_number.cpp` **未列入** `mex/CMakeLists.txt` 编译目标（疑似遗漏） |
| `.m` 非桩 | `winding_number.m`、`readMSH.m`、`psd_project_rows.m`（纯 MATLAB 回退）、`fit_rotations_mex.m`（探测函数）、`gptoolbox_mexopts.m`（普通函数） |
| 陈旧/错误注释 | `tetrahedralize.m` 示例误写 `tetgen`；`mesh_boolean.m` 的 `See also: self_intersect` 名称有误；`selfintersect` 提及的 `selfintersect.h` 不存在 |

---

## 6. 交叉引用 / Cross references

- 项目总览与架构：[00-overview/01-project-overview.md](../00-overview/01-project-overview.md)、
  [00-overview/02-architecture.md](../00-overview/02-architecture.md)
- 构建与安装（MEX/vcpkg/CGAL/Embree）：[00-overview/03-build-and-install.md](../00-overview/03-build-and-install.md)、
  [mex/README.md](file:///c:/VSWork/gptoolbox-1/mex/README.md)
- MEX 的 MATLAB 侧调用方（距离/测地线）：[10-mesh/13-distances-and-geodesics.md](../10-mesh/13-distances-and-geodesics.md)
- 外部程序封装（TetGen/Triangle/QSlim 的命令行版）：[50-wrappers/01-external-binary-wrappers.md](../50-wrappers/01-external-binary-wrappers.md)
- 文档写作规范：[appendices/03-doc-template.md](../appendices/03-doc-template.md)
