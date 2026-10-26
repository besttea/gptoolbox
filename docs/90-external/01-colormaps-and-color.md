# 90.1 配色与颜色转换（第三方） / Colormaps and Color Conversion (Vendored)

> `external/` 是 gptoolbox **直接收录的第三方 MATLAB 代码**目录（另见 [CLAUDE.md](file:///c:/VSWork/gptoolbox-1/CLAUDE.md)
> 的目录说明："vendored third-party MATLAB code"）。这里的文件**一般不由 gptoolbox 作者编写或维护**，
> 而是原样搬运社区/其他作者的 `.m` 实现；gptoolbox 只在需要时调用它们（例如 `utility/zoe.m`、
> `utility/lipmanya.m` 调用本目录的 `hex2rgb`）。因此本章的职责是**忠实记录作者、来源、许可证与行为**，
> 并明确指出**哪些函数遮蔽了 MATLAB 内置函数（builtin shadowing）**、**哪些与 gptoolbox 自家函数功能重叠（duplication）**。
>
> This chapter documents the **vendored third-party MATLAB code** under `external/` that deals with
> colormaps and color-space conversion. Vendoring here means the files are copied from other authors
> essentially verbatim; gptoolbox may call them (e.g. `utility/zoe.m`, `utility/lipmanya.m` call
> `hex2rgb`). The chapter records the original author/source/license faithfully and flags
> **builtin shadowing** and **duplication of gptoolbox-owned functions**.
>
> 本目录其余 `external/` 文件（fast marching、exact geodesics、Hungarian、`tilefigs` 等）由
> `90-external/` 下的姊妹文档覆盖，本文只处理下列 **11 个** 配色/颜色转换文件。

- 语言 / Language：中英对照（中文为主）
- 相关章节 / See also：[40.2 绘图辅助与配色](../40-utility/02-plotting-and-colors.md)（gptoolbox **自家**色图与颜色常量）、
  [appendices/03-doc-template.md](../appendices/03-doc-template.md)（本文写作规范）
- 命名约定 / Notation：`n` 表示色图采样数；`C`/`CM`/`colormap`/`cm_data`/`rgb` 表示 `n by 3` 的 RGB 或色图矩阵；
  `hex` 表示十六进制颜色字符串。

---

## 本章概览 / Chapter Overview

| 函数 / Function | 来源 / 署名 (Attribution) | 是否遮蔽内置 (Shadows builtin?) | 一句话功能 (One-liner) |
|---|---|---|---|
| `cbrewer` | Author: Charles Robert（BSD，2011） | 否 | 生成 ColorBrewer 色表（可选插值扩展） |
| `interpolate_cbrewer` | Author: Charles Robert（BSD，2011） | 否 | 把 ColorBrewer 色表插值到 `ncolors` 级 |
| `plot_brewer_cmap` | Author: Charles Robert（BSD，2011） | 否 | 在一个 figure 中画出全部 ColorBrewer 色表 |
| `viridis` | (c) 2016 Ander Biguri（BSD）；数据源自 matplotlib | 否（非 MATLAB 内置） | matplotlib viridis 色图（256 锚点/重采样） |
| `inferno` | (c) 2016 Ander Biguri（BSD）；数据源自 matplotlib | 否（非 MATLAB 内置） | matplotlib inferno 色图（256 锚点/重采样） |
| `magma` | (c) 2016 Ander Biguri（BSD）；数据源自 matplotlib | 否（非 MATLAB 内置） | matplotlib magma 色图（256 锚点/重采样） |
| `plasma` | (c) 2016 Ander Biguri（BSD）；数据源自 matplotlib | 否（非 MATLAB 内置） | matplotlib plasma 色图（256 锚点/重采样） |
| `cmyk2rgb` | 无可考署名 / no attribution in header | 否 | CMYK → sRGB（依赖 Image Processing Toolbox） |
| `rgb2cmyk` | 无可考署名 / no attribution in header | 否 | sRGB → CMYK（依赖 Image Processing Toolbox） |
| `hex2rgb` | Chad A. Greene（2014，头注无许可证） | 否 | 十六进制颜色 → RGB（0–1 或 0–255） |
| `rgb2hex` | Chad A. Greene（2014，头注无许可证） | 否 | RGB → 十六进制颜色字符串 |

> ⚠️ 关于"遮蔽内置"的**重要更正**：本章模板最初假定 `viridis`/`inferno`/`magma`/`plasma` 自 **R2020b** 起
> 是 MATLAB 内置色图。但据 MathWorks 官方文档，R2020b 新增的内置色图**只有 `turbo`**
> （[mathworks.com/help/matlab/ref/turbo.html](https://ww2.mathworks.cn/help/matlab/ref/turbo.html)：版本历史"在 R2020b 中推出"）；
> MATLAB **没有**内置 `viridis`/`inferno`/`magma`/`plasma`。因此本目录这 4 个函数**不构成对 MATLAB 内置的遮蔽**，
> 仅存在与同名 **File Exchange** 实现（Ander Biguri 版本被广泛分发）发生**命名冲突**的风险。
> 真正遮蔽内置的是 gptoolbox 自家的 [`utility/turbo.m`](file:///c:/VSWork/gptoolbox-1/utility/turbo.m#L1-L22)
> （与 R2020b 内置 `turbo` 同名）。详见各条的"备注 / Note"。

---

## 逐函数详解 / Per-function reference

### `cbrewer` — 生成 ColorBrewer 色表
- **文件 / File**: [external/cbrewer.m](file:///c:/VSWork/gptoolbox-1/external/cbrewer.m#L1-L140)
- **签名 / Signature**: `[colormap] = cbrewer(cname, ncol, interp_method)`
- **署名 / Attribution**: 头注 `Author: Charles Robert`，`email: tannoudji@hotmail.com`，`Date: 06.12.2011`；
  许可证为 **BSD 三条款**（"Copyright (c) 2011, Charles Robert / All rights reserved." + 标准 BSD 再分发条款，
  见 [external/cbrewer.m#L116-L139](file:///c:/VSWork/gptoolbox-1/external/cbrewer.m#L116-L139)）。
  色表数据来源：Cynthia Brewer 的 ColorBrewer（http://colorbrewer.org/），
  由 psu.edu 上的 MS-Excel 文件导出。
- **功能 / Purpose**: `EN: Produce a ColorBrewer colormap table (n-by-3 RGB) for a given table name and number of colors, interpolating when more colors are requested than originally available.` /
  中文：按表名与颜色数生成 ColorBrewer 色表（`n by 3` RGB），当请求颜色数超过原表最大色数时用插值"扩展"。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `cname` | `char` | 色表名（见下三类清单）/ colormap table name |
  | `ncol` | 整数 | 需要的颜色数 / number of colors |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `colormap` | `ncol by 3` | RGB（值域 **0–1**，源码末尾除以 255）/ colormap matrix in [0,1] |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `interp_method` | `'pchip'`/`'linear'`/`'spline'`/`'cubic'`/`'nearest'` | `'pchip'`（"aka 'cubic'"，L50/L79） | 需要扩展时透传给 `interp1` 的插值方法 / interp1 method when extending |

- **色表类型字母与数量 / Type letters and counts**（源码 L83 `ctype_names={'div','seq','qual'}`，头注 L10-L47）:

  | 类型字母 | 语义 | 表名数量 | 表名 |
  |---|---|---|---|
  | `seq` | Sequential 顺序型 | 18 | `Blues`,`BuGn`,`BuPu`,`GnBu`,`Greens`,`Greys`,`Oranges`,`OrRd`,`PuBu`,`PuBuGn`,`PuRd`,`Purples`,`RdPu`,`Reds`,`YlGn`,`YlGnBu`,`YlOrBr`,`YlOrRd` |
  | `div` | Divergent 发散型 | 8 | `BrBG`,`PiYG`,`PRGn`,`PuOr`,`RdBu`,`RdGy`,`RdYlBu`,`RdYlGn` |
  | `qual` | Qualitative 定性型 | 8 | `Accent`,`Dark2`,`Paired`,`Pastel1`,`Pastel2`,`Set1`,`Set2`,`Set3` |

  > 合计 34 张表。每张表的**预置色数**（如 3–9 等）存在同目录的 `colorbrewer.mat` 中，
  > 源码用 `length(colorbrewer.(ctype).(cname))` 取得"该表的档位数"，`colorbrewer.(ctype).(cname){ncol}`
  > 取第 `ncol` 档的配色；并非每档都存在（可能出现 `[]`）。

- **变量流动 / Variable flow**: `cname,ncol,interp_method →(1 load colorbrewer.mat)→(2 在 {div,seq,qual} 中查 ctype)→ ctype →(3a ncol>档位上限: 取最大档 cbrew_init → interpolate_cbrewer → /255 : 3b 否则取 colormap=table{ncol}/255)→ colormap`
- **算法步骤 / Algorithm**:
  1. `load('colorbrewer.mat')`，并初始化 `colormap=[]`；`interp_method` 缺省 `'pchip'`（L75–L80）。
  2. 依次在 `{'div','seq','qual'}` 中查 `isfield(colorbrewer.(try_ctype),cname)`，命中即 `ctype`；
     全未命中则 `error(['Could not find colormap named: ' cname])`（L83–L94）。
  3. **扩展分支**：若 `ncol > length(colorbrewer.(ctype).(cname))`（请求数超过该表档位数上限），
     取该表**最后一档（最大色数）** `cbrew_init`，调用 `interpolate_cbrewer(cbrew_init, interp_method, ncol)`，
     再 `colormap./255` 得到 0–1，随后 `return`（L96–L101）。
  4. **取档分支**：若 `colorbrewer.(ctype).(cname){ncol}` 为**空**，用 `while isempty(...)` 递增 `ncol`
     直到非空，并 `warning` 提示该表的最小色数（L103–L111）。
  5. `colormap = (colorbrewer.(ctype).(cname){ncol})./255` 并返回（L113）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 cname, ncol, interp_method"] --> B["load colorbrewer.mat"]
    B --> C{"interp_method 缺省 ?"}
    C -- 是 --> C1["interp_method = pchip"]
    C -- 否 --> D
    C1 --> D["在 div seq qual 中按名查找 ctype"]
    D --> E{"找到 ctype ?"}
    E -- 否 --> E1["error: Could not find colormap"]
    E -- 是 --> F{"ncol 大于该表档位上限 ?"}
    F -- 是 --> F1["取最大档 interpolate_cbrewer 扩展 再 /255"]
    F -- 否 --> G{"table{ncol} 为空 ?"}
    G -- 是 --> G1["ncol 递增直到非空 并 warning 最小色数"]
    G -- 否 --> H
    G1 --> H["colormap = table{ncol} / 255"]
    H --> Z["输出 colormap"]
    F1 --> Z
  ```

- **依赖 / Dependencies**: `interpolate_cbrewer` ([external/interpolate_cbrewer.m](file:///c:/VSWork/gptoolbox-1/external/interpolate_cbrewer.m#L1-L61))、
  `load('colorbrewer.mat')`（需同目录数据文件 [external/colorbrewer.mat](file:///c:/VSWork/gptoolbox-1/external/colorbrewer.mat) 在路径上）、
  `interp1`（经 interpolate_cbrewer 间接调用，MATLAB 内置）
- **See also**: `plot_brewer_cmap`, `interpolate_cbrewer`, `colormap`
- **平台/工具箱要求 / Platform**: 纯 MATLAB，无工具箱依赖；**必须保证 `colorbrewer.mat` 与本文件在同一目录且该目录在 MATLAB 路径上**。
- **备注 / Note**: 非 MATLAB 内置函数，gptoolbox 亦无同名函数，**无遮蔽、无重复**。
  头注声明可选参数 `interp_method`，但**调用者若不给 `ncol` 会直接出错**；此外若把 `cbrewer` 无参调用，
  `cname` 未定义会报错（详见 `plot_brewer_cmap` 的备注）。

### `interpolate_cbrewer` — 把 ColorBrewer 色表插值到 ncolors 级
- **文件 / File**: [external/interpolate_cbrewer.m](file:///c:/VSWork/gptoolbox-1/external/interpolate_cbrewer.m#L1-L61)
- **签名 / Signature**: `[interp_cmap] = interpolate_cbrewer(cbrew_init, interp_method, ncolors)`
- **署名 / Attribution**: 头注 `Author: Charles Robert`，`email: tannoudji@hotmail.com`，`Date: 14.10.2011`；
  许可证同样为 **BSD 三条款**（`Copyright (c) 2011, Charles Robert`，
  见 [external/interpolate_cbrewer.m#L37-L60](file:///c:/VSWork/gptoolbox-1/external/interpolate_cbrewer.m#L37-L60)）。
- **功能 / Purpose**: `EN: Interpolate an N-by-3 ColorBrewer colormap to a requested number of colors using interp1.` /
  中文：把一张 `N by 3` 的 ColorBrewer 色表用 `interp1` 插值到 `ncolors` 级。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `cbrew_init` | `N by 3` | 初始色表（**0–255 整数域**）/ initial colormap |
  | `interp_method` | `char` | 插值方法，取值 `'nearest'`/`'linear'`/`'spline'`/`'cubic'` / interp1 method | 
  | `ncolors` | 整数（可为小数） | 目标颜色数 / desired number of colors |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `interp_cmap` | `ncolors by 3` | 插值后的色表，**四舍五入为整数（0–255 域）** |

- **可选参数 / Optional**: 无 `varargin`；`interp_method` 为需要显式给出的位置参数
  （`'cubic'` 仅在数据等距时才是双三次，否则退化为 `'spline'`，见头注 L11-L12）。
- **变量流动 / Variable flow**: `cbrew_init,interp_method,ncolors →(1 ncolors=round(ncolors))→(2 nmax=size(cbrew_init,1))→(3 a=(ncolors-1)/(nmax-1); X=round([0 a:a:(ncolors-1)]); X2=0:ncolors-1)→(4 逐通道 interp1(X, cbrew_init(:,k), X2, interp_method) k=1,2,3)→ z,z2,z3 →(5 round([z' z2' z3']))→ interp_cmap`
- **算法步骤 / Algorithm**:
  1. `ncolors = round(ncolors)`（容忍小数输入，L21）。
  2. `nmax = size(cbrew_init,1)` 得到原表点数（L24）。
  3. 构造采样轴：`a = (ncolors-1)./(nmax-1)`；`X = round([0 a:a:(ncolors-1)])`；
     `X2 = 0:ncolors-1`（L27–L29）。`X` 是**原表索引轴**，`X2` 是**目标索引轴**。
  4. 三个通道分别 `interp1(X, cbrew_init(:,k), X2, interp_method)` → `z,z2,z3`（L31–L33）。
  5. `interp_cmap = round([z' z2' z3'])` 组合并取整（L34）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 cbrew_init, interp_method, ncolors"] --> B["ncolors = round(ncolors)"]
    B --> C["nmax = size(cbrew_init,1)"]
    C --> D["a = (ncolors-1)/(nmax-1); X = round(0:a:ncolors-1); X2 = 0:ncolors-1"]
    D --> E["逐通道 interp1(X, cbrew_init(:,k), X2, interp_method)"]
    E --> F["round 组合三通道"]
    F --> Z["输出 interp_cmap (ncolors by 3)"]
  ```

- **依赖 / Dependencies**: `interp1`、`round`、`size`（MATLAB 内置）；无本库其它依赖。
- **See also**: `cbrewer`（唯一调用者）, `interp1`
- **平台/工具箱要求 / Platform**: 纯 MATLAB，无工具箱依赖。
- **备注 / Note**: 输出为**取整后的 0–255 整数**（未 `/255`），因此 `cbrewer` 扩展分支在调用它之后才会
  做 `colormap./255`。非内置、无同名冲突。

### `plot_brewer_cmap` — 画出全部 ColorBrewer 色表
- **文件 / File**: [external/plot_brewer_cmap.m](file:///c:/VSWork/gptoolbox-1/external/plot_brewer_cmap.m#L1-L84)
- **签名 / Signature**: `f = plot_brewer_cmap()`
- **署名 / Attribution**: 头注 `Author: Charles Robert`，`email: tannoudji@hotmail.com`，`Date: 14.10.2011`；
  许可证 **BSD 三条款**（`Copyright (c) 2011, Charles Robert`，
  见 [external/plot_brewer_cmap.m#L60-L83](file:///c:/VSWork/gptoolbox-1/external/plot_brewer_cmap.m#L60-L83)）。
- **功能 / Purpose**: `EN: Plot and label every ColorBrewer table in one figure (1-by-3 subplots for div/seq/qual).` /
  中文：在一个 figure 里用 1×3 子图把 div/seq/qual 三类的全部 ColorBrewer 色表逐行画出色块并标注名称。
- **输入 / Inputs**: 无 / None
- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `f` | `figure` 句柄 | 新建 figure 的句柄 / handle to new figure |

- **可选参数 / Optional**: 无 / None
- **变量流动 / Variable flow**: `∅ →(1 load colorbrewer.mat)→(2 ctypes={div,seq,qual}; cnames{1..3} 表名清单)→(3 figure('position',[314 327 807 420]))→(4 for itype=1:3, subplot(1,3,itype))→(5 for iname, ncol=档位数; F=cbrewer(name,ncol))→(6 for icol, fill(X2,Y,F(icol,:)) + text 名称)→(7 title/axis off/set(gcf))→ f`
- **算法步骤 / Algorithm**:
  1. `load('colorbrewer.mat')`（L17）。
  2. 定义三类 `ctypes={'div','seq','qual'}`，以及每类的表名清单 `cnames{1}`(div,8)/`cnames{2}`(seq,18)/`cnames{3}`(qual,8)（L19–L24）。
  3. 建 figure `f = figure('position',[314 327 807 420])`（L26）。
  4. 对 `itype=1:3` 各开一个 `subplot(1,3,itype)`（L27–L31）。
  5. 对内层 `iname`：`ncol=length(colorbrewer.(ctypes{itype}).(cnames{itype}{iname}))`；
     `fg=1./ncol`；调用 `F=cbrewer(cnames{itype}{iname}, ncol)` 取该表配色（L33–L40）。
  6. 对 `icol=1:ncol` 用 `fill(X2,Y,F(icol,:),'linestyle','none')` 画色块，并在左侧 `text` 标注表名（L42–L48）。
  7. 每个子图 `title(...)`、`axis off`、`set(gcf,'color',[1 1 1])`（L50–L52）；
     最后 `set(gcf,'MenuBar','none')`、`set(gcf,'Name','ColorBrewer Color maps')`（L57–L58）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["调用 plot_brewer_cmap()"] --> B["load colorbrewer.mat"]
    B --> C["定义 ctypes(div,seq,qual) 与 cnames 表名清单"]
    C --> D["f = figure(position 314 327 807 420)"]
    D --> E["for itype = 1:3 开 subplot(1,3,itype)"]
    E --> F["for iname 逐表: ncol 取档位; F = cbrewer(name,ncol)"]
    F --> G["for icol = 1:ncol: fill 色块 + text 名称"]
    G --> H{"外层遍历完 ?"}
    H -- 否 --> E
    H -- 是 --> I["title / axis off / set(gcf)"]
    I --> Z["输出 f (figure 句柄)"]
  ```

- **依赖 / Dependencies**: `cbrewer` ([external/cbrewer.m](file:///c:/VSWork/gptoolbox-1/external/cbrewer.m#L1-L140))、
  `load('colorbrewer.mat')`（[external/colorbrewer.mat](file:///c:/VSWork/gptoolbox-1/external/colorbrewer.mat)）、
  `figure`/`subplot`/`fill`/`text`/`title`/`axis`（MATLAB 内置绘图）
- **See also**: `cbrewer`, `interpolate_cbrewer`
- **平台/工具箱要求 / Platform**: 纯 MATLAB 绘图，无工具箱依赖；需 `colorbrewer.mat` 在路径上。
- **备注 / Note**: **头注与实现不符**：头注（L3）称"Is called by `cbrewer.m` when no arguments are given"，
  但当前 `cbrewer` 的签名是 `cbrewer(cname,ncol,interp_method)`，**无参调用会在 L87 因 `cname` 未定义而报错**，
  并不会转到本函数。属文档过期。非内置、无同名冲突。

### `viridis` — matplotlib viridis 色图
- **文件 / File**: [external/viridis.m](file:///c:/VSWork/gptoolbox-1/external/viridis.m#L1-L296)
- **签名 / Signature**: `cm_data = viridis(m)`
- **署名 / Attribution**: 头注 `Copyright (c) 2016, Ander Biguri / All rights reserved.`，许可证 **BSD 三条款**
  （见 [external/viridis.m#L7-L30](file:///c:/VSWork/gptoolbox-1/external/viridis.m#L7-L30)）。
  **但头注未提及 python/matplotlib**：`viridis` 色图本身由 **Nathaniel Smith 与 Stefan van der Walt** 为
  matplotlib 设计（matplotlib/CC0），本文件是其在 MATLAB 中的移植（硬编码锚点 + HSV 插值）。
- **功能 / Purpose**: `EN: Return the matplotlib viridis perceptually-uniform colormap (256-entry table by default, or resampled to m colors via HSV interpolation).` /
  中文：返回 matplotlib 的 `viridis` 感知均匀色图（默认 256 行锚点表；给定 `m` 时在 HSV 空间重采样到 `m` 色）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `m` | 正整数（**可选**） | 输出颜色数 / number of colors |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `cm_data` | `256 by 3`（无 `m`）或 `m by 3` | 色图，值域 0–1 |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `m` | 正整数 | 无（`nargin<1` 时返回完整 256 行表） | 重采样后的颜色数（HSV 空间 `interp1`） |

- **锚点色表 / Anchor colour table**: **256** 行硬编码锚点（[L31–L286](file:///c:/VSWork/gptoolbox-1/external/viridis.m#L31-L286)）；
  首色 `[0.26700401, 0.00487433, 0.32941519]`（深紫 `#440154`），
  末色 `[0.99324789, 0.90615657, 0.1439362]`（亮黄 `#FDE725`）。
- **变量流动 / Variable flow**: `m →(nargin<1 → cm_data = cm 256 by 3 : 否则 1 rgb2hsv(cm) → hsv → 2 interp1(linspace(0,1,256),hsv,linspace(0,1,m)) → cm_data → 3 hsv2rgb(cm_data))→ cm_data`
- **算法步骤 / Algorithm**:
  1. 内置 256 行 `cm` 锚点（L31–L286）。
  2. 若 `nargin < 1`：`cm_data = cm`（返回完整 256 行）（L288–L289）。
  3. 否则：`hsv = rgb2hsv(cm)`；`cm_data = interp1(linspace(0,1,size(cm,1)), hsv, linspace(0,1,m))`；
     `cm_data = hsv2rgb(cm_data)`（**在 HSV 空间插值**以避免 RGB 直插的灰调，L291–L293）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 m (可选)"] --> B{"nargin 小于 1 ?"}
    B -- 是 --> Z["输出 cm_data = cm (256 by 3 原始表)"]
    B -- 否 --> C["hsv = rgb2hsv(cm)"]
    C --> D["interp1 在 HSV 空间重采样到 m 点"]
    D --> E["cm_data = hsv2rgb(...)"]
    E --> Z2["输出 cm_data (m by 3)"]
  ```

- **依赖 / Dependencies**: `rgb2hsv`、`hsv2rgb`、`interp1`、`linspace`、`size`（MATLAB 内置）；无本库其它依赖。
- **See also**: `inferno`, `magma`, `plasma`, `turbo`, `parula`, `colormap`
- **平台/工具箱要求 / Platform**: 纯 MATLAB，无工具箱依赖。
- **遮蔽/重复分析 / Shadowing & duplication**: `viridis` **不是** MATLAB 内置函数（R2020b 新增的内置色图只有
  `turbo`），故**不构成内置遮蔽**；仅可能与 File Exchange 上的同名 `viridis` 冲突。与 gptoolbox 自家
  [`utility/turbo.m`](file:///c:/VSWork/gptoolbox-1/utility/turbo.m#L1-L22) 功能相近（皆为彩虹/强度色图），
  但**二者并存、不重复命名**。

### `inferno` — matplotlib inferno 色图
- **文件 / File**: [external/inferno.m](file:///c:/VSWork/gptoolbox-1/external/inferno.m#L1-L296)
- **签名 / Signature**: `[cm_data] = inferno(m)`
- **署名 / Attribution**: `Copyright (c) 2016, Ander Biguri / All rights reserved.`，**BSD 三条款**
  （见 [external/inferno.m#L6-L29](file:///c:/VSWork/gptoolbox-1/external/inferno.m#L6-L29)）；
  `inferno` 色图数据源自 matplotlib（Nathaniel Smith & Stefan van der Walt），头注未注明。
- **功能 / Purpose**: `EN: Return the matplotlib inferno colormap (256 anchors by default, or resampled to m colors in HSV space with a hardcoded hue wrap).` /
  中文：返回 matplotlib 的 `inferno` 色图（默认 256 锚点；给定 `m` 时在 HSV 空间重采样，并含一处硬编码色相回绕修正）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `m` | 正整数（**可选**） | 输出颜色数 / number of colors |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `cm_data` | `256 by 3`（无 `m`）或 `m by 3` | 色图，值域 0–1 |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `m` | 正整数 | 无（`nargin<1` 时返回完整 256 行表） | 重采样后的颜色数（HSV 空间 `interp1`） |

- **锚点色表 / Anchor colour table**: **256** 行硬编码锚点（[L30–L285](file:///c:/VSWork/gptoolbox-1/external/inferno.m#L30-L285)）；
  首色 `[0.001462, 0.000466, 0.013866]`（近黑 `#000004`），
  末色 `[0.988362, 0.998364, 0.644924]`（淡黄 `#FCFFA4`）。
- **变量流动 / Variable flow**: `m →(nargin<1 → cm_data = cm 256 by 3 : 否则 1 rgb2hsv(cm) → hsv → 2 硬编码 hsv(144:end,1)=hsv(144:end,1)+1 → 3 interp1 到 m 点 → 4 将色相>1 者 -1 → 5 hsv2rgb)→ cm_data`
- **算法步骤 / Algorithm**:
  1. 内置 256 行 `cm` 锚点（L30–L285）。
  2. 若 `nargin < 1`：`cm_data = cm`（L286–L287）。
  3. 否则：`hsv = rgb2hsv(cm)`；**硬编码色相回绕** `hsv(144:end,1) = hsv(144:end,1)+1`（L290）；
     `interp1(...)` 到 `m` 点；把插值后 `色相 > 1` 的分量减 1（解除回绕）；最后 `hsv2rgb`（L291–L293）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 m (可选)"] --> B{"nargin 小于 1 ?"}
    B -- 是 --> Z["输出 cm_data = cm (256 by 3 原始表)"]
    B -- 否 --> C["hsv = rgb2hsv(cm)"]
    C --> D["硬编码色相回绕 hsv(144:end,1) = hsv(144:end,1)+1"]
    D --> E["interp1 在 HSV 空间重采样到 m 点"]
    E --> F["插值后色相大于 1 者减 1"]
    F --> G["cm_data = hsv2rgb(...)"]
    G --> Z2["输出 cm_data (m by 3)"]
  ```

- **依赖 / Dependencies**: `rgb2hsv`、`hsv2rgb`、`interp1`、`linspace`（MATLAB 内置）；无本库其它依赖。
- **See also**: `viridis`, `magma`, `plasma`, `turbo`
- **平台/工具箱要求 / Platform**: 纯 MATLAB，无工具箱依赖。
- **遮蔽/重复分析 / Shadowing & duplication**: 同 `viridis`，`inferno` **非 MATLAB 内置**，不构成内置遮蔽；
  仅与同名 File Exchange 实现有命名冲突风险。

### `magma` — matplotlib magma 色图
- **文件 / File**: [external/magma.m](file:///c:/VSWork/gptoolbox-1/external/magma.m#L1-L299)
- **签名 / Signature**: `[cm_data] = magma(m)`
- **署名 / Attribution**: `Copyright (c) 2016, Ander Biguri / All rights reserved.`，**BSD 三条款**
  （见 [external/magma.m#L6-L29](file:///c:/VSWork/gptoolbox-1/external/magma.m#L6-L29)）；
  `magma` 色图数据源自 matplotlib（Nathaniel Smith & Stefan van der Walt），头注未注明。
- **功能 / Purpose**: `EN: Return the matplotlib magma colormap (256 anchors by default, or resampled to m colors in HSV space with a hardcoded hue wrap).` /
  中文：返回 matplotlib 的 `magma` 色图（默认 256 锚点；给定 `m` 时在 HSV 空间重采样，并含硬编码色相回绕）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `m` | 正整数（**可选**） | 输出颜色数 / number of colors |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `cm_data` | `256 by 3`（无 `m`）或 `m by 3` | 色图，值域 0–1 |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `m` | 正整数 | 无（`nargin<1` 时返回完整 256 行表） | 重采样后的颜色数（HSV 空间 `interp1`） |

- **锚点色表 / Anchor colour table**: **256** 行硬编码锚点（[L31–L286](file:///c:/VSWork/gptoolbox-1/external/magma.m#L31-L286)）；
  首色 `[0.001462, 0.000466, 0.013866]`（近黑 `#000004`），
  末色 `[0.987053, 0.991438, 0.749504]`（米白 `#FCFDBF`）。
- **变量流动 / Variable flow**: `m →(nargin<1 → cm_data = cm 256 by 3 : 否则 1 rgb2hsv(cm) → hsv → 2 硬编码 hsv(170:end,1)=hsv(170:end,1)+1 → 3 interp1 到 m 点 → 4 色相>1 者 -1 → 5 hsv2rgb)→ cm_data`
- **算法步骤 / Algorithm**:
  1. 内置 256 行 `cm` 锚点（L31–L286）。
  2. 若 `nargin < 1`：`cm_data = cm`（L289–L290）。
  3. 否则：`hsv = rgb2hsv(cm)`；**硬编码色相回绕** `hsv(170:end,1) = hsv(170:end,1)+1`（L293）；
     `interp1(...)` 到 `m` 点；`色相 > 1` 者减 1；`hsv2rgb`（L294–L296）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 m (可选)"] --> B{"nargin 小于 1 ?"}
    B -- 是 --> Z["输出 cm_data = cm (256 by 3 原始表)"]
    B -- 否 --> C["hsv = rgb2hsv(cm)"]
    C --> D["硬编码色相回绕 hsv(170:end,1) = hsv(170:end,1)+1"]
    D --> E["interp1 在 HSV 空间重采样到 m 点"]
    E --> F["插值后色相大于 1 者减 1"]
    F --> G["cm_data = hsv2rgb(...)"]
    G --> Z2["输出 cm_data (m by 3)"]
  ```

- **依赖 / Dependencies**: `rgb2hsv`、`hsv2rgb`、`interp1`、`linspace`（MATLAB 内置）；无本库其它依赖。
- **See also**: `inferno`, `plasma`, `viridis`, `turbo`
- **平台/工具箱要求 / Platform**: 纯 MATLAB，无工具箱依赖。
- **遮蔽/重复分析 / Shadowing & duplication**: 同 `viridis`，`magma` **非 MATLAB 内置**，不构成内置遮蔽；
  仅与同名 File Exchange 实现有命名冲突风险。

### `plasma` — matplotlib plasma 色图
- **文件 / File**: [external/plasma.m](file:///c:/VSWork/gptoolbox-1/external/plasma.m#L1-L298)
- **签名 / Signature**: `cm_data = plasma(m)`
- **署名 / Attribution**: `Copyright (c) 2016, Ander Biguri / All rights reserved.`，**BSD 三条款**
  （见 [external/plasma.m#L6-L29](file:///c:/VSWork/gptoolbox-1/external/plasma.m#L6-L29)）；
  `plasma` 色图数据源自 matplotlib（Nathaniel Smith & Stefan van der Walt），头注未注明。
- **功能 / Purpose**: `EN: Return the matplotlib plasma colormap (256 anchors by default, or resampled to m colors in HSV space with a hardcoded hue wrap).` /
  中文：返回 matplotlib 的 `plasma` 色图（默认 256 锚点；给定 `m` 时在 HSV 空间重采样，并含硬编码色相回绕）。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `m` | 正整数（**可选**） | 输出颜色数 / number of colors |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `cm_data` | `256 by 3`（无 `m`）或 `m by 3` | 色图，值域 0–1 |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `m` | 正整数 | 无（`nargin<1` 时返回完整 256 行表） | 重采样后的颜色数（HSV 空间 `interp1`） |

- **锚点色表 / Anchor colour table**: **256** 行硬编码锚点（[L31–L286](file:///c:/VSWork/gptoolbox-1/external/plasma.m#L31-L286)）；
  首色 `[0.050383, 0.029803, 0.527975]`（蓝紫 `#0D0887`），
  末色 `[0.940015, 0.975158, 0.131326]`（亮黄 `#F0F921`）。
- **变量流动 / Variable flow**: `m →(nargin<1 → cm_data = cm 256 by 3 : 否则 1 rgb2hsv(cm) → hsv → 2 硬编码 hsv(153:end,1)=hsv(153:end,1)+1 → 3 interp1 到 m 点 → 4 色相>1 者 -1 → 5 hsv2rgb)→ cm_data`
- **算法步骤 / Algorithm**:
  1. 内置 256 行 `cm` 锚点（L31–L286）。
  2. 若 `nargin < 1`：`cm_data = cm`（L288–L289）。
  3. 否则：`hsv = rgb2hsv(cm)`；**硬编码色相回绕** `hsv(153:end,1) = hsv(153:end,1)+1`（L292）；
     `interp1(...)` 到 `m` 点；`色相 > 1` 者减 1；`hsv2rgb`（L293–L295）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 m (可选)"] --> B{"nargin 小于 1 ?"}
    B -- 是 --> Z["输出 cm_data = cm (256 by 3 原始表)"]
    B -- 否 --> C["hsv = rgb2hsv(cm)"]
    C --> D["硬编码色相回绕 hsv(153:end,1) = hsv(153:end,1)+1"]
    D --> E["interp1 在 HSV 空间重采样到 m 点"]
    E --> F["插值后色相大于 1 者减 1"]
    F --> G["cm_data = hsv2rgb(...)"]
    G --> Z2["输出 cm_data (m by 3)"]
  ```

- **依赖 / Dependencies**: `rgb2hsv`、`hsv2rgb`、`interp1`、`linspace`（MATLAB 内置）；无本库其它依赖。
- **See also**: `inferno`, `magma`, `viridis`, `turbo`
- **平台/工具箱要求 / Platform**: 纯 MATLAB，无工具箱依赖。
- **遮蔽/重复分析 / Shadowing & duplication**: 同 `viridis`，`plasma` **非 MATLAB 内置**，不构成内置遮蔽；
  仅与同名 File Exchange 实现有命名冲突风险。

### `cmyk2rgb` — CMYK 转 sRGB
- **文件 / File**: [external/cmyk2rgb.m](file:///c:/VSWork/gptoolbox-1/external/cmyk2rgb.m#L1-L3)
- **签名 / Signature**: `rgb = cmyk2rgb(cmyk)`
- **署名 / Attribution**: **无可考署名 / no attribution in header**——该文件仅 3 行，无注释、无作者、无许可证。
- **功能 / Purpose**: `EN: Convert CMYK color values to sRGB using the Image Processing Toolbox ICC-profile pipeline (makecform/applycform).` /
  中文：用 Image Processing Toolbox 的 ICC 配置管线（`makecform`/`applycform`）把 CMYK 转为 sRGB。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `cmyk` | `double`（值域 0–1） | CMYK 数据；形状与输出一致 / CMYK data |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `rgb` | `double`（值域 0–1） | 对应的 sRGB 数据 / sRGB result |

- **可选参数 / Optional**: 无 / None
- **变量流动 / Variable flow**: `cmyk →(1 makecform('cmyk2srgb') → cform)→(2 applycform(cmyk, cform))→ rgb`
- **算法步骤 / Algorithm**:
  1. `makecform('cmyk2srgb')` 构造颜色变换结构（依赖 ICC 配置文件）（L2）。
  2. `applycform(cmyk, cform)` 应用该变换，返回 sRGB（L2）。
  3. 整个函数体只有一条语句，无分支、无中间变量。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 cmyk"] --> B["cform = makecform('cmyk2srgb')"]
    B --> C["rgb = applycform(cmyk, cform)"]
    C --> Z["输出 rgb"]
  ```

- **依赖 / Dependencies**: `makecform`、`applycform`（**Image Processing Toolbox**）；无本库其它依赖。
- **See also**: `rgb2cmyk`
- **平台/工具箱要求 / Platform**: **必须安装 Image Processing Toolbox**（`makecform`/`applycform` 属 IPT）
  且本机需有可用的 CMYK↔sRGB ICC 配置文件，否则运行报错。
- **备注 / Note**: 非内置、gptoolbox 无同名函数，**无遮蔽、无重复**。此文件是"无头注释"的极短包装。

### `rgb2cmyk` — sRGB 转 CMYK
- **文件 / File**: [external/rgb2cmyk.m](file:///c:/VSWork/gptoolbox-1/external/rgb2cmyk.m#L1-L3)
- **签名 / Signature**: `cmyk = rgb2cmyk(rgb)`
- **署名 / Attribution**: **无可考署名 / no attribution in header**——仅 3 行，无注释、无作者、无许可证。
- **功能 / Purpose**: `EN: Convert sRGB color values to CMYK using the Image Processing Toolbox ICC-profile pipeline (makecform/applycform).` /
  中文：用 Image Processing Toolbox 的 ICC 配置管线把 sRGB 转为 CMYK。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `rgb` | `double`（值域 0–1） | sRGB 数据 / sRGB data |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `cmyk` | `double`（值域 0–1） | 对应的 CMYK 数据 / CMYK result |

- **可选参数 / Optional**: 无 / None
- **变量流动 / Variable flow**: `rgb →(1 makecform('srgb2cmyk') → cform)→(2 applycform(rgb, cform))→ cmyk`
- **算法步骤 / Algorithm**:
  1. `makecform('srgb2cmyk')` 构造颜色变换结构（L2）。
  2. `applycform(rgb, cform)` 应用该变换，返回 CMYK（L2）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 rgb"] --> B["cform = makecform('srgb2cmyk')"]
    B --> C["cmyk = applycform(rgb, cform)"]
    C --> Z["输出 cmyk"]
  ```

- **依赖 / Dependencies**: `makecform`、`applycform`（**Image Processing Toolbox**）；无本库其它依赖。
- **See also**: `cmyk2rgb`
- **平台/工具箱要求 / Platform**: **必须安装 Image Processing Toolbox** 及可用 ICC 配置文件。
- **备注 / Note**: 非内置、无同名冲突。此文件是 `cmyk2rgb` 的逆变换包装。

### `hex2rgb` — 十六进制颜色转 RGB
- **文件 / File**: [external/hex2rgb.m](file:///c:/VSWork/gptoolbox-1/external/hex2rgb.m#L1-L111)
- **签名 / Signature**: `[ rgb ] = hex2rgb(hex,range)`
- **署名 / Attribution**: 头注 `Chad A. Greene, April 2014`（"Updated August 2014 … Thanks to Stephen Cobeldick for
  the improvement tips"，见 [external/hex2rgb.m#L47-L59](file:///c:/VSWork/gptoolbox-1/external/hex2rgb.m#L47-L59)）；
  **头注无许可证声明**（只有作者与致谢）。
- **功能 / Purpose**: `EN: Convert hex color values to an n-by-3 RGB array, scaled to 0–1 by default or 0–255 when range is 256.` /
  中文：把十六进制颜色值转为 `n by 3` RGB；默认缩放到 **0–1**，给定 `range=256` 时为 **0–255**。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `hex` | `char` 矩阵 / `cell` | 十六进制颜色，可带或不带前导井号；支持 `#RRGGBB`、`RRGGBB`、三位简写 `abc`（展开为 `aabbcc`）/ hex color strings |
  | `range` | 标量（**可选**） | `1` → 0–1 缩放；`255` 或 `256` → 0–255 缩放 / output range selector |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `rgb` | `n by 3` | RGB 数组（0–1 或 0–255，取决于 `range`） |

- **可选参数 / Optional**:

  | 名称 | 取值 | 默认 | 含义 |
  |---|---|---|---|
  | `range` | `1` / `255` / `256`（标量） | `1`（L89-L91） | `1`→缩放 0–1；`255`/`256`→缩放 0–255；其它值报错 |

- **输入形式 / Accepted input forms**: `'#334D66'`（带井号）、`'334D66'`（不带井号）、
  三位简写 `'abc'`（`size(hex,2)==3` 时按 `hex(:,[1 1 2 2 3 3])` 展开为 6 位）、
  以及 `cell`（如 `{'#334D66';'#8099B3'}`，行向量会被转置，随后 `cell2mat`）。
- **变量流动 / Variable flow**: `hex,range →(1 assert nargin∈{1,2}; range 为标量)→(2 cell→cell2mat 并转置为列; 去首列井号)→(3 size(hex,2)==3 时三位展开六位)→(4 sscanf(hex.','%2x') 解析→ reshape 3×n → .')→ rgb0 →(5 range==1 → rgb=rgb0/255 : range∈{255,256} → rgb=rgb0)→ rgb`
- **算法步骤 / Algorithm**:
  1. 输入校验：`assert(nargin>0&nargin<3,...)`；`nargin==2` 时 `assert(isscalar(range)==1,...)`（L64–L68）。
  2. 若 `hex` 是 `cell`：校验为向量，行向量转置为列，`hex = cell2mat(hex)`（L72–L83）。
  3. 若首字符是井号 `strcmpi(hex(1,1),'#')`：删除第一列（L85–L87）。
  4. `nargin==1` 时 `range = 1`（L89–L91）。
  5. 若 `size(hex,2)==3`（三位简写）：`hex = hex(:,[1 1 2 2 3 3])` 展开为六位（L93–L96）。
  6. `switch range`（L100–L109）：`1` → `reshape(sscanf(hex.','%2x'),3,[]).'/255`；
     `{255,256}` → `reshape(sscanf(hex.','%2x'),3,[]).'`；其它 → `error(...)`。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 hex, range"] --> B{"nargin 合法 (1..2) ?"}
    B -- 否 --> B1["assert 失败"]
    B -- 是 --> C{"hex 为 cell ?"}
    C -- 是 --> C1["校验收 cell2mat 并转置为列"]
    C -- 否 --> D
    C1 --> D{"首字符为井号 ?"}
    D -- 是 --> D1["删除首列"]
    D -- 否 --> E
    D1 --> E{"nargin 等于 1 ?"}
    E -- 是 --> E1["range = 1"]
    E -- 否 --> F
    E1 --> F{"size(hex,2) 等于 3 ?"}
    F -- 是 --> F1["三位简写展开为六位"]
    F -- 否 --> G
    F1 --> G{"range 分支"}
    G -- "1" --> G1["sscanf %2x + reshape + /255 得 0..1"]
    G -- "255 或 256" --> G2["sscanf %2x + reshape 得 0..255"]
    G -- "其它" --> G3["error"]
    G1 --> Z["输出 rgb"]
    G2 --> Z
  ```

- **依赖 / Dependencies**: `sscanf`、`reshape`、`cell2mat`、`strcmpi`、`isrow`、`isvector`、`isscalar`、`assert`（MATLAB 内置）；无本库其它依赖。
- **See also**: `rgb2hex`, `html_color`, `dec2hex`, `hex2num`, `ColorSpec`
- **平台/工具箱要求 / Platform**: 纯 MATLAB，无工具箱依赖。
- **重复分析 / Duplication**: gptoolbox 无同名函数，**不遮蔽内置**；但功能与自家
  [`utility/html_color.m`](file:///c:/VSWork/gptoolbox-1/utility/html_color.m#L1-L45) **部分重叠**——
  两者都把"颜色记号"转为 0–1 RGB，只是 `html_color` 处理 **HTML 颜色名**（`aqua`/`red`…），
  而 `hex2rgb` 处理 **十六进制串**，输入域不重叠，因此是"分工"而非"完全重复"。
  实际关系是**被依赖**：gptoolbox 自家的 [`utility/zoe.m`](file:///c:/VSWork/gptoolbox-1/utility/zoe.m#L1-L21) 与
  [`utility/lipmanya.m`](file:///c:/VSWork/gptoolbox-1/utility/lipmanya.m#L1-L25) 均调用本函数，
  `mesh/readSVG.m`、`mesh/readSVG_cubics.m` 亦引用之。

### `rgb2hex` — RGB 转十六进制颜色
- **文件 / File**: [external/rgb2hex.m](file:///c:/VSWork/gptoolbox-1/external/rgb2hex.m#L1-L64)
- **签名 / Signature**: `[ hex ] = rgb2hex(rgb)`
- **署名 / Attribution**: 头注 `Chad A. Greene, April 2014`（"Updated August 2014 … Thanks to Stephen Cobeldick
  for his suggestions"，见 [external/rgb2hex.m#L33-L40](file:///c:/VSWork/gptoolbox-1/external/rgb2hex.m#L33-L40)）；
  **头注无许可证声明**。
- **功能 / Purpose**: `EN: Convert n-by-3 RGB values (0–1 or 0–255) to hex color strings of the form #RRGGBB.` /
  中文：把 `n by 3` 的 RGB（0–1 或 0–255）转为 `#RRGGBB` 形式的十六进制字符串。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `rgb` | `n by 3` `numeric` | RGB 值；若最大值 ≤ 1 视为 0–1 域，否则视为 0–255 域 / numeric triple(s) |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `hex` | `n by 7` `char` | 每行形如 `#RRGGBB` 的十六进制字符串 / hex color strings |

- **可选参数 / Optional**: 无 / None（**输入域由数据自动判定**：`max(rgb(:))<=1` → 0–1；否则 0–255）
- **变量流动 / Variable flow**: `rgb →(1 assert nargin==1 且 numeric)→(2 assert 第二维==3 且 全体∈[0,255])→(3 max(rgb(:))<=1 ? rgb=round(rgb*255) : rgb=round(rgb))→(4 sprintf('%02X',rgb.') → reshape 6×n → .' 写入 hex(:,2:7))→(5 hex(:,1)='#')→ hex`
- **算法步骤 / Algorithm**:
  1. 输入校验：`assert(nargin==1,...)`、`assert(isnumeric(rgb)==1,...)`（L44–L45）。
  2. `sizergb=size(rgb)`；`assert(sizergb(2)==3,...)`；`assert(max(rgb(:))<=255 & min(rgb(:))>=0,...)`（L47–L49）。
  3. 缩放：若 `max(rgb(:))<=1` → `rgb=round(rgb*255)`，否则 `rgb=round(rgb)`（L52–L56）。
  4. `hex(:,2:7) = reshape(sprintf('%02X',rgb.'),6,[]).'`（把 3 通道拼成 6 位十六进制）（L60）。
  5. `hex(:,1) = '#'` 填前导井号（L61）。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 rgb"] --> B{"nargin 等于 1 且 numeric ?"}
    B -- 否 --> B1["assert 失败"]
    B -- 是 --> C{"第二维等于 3 且值域在 0..255 ?"}
    C -- 否 --> C1["assert 失败"]
    C -- 是 --> D{"max(rgb(:)) 小于等于 1 ?"}
    D -- 是 --> D1["rgb = round(rgb*255)"]
    D -- 否 --> D2["rgb = round(rgb)"]
    D1 --> E
    D2 --> E["sprintf('%02X') 拼接 6 位并 reshape 到 n 行"]
    E --> F["hex 首列填井号"]
    F --> Z["输出 hex (n by 7)"]
  ```

- **依赖 / Dependencies**: `sprintf`、`reshape`、`round`、`size`、`assert`（MATLAB 内置）；无本库其它依赖。
- **See also**: `hex2rgb`, `html_color`, `dec2hex`, `hex2num`, `ColorSpec`
- **平台/工具箱要求 / Platform**: 纯 MATLAB，无工具箱依赖。
- **重复分析 / Duplication**: gptoolbox 无同名函数，**不遮蔽内置**；是 `hex2rgb` 的**逆变换**，
  与 [`utility/html_color.m`](file:///c:/VSWork/gptoolbox-1/utility/html_color.m#L1-L45) 的"RGB→名称"方向**语义相近但输出格式不同**
  （十六进制串 vs HTML 颜色名），非完全重复。

---

## 函数清单核对表 / Coverage Checklist

> 本目录 11 个文件逐一对应条目；锚点均可点击跳转至源码真实行范围。

| # | 函数 / File | 锚点 / Anchor | 署名 / Attribution | 一句话用途 / One-line purpose |
|---|---|---|---|---|
| 1 | `cbrewer` | [external/cbrewer.m](file:///c:/VSWork/gptoolbox-1/external/cbrewer.m#L1-L140) | Charles Robert（BSD 2011） | 生成 ColorBrewer 色表（可插值扩展） |
| 2 | `interpolate_cbrewer` | [external/interpolate_cbrewer.m](file:///c:/VSWork/gptoolbox-1/external/interpolate_cbrewer.m#L1-L61) | Charles Robert（BSD 2011） | 把色表插值到 `ncolors` 级 |
| 3 | `plot_brewer_cmap` | [external/plot_brewer_cmap.m](file:///c:/VSWork/gptoolbox-1/external/plot_brewer_cmap.m#L1-L84) | Charles Robert（BSD 2011） | 一个 figure 画出全部 ColorBrewer 色表 |
| 4 | `viridis` | [external/viridis.m](file:///c:/VSWork/gptoolbox-1/external/viridis.m#L1-L296) | Ander Biguri 2016（BSD）；数据源自 matplotlib | matplotlib viridis 色图 |
| 5 | `inferno` | [external/inferno.m](file:///c:/VSWork/gptoolbox-1/external/inferno.m#L1-L296) | Ander Biguri 2016（BSD）；数据源自 matplotlib | matplotlib inferno 色图 |
| 6 | `magma` | [external/magma.m](file:///c:/VSWork/gptoolbox-1/external/magma.m#L1-L299) | Ander Biguri 2016（BSD）；数据源自 matplotlib | matplotlib magma 色图 |
| 7 | `plasma` | [external/plasma.m](file:///c:/VSWork/gptoolbox-1/external/plasma.m#L1-L298) | Ander Biguri 2016（BSD）；数据源自 matplotlib | matplotlib plasma 色图 |
| 8 | `cmyk2rgb` | [external/cmyk2rgb.m](file:///c:/VSWork/gptoolbox-1/external/cmyk2rgb.m#L1-L3) | 无可考署名 | CMYK → sRGB（需 IPT） |
| 9 | `rgb2cmyk` | [external/rgb2cmyk.m](file:///c:/VSWork/gptoolbox-1/external/rgb2cmyk.m#L1-L3) | 无可考署名 | sRGB → CMYK（需 IPT） |
| 10 | `hex2rgb` | [external/hex2rgb.m](file:///c:/VSWork/gptoolbox-1/external/hex2rgb.m#L1-L111) | Chad A. Greene 2014（无许可证声明） | 十六进制 → RGB（0–1 或 0–255） |
| 11 | `rgb2hex` | [external/rgb2hex.m](file:///c:/VSWork/gptoolbox-1/external/rgb2hex.m#L1-L64) | Chad A. Greene 2014（无许可证声明） | RGB → 十六进制串 |

---

## 遮蔽 / 重复 一览 (Shadowing & Duplication Summary)

| 类型 / Kind | 具体 / Detail | 结论 / Verdict |
|---|---|---|
| **内置遮蔽 (builtin shadow)** | `viridis`/`inferno`/`magma`/`plasma` vs MATLAB 内置 | **不构成遮蔽**：MATLAB 未内置这 4 个色图；R2020b 新增的内置色图仅 `turbo`。仅存在与同名 File Exchange 实现的冲突风险 |
| **内置遮蔽（旁证）** | gptoolbox 自家 [`utility/turbo.m`](file:///c:/VSWork/gptoolbox-1/utility/turbo.m#L1-L22) vs R2020b 内置 `turbo` | 真实遮蔽（本章范围外，作对照） |
| **功能重复 (duplication)** | `hex2rgb`/`rgb2hex` vs [`utility/html_color.m`](file:///c:/VSWork/gptoolbox-1/utility/html_color.m#L1-L45) | **部分重叠**：`html_color` 处理 HTML 颜色名，`hex2rgb`/`rgb2hex` 处理十六进制串，输入/输出域不同（分工而非重复） |
| **被依赖关系 (used-by)** | `hex2rgb` ← [`utility/zoe.m`](file:///c:/VSWork/gptoolbox-1/utility/zoe.m#L1-L21), [`utility/lipmanya.m`](file:///c:/VSWork/gptoolbox-1/utility/lipmanya.m#L1-L25), `mesh/readSVG.m`, `mesh/readSVG_cubics.m` | vendored 函数被 gptoolbox 自家代码调用 |
| **无冲突 (no conflict)** | `cbrewer`/`interpolate_cbrewer`/`plot_brewer_cmap`/`cmyk2rgb`/`rgb2cmyk` | 无内置遮蔽、无同名 gptoolbox 函数 |

---

## 交叉引用 / Cross references

- gptoolbox **自家**色图与颜色常量（`turbo`/`zoe`/`pride`/`lipmanya`/`html_color` 等）：见
  [40.2 绘图辅助与配色](../40-utility/02-plotting-and-colors.md)
- 十六进制颜色相关的上游/下游用法：`utility/zoe.m`、`utility/lipmanya.m`、`mesh/readSVG.m`
- `external/` 其余非配色类 vendored 代码（fast marching、exact geodesics、Hungarian、`tilefigs` 等）：
  见 `90-external/` 下的姊妹文档
- 文档写作规范：见 [appendices/03-doc-template.md](../appendices/03-doc-template.md)
- 内置色图版本依据 / Builtin-release reference：
  [MATLAB turbo（R2020b 推出）](https://ww2.mathworks.cn/help/matlab/ref/turbo.html)、
  [MATLAB jet](https://cn.mathworks.com/help/matlab/ref/jet.html)
