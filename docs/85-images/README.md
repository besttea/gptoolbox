# 85 images 资源模块 / Images Asset Module

> `images/` 是 gptoolbox 的**示例素材目录**：包含 **1 个 `.m` 文件** `imdata`，以及 **14 个示例图像素材**。
> `imdata` 把图像文件内容读取为 **RFC-2397 data URI**（`data:<mime>;base64,<...>`），主要用于把贴图
> **内联**进网格导出文件（如 glTF）。
>
> `images/` is gptoolbox's **demo-asset directory**: **one `.m` file** (`imdata`) plus **14 demo image assets**.
> `imdata` reads an image file and returns its content as an **RFC-2397 data URI**
> (`data:<mime>;base64,<...>`), typically to **inline** a texture into an exported mesh file (e.g. glTF).

- 语言 / Language：中英对照（中文为主）
- 覆盖 / Coverage：`images/*.m` **1 / 1**（100%）+ 14 个素材已清点（见 [85.1 章节](01-images-module.md)）
- 粒度 / Granularity：逐函数 API + 素材清单 + 真实现状 + 算法 + 流程图 + 变量流动

---

## 1. 章节结构 / Chapter layout

| 章节 | 文件 | 函数数 | 主题 |
|---|---|---|---|
| 85.1 | [01-images-module.md](01-images-module.md) | 1 | 图像文件 → base64 data URI / Image file → base64 data URI |
| — | **合计 / Total** | **1** | — |

---

## 2. 模块职责 / Module responsibility

- **贴图内联**：为 glTF 等需要自包含贴图的导出格式，把图像字节编成 data URI 字符串
  （`writeGLTF` 在写贴图时调用 `imdata`）。
- **素材存放**：`images/` 同时是**示例图像**的存放处（`baboon.jpg`、`david.png` 等 14 个），
  供 demo / 文档 / 测试使用。
- 模块**只有 1 个函数** `imdata`，无内部调用；是依赖图中的**叶子**，仅被 `mesh/writeGLTF` 调用。

---

## 3. 全量函数索引 / Complete function index

| 函数 (anchor) | 文件 | 一句话功能 |
|---|---|---|
| [`imdata`](file:///c:/VSWork/gptoolbox-1/images/imdata.m#L1-L35) | [images/imdata.m](file:///c:/VSWork/gptoolbox-1/images/imdata.m#L1-L35) | 按扩展名判定 MIME，把图像文件读成 `data:<mime>;base64,...` 字符串 |

---

## 4. 目录资产清单 / Directory asset inventory

`images/` 共 **15 个文件** = **1 个 `.m`** + **14 个图像素材**。

| 文件 | 类型 | `imdata` 支持 | 说明 |
|---|---|---|---|
| `baboon.jpg` | JPEG | ✅ `image/jpeg` | 演示贴图 |
| `david.png` | PNG | ✅ `image/png` | 演示贴图 |
| `dorgan-brothers.jpg` | JPEG | ✅ `image/jpeg` | 演示贴图 |
| `gorilla-yawning-fake-depth.jpg` | JPEG | ✅ `image/jpeg` | 伪深度图 |
| `gorilla-yawning.png` | PNG | ✅ `image/png` | 演示贴图 |
| `gorilla-yawning.psd` | PSD | ❌ 抛错 | Photoshop 源文件 |
| `gptoolbox-bounce.gif` | GIF | ❌ 抛错 | 动图素材 |
| `half-black-half-white-square.png` | PNG | ✅ `image/png` | 二值测试图 |
| `hans-hass.jpg` | JPEG | ✅ `image/jpeg` | 演示贴图 |
| `max-schmeling-landscape.png` | PNG | ✅ `image/png` | 演示贴图 |
| `max-schmeling.jpg` | JPEG | ✅ `image/jpeg` | 演示贴图 |
| `nuclear-blast-observers.png` | PNG | ✅ `image/png` | 演示贴图 |
| `nuclear-blast.png` | PNG | ✅ `image/png` | 演示贴图 |
| `otto-freundlich.jpg` | JPEG | ✅ `image/jpeg` | 演示贴图 |
| `imdata.m` | MATLAB | — | 本章记录的函数 / the only `.m` file |

> 统计：**PNG 6 个、JPG 6 个、PSD 1 个、GIF 1 个，共 14 个素材**。
> Counts: 6 PNG, 6 JPG, 1 PSD, 1 GIF (14 assets).

---

## 5. 关键结论 / Key findings

- **支持格式有限**：`imdata` 仅支持 `.png` / `.jpg` / `.jpeg` / `.bmp`；素材中的
  `gptoolbox-bounce.gif` 与 `gorilla-yawning.psd` **无法**被读取，命中 `error('Unsupported image format')`，
  无格式转换/降级回退。
- **不做路径解析**：`imdata` **没有**使用 `mfilename('fullpath')` / `fileparts` 去定位 `images/` 资源目录，
  也**不会**自动查找内置素材；它把入参 `src` **原样**交给 `fopen`。调用方需传入
  相对于当前工作目录（或 `fopen` 可解析）的路径，且 `images/` 需在搜索路径上
  （`CLAUDE.md` 的 `addpath(... 'images' ...)`）。
- **无显式存在性检查**：未校验 `fopen` 是否成功（失败返回 `-1`），文件缺失会在 `fread` 阶段报错。
- **实现来源**：改编自 **Thomas Spriestersbach（2018-11-05）** 的代码（头注 L10-L11）；
  文件名与函数名一致，符合 `style.md`。

---

## 6. 依赖与构建 / Dependencies & build

- **同库依赖 / Same-library deps**：`base64encode`（[utility/base64encode.m](file:///c:/VSWork/gptoolbox-1/utility/base64encode.m#L1-L40)，
  自实现快速 Base64）。这也是 `images/` 指向 `utility/` 的唯一出边（`images → utility`）。
- **被调用方 / Callers（入边）**：`mesh/writeGLTF`（[mesh/writeGLTF.m L273](file:///c:/VSWork/gptoolbox-1/mesh/writeGLTF.m#L273)），
  写 glTF 贴图时把图像转 data URI。
- **外部/内置 / Built-ins**：`lower`、`endsWith`、`error`、`fopen`、`fread`、`fclose`；
  注释中提到的 `matlab.net.base64encode`（[L33](file:///c:/VSWork/gptoolbox-1/images/imdata.m#L30-L34)）为**已弃用**的旧实现，当前未启用。
- **构建 / Build**：**纯 MATLAB，无需编译**。

---

## 7. 交叉引用 / Cross references

- 本章章节：[85.1 images 资源模块](01-images-module.md)
- glTF 导出与贴图内联（`writeGLTF`）：[10-mesh/02 io-write](../10-mesh/02-io-write.md)
- Base64 编解码工具（`base64encode`/`base64decode`）：[40-utility/01](../40-utility/01-utilities-core.md)
- 模块架构中 `images/` 的角色：[00-overview/02-architecture.md](../00-overview/02-architecture.md)
- 文档写作规范：[appendices/03-doc-template.md](../appendices/03-doc-template.md)
