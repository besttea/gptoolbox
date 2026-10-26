# 80 Python 桥接模块 / Python Bridge Module

> `python/` 是 gptoolbox 中通过 **MATLAB 的 Python 互操作（`py.*` / `pyrunfile`）** 把几何算法
> 委托给 Python 的模块。目录内共 **4 个文件**：**3 个 MATLAB 侧 `.m`**（`marching_cubes_sparse`、
> `pysparse`、`remesh_botsch`）+ **1 个被调用的 Python 脚本**（`marching_cubes_sparse.py`）。
> 桥接走的是 MATLAB 自带的**独立 Python 解释器进程**（out-of-process），**不是** `system('python ...')`
> 式的 shell 命令行调用。
>
> `python/` delegates geometry work to Python through **MATLAB's Python interoperability
> (`py.*` / `pyrunfile`)**. The directory holds **4 files**: **three MATLAB-side `.m`** (`marching_cubes_sparse`,
> `pysparse`, `remesh_botsch`) plus **one called Python script** (`marching_cubes_sparse.py`). Bridging uses
> MATLAB's own **out-of-process Python interpreter** — **not** a `system('python ...')` shell call.

- 语言 / Language：中英对照（中文为主）
- 覆盖 / Coverage：`python/*.m` **3 / 3**（100%）+ 脚本 `python/*.py` **1 / 1**（见 [80.1 章节](01-python-bindings.md)）
- 粒度 / Granularity：逐函数 API + 桥接模式 + 数据编组 + 算法 + 流程图 + 变量流动

---

## 1. 章节结构 / Chapter layout

| 章节 | 文件 | 条目数 | 主题 |
|---|---|---|---|
| 80.1 | [01-python-bindings.md](01-python-bindings.md) | 3 | Python 桥接三模式（`pyrunfile` / `py.*` 就地 / `py.*` 包调用）/ Three bridge patterns |
| — | **合计 / Total** | **3** | — |

---

## 2. 模块职责 / Module responsibility

- **跨语言委托**：把 MATLAB 不擅长或已有成熟 Python 实现的几何算法（等值面抽取、稀疏矩阵互转、
  Botsch–Kobbelt 重网格）交给 Python 生态（`igl` / `scipy` / `gpytoolbox`）执行。
- **数据编组（marshalling）**：在 MATLAB 1 基 ↔ Python 0 基索引、`double(float64)` 位置 ↔
  `int32` 索引之间显式转换（`−1` 下行 / `+1` 上行），以符合各 Python 绑定的类型要求。
- **环境适配**：`remesh_botsch` 独家提供**可操作错误提示**（`assert_gpytoolbox`），
  其余两者在缺包时直接抛底层异常。

### 目录构成核对 / Directory inventory

| 文件 | 类型 | 角色 |
|---|---|---|
| [python/marching_cubes_sparse.m](file:///c:/VSWork/gptoolbox-1/python/marching_cubes_sparse.m#L1-L14) | MATLAB | `pyrunfile` 驱动脚本的包装 |
| [python/marching_cubes_sparse.py](file:///c:/VSWork/gptoolbox-1/python/marching_cubes_sparse.py#L1-L14) | Python 脚本 | 被 `pyrunfile` 调用的 libigl 等值面脚本 |
| [python/pysparse.m](file:///c:/VSWork/gptoolbox-1/python/pysparse.m#L1-L25) | MATLAB | MATLAB ⇄ SciPy 稀疏矩阵双向转换 |
| [python/remesh_botsch.m](file:///c:/VSWork/gptoolbox-1/python/remesh_botsch.m#L1-L106) | MATLAB | 调 `gpytoolbox.remesh_botsch` 重网格（含内嵌 `assert_gpytoolbox`） |

---

## 3. 全量函数索引 / Complete function index

### 3.1 MATLAB 侧函数 / MATLAB-side functions

| 函数 (anchor) | 桥接方式 | 一句话功能 |
|---|---|---|
| [`marching_cubes_sparse`](file:///c:/VSWork/gptoolbox-1/python/marching_cubes_sparse.m#L1-L14) | `pyrunfile('marching_cubes_sparse.py',...)` | 从稀疏标量体数据抽等值面（委托 `igl.marching_cubes`） |
| [`pysparse`](file:///c:/VSWork/gptoolbox-1/python/pysparse.m#L1-L25) | `py.scipy.sparse.*`（就地） | MATLAB `sparse` ⇄ SciPy CSC/COO 稀疏矩阵**双向**转换 |
| [`remesh_botsch`](file:///c:/VSWork/gptoolbox-1/python/remesh_botsch.m#L1-L106) | `py.gpytoolbox.remesh_botsch(...)` | 按目标边长做 Botsch–Kobbelt 三角网格重网格 |

### 3.2 Python 侧脚本 / Python-side script

| 脚本 | 被谁调用 | 一句话功能 |
|---|---|---|
| [marching_cubes_sparse.py](file:///c:/VSWork/gptoolbox-1/python/marching_cubes_sparse.py#L1-L14) | `marching_cubes_sparse`（`pyrunfile`） | `igl.marching_cubes(S,GV,GI,isovalue)` → 打包为带 `V`/`F` 属性的 `res` 对象返回 |

---

## 4. 三种桥接模式 / Three bridging patterns

| 函数 | 机制 | 方向 | 关键外部依赖 |
|---|---|---|---|
| `marching_cubes_sparse` | `pyrunfile(.py)` | MATLAB → 脚本 → MATLAB | `python`、`numpy`、`igl`；脚本须可解析 |
| `pysparse` | `py.scipy.sparse.*`（就地） | **双向** | `python`、`scipy` |
| `remesh_botsch` | `py.gpytoolbox.*`（包调用） | MATLAB → 包 → MATLAB | `python`、`numpy`、`gpytoolbox`（C++ 绑定） |

---

## 5. 依赖与环境 / Dependencies & environment

- **同库依赖 / Same-library deps**：
  - `marching_cubes_sparse` → **无**（仅依赖脚本文件 `marching_cubes_sparse.py`）。
  - `pysparse` → **无**。
  - `remesh_botsch` → **无**；含内嵌函数 `assert_gpytoolbox`（[L85-L106](file:///c:/VSWork/gptoolbox-1/python/remesh_botsch.m#L85-L106)）。
- **外部/内置 / Built-ins**：`pyrunfile`（R2021b 起）、`pyenv`、`py.*`、`containers.Map`、`assignin`、
  `assert`、`reshape`、`permute`、`find`、`sparse`、`startsWith`。
- **Python 包 / Python packages**：`numpy`、`scipy`、`igl`、`gpytoolbox`（须安装在 `pyenv` 指定的解释器中）。
- **路径注意 / Path note**：仓库 `CLAUDE.md` 的 `addpath(...)` 示例**未包含 `python`**；`marching_cubes_sparse`
  依赖 `pyrunfile` 解析 `marching_cubes_sparse.py`，需确保该脚本所在目录可被解析。
- **构建 / Build**：**纯 MATLAB + Python，无需编译**。

---

## 6. 文档过程中发现的源码问题 / Source issues found while documenting

| 类别 | 实例 |
|---|---|
| 陈旧注释 | [marching_cubes_sparse.m L2](file:///c:/VSWork/gptoolbox-1/python/marching_cubes_sparse.m#L2) 头注写 `% [V,F] = marching_cubes(X,Y,Z,S)`，与实际签名 `marching_cubes_sparse(S,GV,GI,isovalue)` 不符 |
| 缺失依赖 | `remesh_botsch` 头注 See also 与错误提示引用 `pip_install`，但 **`pip_install.m` 在仓库中不存在** |
| 缺失友好提示 | `marching_cubes_sparse` / `pysparse` 在缺包时**无**可操作提示（仅 `remesh_botsch` 有 `assert_gpytoolbox`） |
| 不存在的同名文件 | 仓库**无** `mesh/marching_cubes.m`；MATLAB 侧 marching-cubes 由 MEX `signed_distance_isosurface`（`'ContouringMethod','marching_cubes'`）提供 |

---

## 7. 交叉引用 / Cross references

- 本章章节：[80.1 Python 接口](01-python-bindings.md)
- 安装与环境（`pyenv`、Python 包）：[00-overview/03-build-and-install.md](../00-overview/03-build-and-install.md)
- 重网格与简化（MATLAB 原生实现）：[10-mesh/09](../10-mesh/09-remeshing-and-simplification.md)
- 模块依赖图：[00-overview/05-dependency-map.md](../00-overview/05-dependency-map.md)
- 文档写作规范：[appendices/03-doc-template.md](../appendices/03-doc-template.md)
