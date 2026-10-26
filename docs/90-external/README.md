# 90 external 模块 / Vendored Third-Party Code

> `external/` 是 gptoolbox 中**第三方代码的汇集区**：作者把不属于本工具箱、但被依赖或提供
> 便利功能的 MATLAB 代码原样收入，来源包括 Gabriel Peyre 的 *Toolbox Fast Marching*、
> Danil Kirsanov 的 *Exact Geodesics on Triangulated Surfaces*、Charles Robert 的
> *ColorBrewer*、以及大量 MATLAB File Exchange 单文件工具。
>
> 本目录下的 13 篇章节对 **external/ 全部 197 个 `.m` 文件逐条**给出：可点击文件锚点、
> 署名与许可、签名、输入/输出/可选参数、变量流动链、算法步骤、Mermaid 流程图、依赖
> 与 See also，并在每章末尾附「函数清单核对表」。
>
> `external/` collects **vendored third-party MATLAB code**: *Toolbox Fast Marching*
> (Gabriel Peyre), *Exact Geodesics on Triangulated Surfaces* (Danil Kirsanov),
> *ColorBrewer* (Charles Robert) and many single-file MATLAB File Exchange utilities.
> The 13 chapters here document **all 197 `.m` files** one by one with anchors,
> attribution, signature, I/O tables, variable-flow chain, algorithm steps, a Mermaid
> flowchart, dependencies and See also; each chapter ends with a coverage checklist.

- 语言 / Language：中英对照（中文为主）
- 覆盖 / Coverage：`external/**/*.m` **197 / 197**（100%）
- 粒度 / Granularity：逐文件 API + 算法 + 流程图 + 变量流动 + 署名/许可 + 遮蔽与重复分析

---

## 1. 章节结构 / Chapter layout

| 章节 | 文件 | 函数数 | 主题 |
|---|---|---|---|
| 90.1 | [01-colormaps-and-color.md](01-colormaps-and-color.md) | 11 | 配色（ColorBrewer、matplotlib 四色图）与颜色空间转换 |
| 90.2 | [02-numeric-linear-algebra.md](02-numeric-linear-algebra.md) | 10 | 数值线性代数辅助 |
| 90.2b | [02b-geometry-and-format-helpers.md](02b-geometry-and-format-helpers.md) | 16 | 几何/拓扑辅助、PLY/VRML 读写、四元数 |
| 90.3 | [03-matlab-environment-and-plotting.md](03-matlab-environment-and-plotting.md) | 11 | MATLAB 环境与绘图辅助 |
| 90.4 | [04-exactgeodesic-toolbox.md](04-exactgeodesic-toolbox.md) | 17 | Kirsanov 精确测地线工具箱（句柄式 API） |
| 90.5 | [05-fast-marching-solvers.md](05-fast-marching-solvers.md) | 16 | Fast Marching / Fast Sweeping 求解器核心 |
| 90.5b | [05b-fast-marching-compute-helpers.md](05b-fast-marching-compute-helpers.md) | 20 | Fast Marching 派生量计算 |
| 90.5c | [05c-fast-marching-plotting-and-tools.md](05c-fast-marching-plotting-and-tools.md) | 16 | Fast Marching 绘图、交互与批处理 |
| 90.6a | [06a-fast-marching-internals-data-structures.md](06a-fast-marching-internals-data-structures.md) | 17 | `toolbox/`：数据结构与选项解析 |
| 90.6b | [06b-fast-marching-internals-operators-and-io.md](06b-fast-marching-internals-operators-and-io.md) | 16 | `toolbox/`：算子、网格 I/O 与进度显示 |
| 90.7a | [07a-fast-marching-tests-a.md](07a-fast-marching-tests-a.md) | 23 | `tests/`：测试与示例 A |
| 90.7b | [07b-fast-marching-tests-b.md](07b-fast-marching-tests-b.md) | 22 | `tests/`：测试与示例 B |
| 90.8 | [08-fast-marching-volume-rendering-and-anisotropic-demo.md](08-fast-marching-volume-rendering-and-anisotropic-demo.md) | 2 | 体渲染 `vol3d` 与各向异性 FMM 示例 |
| — | **合计 / Total** | **197** | — |

---

## 2. 目录构成与覆盖核对 / Directory composition and coverage

| 目录 | `.m` 数 | 覆盖章节 | 核对 |
|---|---|---|---|
| `external/`（根） | 48 | 90.1(11) + 90.2(10) + 90.2b(16) + 90.3(11) | 11+10+16+11 = 48 ✓ |
| `external/exactgeodesic/` | 17 | 90.4 | 17 ✓ |
| `external/toolbox_fast_marching/`（根） | 53 | 90.5(16) + 90.5b(20) + 90.5c(16) + 90.8(1) | 16+20+16+1 = 53 ✓ |
| `external/toolbox_fast_marching/toolbox/` | 33 | 90.6a(17) + 90.6b(16) | 17+16 = 33 ✓ |
| `external/toolbox_fast_marching/tests/` | 45 | 90.7a(23) + 90.7b(22) | 23+22 = 45 ✓ |
| `external/toolbox_fast_marching/mex/anisotropic-fm-feth/` | 1 | 90.8 | 1 ✓ |
| **总计 / Total** | **197** | 13 篇 | **197 ✓** |

> 注：`toolbox_fast_marching/mex/` 下的其余文件为 C/C++（`perform_front_propagation_*.cpp`、
> `skeleton.cpp`、`eucdist2.c`、`fheap/`、`gw/` 及 `*.vcproj`/`*.sln`），不属 `.m` 计价范围，
> 已在 [90.5](05-fast-marching-solvers.md) 与 [90.5c](05c-fast-marching-plotting-and-tools.md) 的
> `compile_mex` 条目中说明其构建方式。

---

## 3. 全量函数索引 / Complete function index

### 3.1 `external/` 根目录（48 个）/ Repository root of `external/`

**配色与颜色（90.1）**

| 函数 | 署名 | 一句话功能 |
|---|---|---|
| [`cbrewer`](file:///c:/VSWork/gptoolbox-1/external/cbrewer.m) | Charles Robert（BSD） | 生成 ColorBrewer 配色方案的 n 色 RGB 表 |
| [`interpolate_cbrewer`](file:///c:/VSWork/gptoolbox-1/external/interpolate_cbrewer.m) | Charles Robert（BSD） | 对既有配色表做插值加密 |
| [`plot_brewer_cmap`](file:///c:/VSWork/gptoolbox-1/external/plot_brewer_cmap.m) | Charles Robert（BSD） | 展示 ColorBrewer 配色 |
| [`viridis`](file:///c:/VSWork/gptoolbox-1/external/viridis.m) | Ander Biguri（BSD，数据出自 matplotlib） | matplotlib `viridis` 色图 |
| [`inferno`](file:///c:/VSWork/gptoolbox-1/external/inferno.m) | Ander Biguri（BSD） | matplotlib `inferno` 色图 |
| [`magma`](file:///c:/VSWork/gptoolbox-1/external/magma.m) | Ander Biguri（BSD） | matplotlib `magma` 色图 |
| [`plasma`](file:///c:/VSWork/gptoolbox-1/external/plasma.m) | Ander Biguri（BSD） | matplotlib `plasma` 色图 |
| [`cmyk2rgb`](file:///c:/VSWork/gptoolbox-1/external/cmyk2rgb.m) | 无可考署名 | CMYK → RGB（需 IPT） |
| [`rgb2cmyk`](file:///c:/VSWork/gptoolbox-1/external/rgb2cmyk.m) | 无可考署名 | RGB → CMYK（需 IPT） |
| [`hex2rgb`](file:///c:/VSWork/gptoolbox-1/external/hex2rgb.m) | Chad A. Greene | 十六进制颜色串 → RGB |
| [`rgb2hex`](file:///c:/VSWork/gptoolbox-1/external/rgb2hex.m) | Chad A. Greene | RGB → 十六进制颜色串 |

**数值线性代数（90.2）**

| 函数 | 署名 | 一句话功能 |
|---|---|---|
| [`affine_fit`](file:///c:/VSWork/gptoolbox-1/external/affine_fit.m) | Adrien Leygue（BSD 式） | 最小二乘仿射映射 `y = A x + b` 拟合 |
| [`inv2`](file:///c:/VSWork/gptoolbox-1/external/inv2.m) | Bruno Luong | 2×2 块矩阵快速求逆 |
| [`inv3`](file:///c:/VSWork/gptoolbox-1/external/inv3.m) | Bruno Luong | 3×3 块矩阵快速求逆 |
| [`luq`](file:///c:/VSWork/gptoolbox-1/external/luq.m) | Pawel Kowal（非商业学术） | 带列置换的 LU 分解 |
| [`spnull`](file:///c:/VSWork/gptoolbox-1/external/spnull.m) | Jan Schellenberger | 稀疏零空间基（`spnull → spspaces → luq`） |
| [`spspaces`](file:///c:/VSWork/gptoolbox-1/external/spspaces.m) | Pawel Kowal（非商业学术） | 稀疏四基本子空间 |
| [`combn`](file:///c:/VSWork/gptoolbox-1/external/combn.m) | Jos van der Geest（BSD） | 笛卡尔幂式组合枚举 |
| [`trandn`](file:///c:/VSWork/gptoolbox-1/external/trandn.m) | Zdravko I. Botev（BSD 式） | 截断正态分布采样 |
| [`ssim`](file:///c:/VSWork/gptoolbox-1/external/ssim.m) | Zhou Wang 等（仅教育研究） | SSIM 结构相似度（含自动降采样） |
| [`ssim_index`](file:///c:/VSWork/gptoolbox-1/external/ssim_index.m) | Zhou Wang 等（仅教育研究） | SSIM 结构相似度（原始实现） |

**几何与格式（90.2b）**

| 函数 | 署名 | 一句话功能 |
|---|---|---|
| [`dijk`](file:///c:/VSWork/gptoolbox-1/external/dijk.m) | Michael G. Kay | Dijkstra 最短路 |
| [`dpsimplify`](file:///c:/VSWork/gptoolbox-1/external/dpsimplify.m) | Wolfgang Schwanghart | Ramer–Douglas–Peucker 折线简化 |
| [`geodome`](file:///c:/VSWork/gptoolbox-1/external/geodome.m) | Peter Kovesi | 测地穹顶网格生成 |
| [`icosahedron`](file:///c:/VSWork/gptoolbox-1/external/icosahedron.m) | Peter Gagarinov | 正二十面体棱边 |
| [`hilbert2`](file:///c:/VSWork/gptoolbox-1/external/hilbert2.m) | Ivan Martynov / Federico Forte | 2-D Hilbert 空间填充曲线索引映射 |
| [`hilbert3`](file:///c:/VSWork/gptoolbox-1/external/hilbert3.m) | Ivan Martynov / Federico Forte | 3-D Hilbert 空间填充曲线索引映射 |
| [`isocontour_off`](file:///c:/VSWork/gptoolbox-1/external/isocontour_off.m) | D. Kroon | 图像网格上的 Marching Squares 等值线 |
| [`lineSegmentIntersect`](file:///c:/VSWork/gptoolbox-1/external/lineSegmentIntersect.m) | U. Murat Erdem | 2-D 线段两两相交测试（向量化） |
| [`mask2poly`](file:///c:/VSWork/gptoolbox-1/external/mask2poly.m) | Nikolay S. | 二值掩膜边界轮廓追踪 |
| [`Hungarian`](file:///c:/VSWork/gptoolbox-1/external/Hungarian.m) | Alex Melin | Kuhn–Munkres 指派问题求解 |
| [`read_ply`](file:///c:/VSWork/gptoolbox-1/external/read_ply.m) | Gabriel Peyre + Pascal Getreuer | 读取 PLY 网格 |
| [`write_ply`](file:///c:/VSWork/gptoolbox-1/external/write_ply.m) | Gabriel Peyre + Pascal Getreuer | 写出 PLY 网格 |
| [`vrml`](file:///c:/VSWork/gptoolbox-1/external/vrml.m) | HUMUSOFT / MathWorks | VRML 场景写出 |
| [`msm_to_hb`](file:///c:/VSWork/gptoolbox-1/external/msm_to_hb.m) | Xiaoye Li | MATLAB 稀疏矩阵 → Harwell–Boeing |
| [`newquaternion`](file:///c:/VSWork/gptoolbox-1/external/newquaternion.m) | Peter Kovesi | 构造四元数 `[w x y z]` |
| [`quaternionrotate`](file:///c:/VSWork/gptoolbox-1/external/quaternionrotate.m) | Peter Kovesi | 用四元数旋转向量 |

**环境与绘图（90.3）**

| 函数 | 署名 | 一句话功能 |
|---|---|---|
| [`cprintf`](file:///c:/VSWork/gptoolbox-1/external/cprintf.m) | Yair Altman（BSD 式） | 命令行彩色输出 |
| [`GetFullPath`](file:///c:/VSWork/gptoolbox-1/external/GetFullPath.m) | Jan Simon（BSD） | 路径绝对化 |
| [`progressbar`](file:///c:/VSWork/gptoolbox-1/external/progressbar.m) | Gabriel Peyre | 文本进度条（`w=80`，`□`/`■`） |
| [`tabcomplete`](file:///c:/VSWork/gptoolbox-1/external/tabcomplete.m) | Yair Altman | 命令行 Tab 补全 |
| [`preserve_history`](file:///c:/VSWork/gptoolbox-1/external/preserve_history.m) | Bogdan Roman | 保存/恢复命令历史 |
| [`timeit`](file:///c:/VSWork/gptoolbox-1/external/timeit.m) | Steve Eddins / MathWorks（BSD） | 函数计时（**遮蔽内置 `timeit`**） |
| [`myaa`](file:///c:/VSWork/gptoolbox-1/external/myaa.m) | Anders Brun（BSD） | 超采样抗锯齿截图 |
| [`saveTightFigure`](file:///c:/VSWork/gptoolbox-1/external/saveTightFigure.m) | Emre Akbas（BSD） | 紧凑裁剪保存图窗 |
| [`tilefigs`](file:///c:/VSWork/gptoolbox-1/external/tilefigs.m) | Charles Plum | 平铺所有图窗 |
| [`patcht`](file:///c:/VSWork/gptoolbox-1/external/patcht.m) | D.-J. Kroon | 带纹理贴图的 `patch` |
| [`exactgeodesic`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic.m) | 无可考署名（封装 Kirsanov 库，MIT） | 精确测地距离函数式封装 |

### 3.2 `external/exactgeodesic/`（17 个，90.4）

| 函数 | 类别 | 一句话功能 |
|---|---|---|
| [`geodesic_new_mesh`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/geodesic_new_mesh.m) | 句柄 | 上传网格，返回 `mesh_id` |
| [`geodesic_new_algorithm`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/geodesic_new_algorithm.m) | 句柄 | 创建算法实例，返回 `algo_id` |
| [`geodesic_propagate`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/geodesic_propagate.m) | 计算 | 从源点做 MMP 窗口传播 |
| [`geodesic_trace_back`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/geodesic_trace_back.m) | 计算 | 回溯出离散测地路径 |
| [`geodesic_distance_and_source`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/geodesic_distance_and_source.m) | 查询 | 查询距离与最近源（**文件内定义函数名为 `geodesic_best_source`**） |
| [`geodesic_create_surface_point`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/geodesic_create_surface_point.m) | 数据 | 构造面/边/顶点上的曲面点 |
| [`geodesic_convert_surface_points`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/geodesic_convert_surface_points.m) | 数据 | 曲面点集合 ↔ 坐标转换 |
| [`geodesic_delete`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/geodesic_delete.m) | 生存期 | 释放 mex 侧对象 |
| [`create_flat_triangular_mesh`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/create_flat_triangular_mesh.m) | 测试数据 | 构造平面三角网格 |
| [`create_hedgehog_mesh`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/create_hedgehog_mesh.m) | 测试数据 | 构造刺猬状测试网格 |
| [`create_subdivision_pattern`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/create_subdivision_pattern.m) | 测试数据 | 构造细分图案 |
| [`extract_coordinates_from_path`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/extract_coordinates_from_path.m) | 工具 | 从路径中抽取坐标 |
| [`example1`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/example1.m)–[`example5`](file:///c:/VSWork/gptoolbox-1/external/exactgeodesic/example5.m) | 示例 | 5 个端到端演示脚本（句柄流水线用法） |

### 3.3 `external/toolbox_fast_marching/`（132 个，90.5–90.8）

该子树按功能拆为 8 章，**各章末尾的「函数清单核对表」为逐文件权威列表**：

| 章 | 范围 | 数量 | 代表性函数 |
|---|---|---|---|
| [90.5](05-fast-marching-solvers.md) | 根目录 `perform_*` | 16 | `perform_fast_marching`、`perform_front_propagation_2d`、`perform_front_propagation_2d_slow`、`perform_fmstar_2d`/`_3d`、`perform_farthest_point_sampling(_mesh/_boundary)`、`perform_farthest_landmark_sampling`、`perform_lloyd_mesh`、`perform_redistancing`、`perform_active_contour`、`perform_geodesic_interpolation`、`perform_circular_fast_marching_2d`、`perform_fast_marching_old`、`perform_fast_marching_mesh` |
| [90.5b](05b-fast-marching-compute-helpers.md) | 根目录派生量 | 20 | `compute_distance_landmark`、`compute_geodesic(_mesh)`、`compute_eccentricity_transform`、`compute_bending_invariant`、`compute_edge_energy`、`compute_heuristic_landmark`、`compute_heuristic_multiresolution`、`compute_alpha_map`、`compute_levelset_shape`、`compute_saddle_points`、`compute_shape_boundary`、`compute_voronoi_triangulation(_mesh)`、`generate_constrained_map`、`load_potential_map`、`convert_distance_color`、`divgrad`、`eucdist2`、`callback_active_contour` |
| [90.5c](05c-fast-marching-plotting-and-tools.md) | 根目录绘图/交互/批处理 | 16 | `plot_fast_marching_2d`/`_3d`/`_mesh`、`plot_constrained_path_planing`、`plot_volumetric_data`、`display_eccentricity`、`display_segmentation`、`pick_curves`、`pick_start_end_point`、`batch_landmarks_error`、`batch_propagation_mesh`、`batch_shape_meshing`、`compile_mex`、`publish_html`、`content`、`toolbox_fast_marching` |
| [90.6a](06a-fast-marching-internals-data-structures.md) | `toolbox/` 数据结构与选项 | 17 | `getoptions`、`compute_edges`、`compute_vertex_ring`、`compute_edge_face_ring`、`check_face_vertex`、`compute_grad`、`compute_distance_to_points`、`compute_gaussian_filter`、`perform_blurring`、`clamp`、`crop`、`mmax`、`nb_dims`、`num2string_fixeddigit`、`load_image`、`imageplot`、`compute_cuvilinear_abscice` |
| [90.6b](06b-fast-marching-internals-operators-and-io.md) | `toolbox/` 算子与 I/O | 16 | `perform_conjugate_gradient`、`perform_convolution`、`perform_contour_extraction`、`perform_curve_extraction`、`perform_curve_resampling`、`perform_histogram_equalization`、`perform_image_resize`（函数名实为 `image_resize`）、`perform_tensor_recomp`、`perform_vf_normalization`、`plot_mesh`、`prod_vf_sf`、`progressbar`、`read_mesh`、`read_off`、`rescale`（**遮蔽内置**）、`triangulation2adjacency` |
| [90.7a](07a-fast-marching-tests-a.md) | `tests/` A | 23 | `test_active_contour`、`test_anisotropic*`（4）、`test_bending_invariants`、`test_circular*`（3）、`test_constrained_map`、`test_distance_approximation`、`test_distance_compression`、`test_eccentricity`、`test_eucldist`、`test_farthest_sampling_*`（4）、`test_fast_marching_2d`/`_3d`、`test_fmstar_2d`/`_3d`、`test_bug` |
| [90.7b](07b-fast-marching-tests-b.md) | `tests/` B | 22 | `test_fmstar_error`、`test_fmstar_landmark`、`test_fmstar_path_planing`、`test_fmstar_weight_2d`、`test_geodesic_interpolation`、`test_geodesic_vs_euclidean`、`test_heuristic_mesh`、`test_influence`、`test_landmark(_error)`、`test_multiple_paths_2d`/`_3d`、`test_path_planing`、`test_propagation_2d`/`_mesh`/`_shape`、`test_redistancing`、`test_segmentation`、`test_skeleton`、`test_vol3d`、`test_voronoi_segmentation`、`test_voronoi_triangulation` |
| [90.8](08-fast-marching-volume-rendering-and-anisotropic-demo.md) | 体渲染与各向异性示例 | 2 | `vol3d`、`mex/anisotropic-fm-feth/testFM2dAniso` |

---

## 4. 构建前提 / Build prerequisites

| 子树 | 是否需要编译 | 说明 |
|---|---|---|
| `external/` 根（48 个） | 否 | 纯 `.m`，`addpath` 即可用 |
| `external/exactgeodesic/` | 是（或使用随附二进制） | 依赖 `src/geodesic_matlab_api.cpp` 导出的动态库；随附 Windows `.dll` / macOS `.dylib`，`geodesic_new_mesh` 的加载分支**只有 `ispc`/`ismac`**，Linux 无分支 |
| `external/toolbox_fast_marching/` | 部分需要 | 运行 `compile_mex.m` 编译 `perform_front_propagation_2d`/`_3d`、`perform_circular_front_propagation_2d`、GW 版 `perform_front_propagation_mesh`、`skeleton`、`eucdist2`；2-D 有纯 MATLAB 回退（`*_slow.m`），3-D 与网格传播**无回退** |

---

## 5. 关键发现汇总 / Consolidated findings

### 5.1 遮蔽 MATLAB 内置 / Shadowing MATLAB builtins

| 文件 | 判定 | 说明 |
|---|---|---|
| `external/timeit.m` | **确认遮蔽** | 与 R2013b+ 内置 `timeit` 同名；额外返回 `measurement_overhead`/`measurement_details`。规避：`builtin('timeit',f)` |
| `toolbox_fast_marching/toolbox/rescale.m` | **确认遮蔽** | 与 R2017b+ 内置 `rescale` 同名；本版仅 `(x,a,b)` 形态，不支持沿维度与 `NaN` 处理 |
| `external/vrml.m` | **疑似遮蔽** | 与 MATLAB 旧版内置 `vrml`（已移除）同名 |
| `external/viridis|inferno|magma|plasma.m` | **不遮蔽** | MATLAB 仅内置 `turbo`（R2020b+），未内置这四个；重名风险来自 File Exchange |
| `vol3d`、`publish_html`、`convert_distance_color` 等 | 不遮蔽 | 无同名内置 |

### 5.2 函数名冲突 / Name collisions

| 名称 | 出现位置 | 差异 |
|---|---|---|
| `progressbar` | [`external/progressbar.m`](file:///c:/VSWork/gptoolbox-1/external/progressbar.m) 与 [`toolbox/progressbar.m`](file:///c:/VSWork/gptoolbox-1/external/toolbox_fast_marching/toolbox/progressbar.m) | 前者 `w=80`、`□`/`■`、无括号；后者 `w=20`、`'.'`/`'*'`、带 `[ ]`。同为 Gabriel Peyre，**由搜索路径顺序决定生效者** |

### 5.3 文件名 ≠ 函数名 / Filename ≠ function name

| 文件 | 实际定义的函数 |
|---|---|
| `external/exactgeodesic/geodesic_distance_and_source.m` | `geodesic_best_source`（但 `example*.m` 均按文件名调用 → 会失败） |
| `external/toolbox_fast_marching/toolbox/perform_image_resize.m` | `image_resize`（**不存在**独立的 `image_resize.m`） |

### 5.4 缺失依赖（源码调用但全库无定义）/ Undefined dependencies

`plot_tensor_field`、`pick_start_points`、`fm2dAniso`（仅 macOS 预编译）、`image_3d_resize`、
`compute_voronoi_mesh`、`compute_euclidean_distance`、`image_resize`、`matrix_sampling_get`、
`gen_signal`、`norme`、`plot_curve`、`plot_scattered`、`compute_hausdorff_distance`、
`get_color_from_index`、`perform_faces_reorientation`、`apply_colormap`、`plot_tensor_field`；
`read_mesh` 内部依赖的 `read_ply`/`read_smf`/`read_wrl`/`read_obj`/`read_tet`/`load_gim` 亦缺。
另有多个函数仅为 `perform_farthest_point_sampling.m` / `compute_geodesic.m` 的**局部子函数**
（`my_eval_distance`、`compute_discrete_geodesic`），外部脚本不可直接调用。

### 5.5 不可运行的测试脚本 / Non-runnable test scripts

45 个 `tests/*.m` 中，**约 21 个无法直接运行**：一部分因上述缺失函数，一部分因缺失数据
（`data/*.png`、`.off` 网格、`brain1-crop-256.mat`），还有 2 个因引用未定义变量
（`test_redistancing` 的 `X`、`test_voronoi_triangulation` 的 `vertex`）。详见
[90.7a](07a-fast-marching-tests-a.md) 与 [90.7b](07b-fast-marching-tests-b.md) 的核对表。

### 5.6 许可证混杂 / Mixed licensing

| 类别 | 实例 |
|---|---|
| BSD / 宽松 | ColorBrewer（Charles Robert）、`viridis` 系列（Ander Biguri）、`inv2`/`inv3`/`combn`/`trandn`、`GetFullPath`/`cprintf`/`myaa`/`saveTightFigure`、Toolbox Fast Marching（Gabriel Peyre）、Kirsanov 库（MIT） |
| **非商业/学术限定** | `luq`、`spspaces`（LREM_SOLVE，仅限非商业学术用途） |
| **仅教育/研究** | `ssim`、`ssim_index`（Zhou Wang 等） |
| 无声明 | `inv2`/`inv3`、`spnull`、`hex2rgb`/`rgb2hex`、`vrml`、`patcht`、`tilefigs`、`preserve_history`、`cmyk2rgb`/`rgb2cmyk`、`geodesic_*`（`.m` 层） |

> ⚠️ 使用 `luq`/`spspaces` 与 `ssim`/`ssim_index` 前需确认许可条件（前者限制商用，后者限制用途）。

### 5.7 已验证的一致性 / Verified consistency

- **四元数约定**：`external/newquaternion.m` 与 `external/quaternionrotate.m` 采用
  **标量在前 `[w x y z]`**，与 [`quat/quatmultiply.m`](file:///c:/VSWork/gptoolbox-1/quat/quatmultiply.m)
  及 [10-mesh/16](../10-mesh/16-rigid-transforms-and-quaternion.md) 的约定**一致**；
  唯一差异是 `newquaternion` 返回 **4×1 列向量**，而 `quatmultiply` 期望 **M×4 行式**，互用需转置。
- **内部依赖链**：`external/spnull → external/spspaces → external/luq`（唯一根目录内部调用链）。
- **`external/read_ply`/`write_ply` 与 `mesh/readPLY`/`mesh/writePLY`**：后者调用前者，属**封装**而非重复实现。
- **Toolbox Fast Marching 的选项惯用法**：全子树使用 `options` 结构体 + `getoptions`
  （`options.null=0` 起手），与 gptoolbox 主库的 `varargin` + Name/Value 风格**不同**。

---

## 6. 交叉引用 / Cross references

- 项目总览：[00-overview/01-project-overview.md](../00-overview/01-project-overview.md)、[00-overview/02-architecture.md](../00-overview/02-architecture.md)（`external/` 为架构 L0 层）
- 依赖全景：[00-overview/05-dependency-map.md](../00-overview/05-dependency-map.md)
- 与 gptoolbox 自有实现的对照：[10-mesh/13-distances-and-geodesics.md](../10-mesh/13-distances-and-geodesics.md)、[10-mesh/16-rigid-transforms-and-quaternion.md](../10-mesh/16-rigid-transforms-and-quaternion.md)、[40-utility/01-utilities-core.md](../40-utility/01-utilities-core.md)、[40-utility/02-plotting-and-colors.md](../40-utility/02-plotting-and-colors.md)
- 编译型后端对照：[60-mex/README.md](../60-mex/README.md)
- 文档写作规范：[appendices/03-doc-template.md](../appendices/03-doc-template.md)

---

## 7. 与他处索引的关系 / Relation to other indexes

本章为 **`external/` 的模块级索引**：第 3 节给出根目录与 `exactgeodesic/` 的完整函数表，
`toolbox_fast_marching/`（132 个）按 8 章给出代表性函数与章节定位；**每个文件的权威条目**
位于对应章节末尾的「函数清单核对表 / Coverage Checklist」。全库统一索引见
[appendices/01-global-function-index.md](../appendices/01-global-function-index.md)。
