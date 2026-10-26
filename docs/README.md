# gptoolbox 工程文档 / Engineering Documentation

> 本目录是对 `gptoolbox-1` 的**完整工程化逆向文档**。目标：在不阅读源码的前提下，
> 仅凭本目录即可理解每个模块的职责、每个函数的签名与算法，并具备**重建项目**的信息完整度。
>
> This directory is a complete, engineering-grade reverse-engineering document set for
> `gptoolbox-1`. Goal: module responsibilities, per-function signatures and algorithms
> are documented with enough fidelity to **rebuild the project**.

- 语言 / Language：中英对照 (Bilingual, 中文为主)
- 覆盖 / Coverage：全部顶层目录（含第三方 `external/`）
- 粒度 / Granularity：模块架构 + **逐函数** API 详解 + 算法流程 + **Mermaid 流程图** + 变量流动分析

---

## 1. 顶层结构 / Top-level layout

| 目录 | .m 文件数 | 文档章节 | 说明 |
|---|---|---|---|
| `mesh/` | 566 | [10-mesh](10-mesh/README.md) | 网格算法（最大模块）：I/O、拓扑、DDG 算子、变形、参数化、测地线、曲线 |
| `matrix/` | 52 | [20-matrix](20-matrix/README.md) | 稀疏矩阵工具、优化求解器（quadprog/active-set/ADMM/LBFGS） |
| `imageprocessing/` | 43 | [30-imageprocessing](30-imageprocessing/README.md) | 图像着色、扩散曲线、抖动、色彩空间 |
| `utility/` | 41 | [40-utility](40-utility/README.md) | 通用工具：缓存、图形导出、色图、路径 |
| `wrappers/` | 19 | [50-wrappers](50-wrappers/README.md) | 外部二进制封装：TetGen/Triangle/QSlim/meshfix/medit |
| `mex/` | 57 (+cpp/CMake) | [60-mex](60-mex/README.md) | 编译型 C++ MEX（libigl/Eigen/CGAL/Embree） |
| `quat/` | 1 | [70-quat](70-quat/README.md) | 四元数运算 |
| `python/` | 3 | [80-python](80-python/README.md) | Python 桥接脚本 |
| `images/` | 1 | [85-images](85-images/README.md) | 示例图像素材与加载器 |
| `external/` | 197 | [90-external](90-external/README.md) | 第三方 vendored MATLAB 代码 |

全局索引与总览：

| 文档 | 内容 |
|---|---|
| [00-overview/01-project-overview.md](00-overview/01-project-overview.md) | 项目总览 / 功能清单 / 数据约定 |
| [00-overview/02-architecture.md](00-overview/02-architecture.md) | 架构分层与模块职责 |
| [00-overview/03-build-and-install.md](00-overview/03-build-and-install.md) | 安装、路径、MEX 编译 |
| [00-overview/04-coding-conventions.md](00-overview/04-coding-conventions.md) | 编码规范（文件名/函数头/varargin） |
| [00-overview/05-dependency-map.md](00-overview/05-dependency-map.md) | 模块与关键函数依赖图 |
| [00-overview/06-glossary.md](00-overview/06-glossary.md) | 术语表（几何处理中英对照） |
| [appendices/01-global-function-index.md](appendices/01-global-function-index.md) | **全库函数总索引**（函数 → 文件 → 模块 → 文档锚点） |
| [appendices/02-module-dependency-graph.md](appendices/02-module-dependency-graph.md) | 全局依赖关系图 |
| [appendices/03-doc-template.md](appendices/03-doc-template.md) | **逐函数文档模板**（写作规范） |

---

## 2. 文档规范 / Documentation conventions

1. **每个函数一条独立条目**，条目必须包含：标识、文件锚点、签名、功能（中英）、
   输入/输出表、变量流动、算法步骤、**Mermaid 流程图**、依赖、See also。
2. 文件锚点统一使用 `file:///c:/VSWork/gptoolbox-1/<相对路径>`（带 `#Lx-Ly` 行号锚点）。
3. 矩阵尺寸使用 `#V by 3` 记法（与源码注释一致）。
4. 流程图使用 `mermaid flowchart`，节点标签简洁；分支条件用 `{...}`。
5. 数据结构缩写：`V` 顶点、`F` 面、`T` 四面体、`E` 边、`C` 角/角标、`I` 索引。
6. 详见 [appendices/03-doc-template.md](appendices/03-doc-template.md)。

### 逐函数条目模板 / Per-function entry template

````markdown
### `function_name` — 中文简述
- **文件 / File**: [mesh/function_name.m](file:///c:/VSWork/gptoolbox-1/mesh/function_name.m#L1-L40)
- **签名 / Signature**: `[out1,out2] = function_name(V,F,'Name',Value,...)`
- **功能 / Purpose**: `EN: ...` / 中文：...
- **输入 / Inputs**:
  | 参数 | 尺寸/类型 | 含义 (中/英) |
  |---|---|---|
  | `V` | `#V by 3` | 顶点坐标 / vertex positions |
- **输出 / Outputs**:
  | 参数 | 尺寸/类型 | 含义 |
- **可选参数 / Optional**:
  | 名称 | 取值 | 默认 | 含义 |
- **变量流动 / Variable flow**: `V,F → (步骤A) → M → (步骤B) → out1`
- **算法步骤 / Algorithm**:
  1. ...
- **流程图 / Flowchart**:
  ```mermaid
  flowchart TD
    A["输入 V,F"] --> B["..."] --> C["输出 out1"]
  ```
- **依赖 / Dependencies**: `cotmatrix`, `massmatrix`
- **See also**: `lscm`
````

---

## 3. 阅读顺序建议 / Suggested reading order

1. [00-overview/01-project-overview.md](00-overview/01-project-overview.md) → 了解定位与约定
2. [00-overview/02-architecture.md](00-overview/02-architecture.md) → 掌握分层
3. [00-overview/03-build-and-install.md](00-overview/03-build-and-install.md) → 跑起来
4. 按模块进入 `10-mesh` … `90-external`
5. 需要查函数时用 [appendices/01-global-function-index.md](appendices/01-global-function-index.md)
