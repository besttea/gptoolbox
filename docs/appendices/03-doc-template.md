# 逐函数文档模板 / Per-function documentation template

> 本文件定义 `docs/` 下所有函数条目的统一格式。所有模块文档必须严格遵循。
> This file defines the unified per-function entry format for every module document.

## 1. 条目结构 / Entry structure

````markdown
### `function_name` — 中文简述
- **文件 / File**: [mesh/function_name.m](file:///c:/VSWork/gptoolbox-1/mesh/function_name.m#L1-L40)
- **签名 / Signature**: `[out1,out2] = function_name(V,F,'Name',Value,...)`
- **功能 / Purpose**: `EN: one-line description.` / 中文：一句话说明。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `V` | `#V by 3` | 顶点坐标 / vertex positions |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `out1` | `#F by 3` | ... |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `'Robust'` | `logical` | `false` | ... |

- **变量流动 / Variable flow**: `V,F →(1 计算面积)→ A →(2 归一化)→ out1`
- **算法步骤 / Algorithm**:
  1. 步骤一 / Step 1
  2. 步骤二 / Step 2
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 V,F"] --> B{"分支条件?"}
    B -- 是 --> C["路径1"]
    B -- 否 --> D["路径2"]
    C --> E["输出 out1"]
    D --> E
  ```

- **依赖 / Dependencies**: `cotmatrix`, `massmatrix`
- **See also**: `lscm`, `arap`
````

## 2. 硬性规则 / Hard rules

| # | 规则 | 说明 |
|---|---|---|
| R1 | 每个 `.m` 函数必须有独立 `###` 条目 | 一个都不能漏；本目录是逆向重建依据 |
| R2 | 文件锚点必须可点击 | `file:///c:/VSWork/gptoolbox-1/<path>#Lx-Ly`，行号取函数体实际范围 |
| R3 | 签名必须与源码一致 | 含 `varargin` 时给出可选参数表 |
| R4 | 必须有 Mermaid 流程图 | 简单函数允许 2–3 节点；有分支/循环必须体现 |
| R5 | 必须有变量流动链 | 用 `→` 串联输入、中间量、输出 |
| R6 | 中英对照 | 标题、功能、表格列头中英并用；算法步骤可用中文为主 |
| R7 | 依赖列表 | 列出被调用的同库函数（用于全局依赖图） |
| R8 | 无源文件时（MEX stub） | 标注 `[MEX]`，文件名沿用 `mex/<name>.m`（注释桩），实现指向 `<name>.cpp` |

## 3. 命名与符号约定 / Naming & notation

| 符号 | 含义 / Meaning |
|---|---|
| `V` | `#V by dim` 顶点位置 / vertex positions |
| `F` | `#F by 3` 三角面索引 / triangle indices |
| `Q` | `#Q by 4` 四边形索引 / quad indices |
| `T` | `#T by 4` 四面体索引 / tet indices |
| `E` | `#E by 2` 边索引 / edge indices |
| `C` | 角标 / corner (triangle-vertex) indices |
| `W` | 权重矩阵 / weights |
| `L` | 拉普拉斯矩阵 / Laplacian |
| `M` | 质量矩阵 / mass matrix |
| `#V` | 顶点数 / number of vertices |

## 4. 流程图约定 / Flowchart conventions

- 使用 `flowchart TD`（自上而下）。
- 输入节点用 `A["输入 ..."]`，输出节点用 `Z["输出 ..."]`。
- 判定节点用 `{"..."} `，边标注条件（`-- 是 -->`）。
- 循环用 `-->|<循环体>|` 回边。
- 复杂函数可拆 `subgraph`；节点标签内避免 `"`、换行。
- 表达式含括号/下标时用双引号包裹标签，例如 `A["A = W\\L"]`。
