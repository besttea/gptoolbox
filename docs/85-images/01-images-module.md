# 85 images 资源模块 / Images Asset Module

> 本章覆盖 `images/` 目录。该目录包含 **1 个 `.m` 文件** `imdata`，以及 **14 个示例图像素材**（demo assets）。
> `imdata` 的作用是：把图像文件内容读取为 **RFC-2397 data URI**（`data:<mime>;base64,<...>`），
> 常用于把贴图**内联**进网格导出文件（如 glTF）。
>
> This chapter covers the `images/` directory: **one `.m` file** (`imdata`) plus **14 demo image assets**.
> `imdata` reads an image file and returns its content as an **RFC-2397 data URI**
> (`data:<mime>;base64,<...>`), typically to **inline** a texture into an exported mesh file (e.g. glTF).

- 语言 / Language：中英对照（中文为主）
- 相关章节 / See also：[85 images 资源模块索引](README.md)、[10-mesh/02 io-write](../10-mesh/02-io-write.md)（`writeGLTF` 调用 `imdata`）、
  [40-utility/01](../40-utility/01-utilities-core.md)（`base64encode` 所在模块）、
  [appendices/03-doc-template.md](../appendices/03-doc-template.md)
- 命名约定 / Notation：`src` 为图像文件路径；`data` 为 data URI 字符串

---

## 目录资产清单 / Directory asset inventory

`images/` 目录内容（`imdata.m` 为唯一的 `.m` 文件，其余为演示素材）：

| 文件 / File | 类型 | `imdata` 是否支持 | 说明 |
|---|---|---|---|
| `baboon.jpg` | JPEG | ✅ `image/jpeg` | 演示贴图 / demo jpg |
| `david.png` | PNG | ✅ `image/png` | 演示贴图 / demo png |
| `dorgan-brothers.jpg` | JPEG | ✅ `image/jpeg` | 演示贴图 |
| `gorilla-yawning-fake-depth.jpg` | JPEG | ✅ `image/jpeg` | 伪深度图 / fake depth |
| `gorilla-yawning.png` | PNG | ✅ `image/png` | 演示贴图 |
| `gorilla-yawning.psd` | PSD | ❌ 抛错 | Photoshop 源文件，`imdata` 不支持 |
| `gptoolbox-bounce.gif` | GIF | ❌ 抛错 | 动图素材；**`imdata` 不支持 `.gif`** |
| `half-black-half-white-square.png` | PNG | ✅ `image/png` | 二值测试图 |
| `hans-hass.jpg` | JPEG | ✅ `image/jpeg` | 演示贴图 |
| `max-schmeling-landscape.png` | PNG | ✅ `image/png` | 演示贴图 |
| `max-schmeling.jpg` | JPEG | ✅ `image/jpeg` | 演示贴图 |
| `nuclear-blast-observers.png` | PNG | ✅ `image/png` | 演示贴图 |
| `nuclear-blast.png` | PNG | ✅ `image/png` | 演示贴图 |
| `otto-freundlich.jpg` | JPEG | ✅ `image/jpeg` | 演示贴图 |
| `imdata.m` | MATLAB | — | 本章记录的函数 / the only `.m` file |

> 统计：PNG 6 个、JPG 6 个、PSD 1 个、GIF 1 个，共 14 个图像素材。
> ⚠️ 素材中的 `.psd` 与 `.gif` **无法**通过 `imdata` 读取（见"真实现状"）。
> Counts: 6 PNG, 6 JPG, 1 PSD, 1 GIF (14 image assets). `.psd` and `.gif` are **not** loadable via `imdata`.

---

## 逐函数条目 / Per-function entries

### `imdata` — 图像文件转 base64 data URI / Image file to base64 data URI
- **文件 / File**: [images/imdata.m](file:///c:/VSWork/gptoolbox-1/images/imdata.m#L1-L35)
- **签名 / Signature**: `data = imdata(src)`
- **功能 / Purpose**: `EN: Convert the image file content into a data URI scheme (RFC-2397) string.` / 中文：把图像文件内容读取并编码为 `data:<mime>;base64,<...>` 形式的 data URI 字符串。
- **输入 / Inputs**:

  | 参数 | 尺寸/类型 | 含义 (中 / EN) |
  |---|---|---|
  | `src` | `char`/`string` | 图像文件路径（扩展名大小写不敏感）/ image file path |

- **输出 / Outputs**:

  | 参数 | 尺寸/类型 | 含义 |
  |---|---|---|
  | `data` | `char` 行向量 | 形如 `data:image/png;base64,iVBORw0KGgo...` 的 data URI / data URI string |

- **可选参数 / Optional**: 无 / none（无 `varargin`，唯一入口参数为文件路径）
- **变量流动 / Variable flow**:
  `src →(lower 归一化小写)→ src →(endsWith 判定扩展名)→ fmt (MIME 类型) →(fopen/fread)→ B (文件字节列向量) →`
  `(base64encode(B))→ Bstr →(字符串拼接 `['data:' fmt ';base64,' Bstr]`)→ data`
- **算法步骤 / Algorithm**:
  1. 初始化 `fmt = []`，并把 `src = lower(src)` 归一化（大小写不敏感匹配扩展名）（L13-L14）。
  2. **扩展名 → MIME 判定**（L15-L23）：
     - `endsWith(src,'.png')` → `fmt = 'image/png'`；
     - `endsWith(src,'.jpg') || endsWith(src,'.jpeg')` → `fmt = 'image/jpeg'`；
     - `endsWith(src,'.bmp')` → `fmt = 'image/bmp'`；
     - 其它（含 `.gif`/`.psd`）→ `error('Unsupported image format')`。
  3. 读文件字节：`fp = fopen(src,'r'); B = fread(fp); fclose(fp)`（以只读方式打开并按数值列读入全部字节）（L25-L27）。
  4. Base64 编码：`Bstr = base64encode(B)`（调用本库 `utility/base64encode`，见"依赖"）（L28）。
  5. 拼接 data URI：`data = ['data:' fmt ';base64,' Bstr]`（L29）。
  6. **（被注释的旧实现）** L30-L34 保留了旧写法：用 `matlab.net.base64encode(bytes)` 编码并用 `assert` 断言与
     `base64encode` 结果一致；当前代码**不使用**该分支。
- **流程图 / Flowchart**:

  ```mermaid
  flowchart TD
    A["输入 src (文件路径)"] --> B["fmt = []; src = lower(src)"]
    B --> C{"扩展名?"}
    C -- .png --> C1["fmt = image/png"]
    C -- .jpg/.jpeg --> C2["fmt = image/jpeg"]
    C -- .bmp --> C3["fmt = image/bmp"]
    C -- 其它 --> E["error: Unsupported image format"]
    C1 --> D["fp = fopen(src,'r'); B = fread(fp); fclose(fp)"]
    C2 --> D
    C3 --> D
    D --> F["Bstr = base64encode(B)"]
    F --> G["data = [ 'data:' fmt ';base64,' Bstr ]"]
    G --> Z["输出 data (data URI 字符串)"]
  ```
- **依赖 / Dependencies**:
  - 同库依赖：`base64encode`（[utility/base64encode.m](file:///c:/VSWork/gptoolbox-1/utility/base64encode.m#L1-L40)，
    自实现的快速 Base64，替代 `matlab.net.base64encode`）。
  - MATLAB 内置：`lower`、`endsWith`、`error`、`fopen`、`fread`、`fclose`。
  - 非本库（仅注释中出现）：`matlab.net.base64encode`（旧实现，L33）。
- **See also**: `writeGLTF`（[mesh/writeGLTF.m L273](file:///c:/VSWork/gptoolbox-1/mesh/writeGLTF.m#L273)，把贴图内联进 glTF）、
  `base64encode`、`base64decode`（[utility/base64decode.m](file:///c:/VSWork/gptoolbox-1/utility/base64decode.m#L1)）
- **备注 / Note（真实现状 / actual behavior）**:
  - ⚠️ **不做路径解析（与部分预期不符）**：本函数**没有**使用 `mfilename('fullpath')` / `fileparts` 去解析
    `images/` 资源目录，也**不会**自动定位内置素材；它把入参 `src` **原样**交给 `fopen`。
    因此调用方需传入**相对于当前工作目录（或 `fopen` 可解析）**的路径；`images/` 目录需在 MATLAB 搜索路径上
    （见 `CLAUDE.md` 的 `addpath(... 'images' ...)`）。
  - ⚠️ **不支持 GIF/PSD**：素材中的 `gptoolbox-bounce.gif` 与 `gorilla-yawning.psd` **无法**被读取，
    会命中 `error('Unsupported image format')`（回退行为即"直接报错"，无格式转换/降级）。
  - ⚠️ **无显式存在性/打开检查**：未校验 `fopen` 是否成功（失败时返回 `-1`），文件缺失会在 `fread` 阶段报错。
  - 该文件是 **Thomas Spriestersbach（2018-11-05）** 代码的改编版（头注 L10-L11）；文件名与函数名一致，符合 `style.md`。

---

## 依赖汇总 / Dependencies summary

- **同库依赖**：`base64encode`（`utility/`）。`images/` 模块本身只暴露 `imdata` 一个函数，无内部调用。
- **被调用方（入边）**：`mesh/writeGLTF`（写 glTF 贴图时把图像转 data URI）。
- **外部/内置**：`fopen`/`fread`/`fclose`/`lower`/`endsWith`/`error`（MATLAB 内置）；
  注释中提到的 `matlab.net.base64encode` 为内置替代实现，当前未启用。
- ⚠️ **注意**：`gptoolbox-bounce.gif` 与 `gorilla-yawning.psd` 属于**素材但不受支持**的格式。

## 交叉引用 / Cross references

- glTF 导出与贴图内联：见 [10-mesh/02 io-write](../10-mesh/02-io-write.md)
- Base64 编解码工具：见 [40-utility/01 utilities-core](../40-utility/01-utilities-core.md)
- 模块概览（`images/` 的角色）：见 [00-overview/02-architecture.md](../00-overview/02-architecture.md)
- 文档写作规范：见 [appendices/03-doc-template.md](../appendices/03-doc-template.md)

---

## 函数清单核对表 / Coverage Checklist

| 函数 (anchor) | 一句话用途 |
|---|---|
| [`imdata`](file:///c:/VSWork/gptoolbox-1/images/imdata.m#L1-L35) | 按扩展名判定 MIME，把图像文件读成 `data:<mime>;base64,...` 字符串；仅支持 png/jpg/jpeg/bmp |
