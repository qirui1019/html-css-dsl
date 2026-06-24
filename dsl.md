# SEESII DSL 规则

## 1. 定义

SEESII DSL 是说明书内容结构语言。它只描述“有什么内容、内容是什么结构、按什么顺序出现”。

DSL 不描述 UI，不描述 CSS，不描述页面坐标，不描述具体排版数值。

实际 Schema 文件是：

```text
seesii.dsl.json
```

DSL 由 Transformer 转换为当前项目的 `index.html` 结构，再由 `style.CSS` 和 Vivliostyle 完成 PDF 排版。

## 2. 顶层结构

实例 DSL 必须包含：

```json
{
  "meta": {},
  "cover": {},
  "catalog": {},
  "content": {}
}
```

## 3. 不属于 DSL 的内容

以下内容不得写入实例 DSL：

- HTML 标签名
- CSS class
- CSS style
- margin / padding / top / left / right / bottom
- width / height
- px / mm / pt
- position / display / flex / grid
- clip-path
- 图片文件路径
- 固定模板资产路径
- 封面和目录中的固定 logo / 箭头装饰

这些由 HTML、CSS、Transformer 或 Layout Engine 负责。

## 4. 富文本

需要局部加粗、斜体、下划线或禁止换行的文本使用 `richText`：

```json
[
  { "text": "Read all " },
  { "text": "safety warnings", "marks": ["bold"] },
  { "text": " before use." }
]
```

支持的 `marks`：

- `bold`
- `italic`
- `underline`
- `regular`
- `nowrap`

## 5. meta

`meta` 描述说明书元信息。

必填字段：

- `brand`
- `language`
- `version`

这些字段当前不直接绑定到页面元素，但用于生成流程识别说明书来源、语言和版本。

## 6. cover

`cover` 描述封面可变内容。

必填字段：

- `productName`
- `manualType`
- `importantNote`
- `productImage`
- `model`
- `supportEmail`

结构：

```json
{
  "productName": "CORDLESS NAIL GUN",
  "manualType": "INSTRUCTION MANUAL",
  "importantNote": {
    "title": "IMPORTANT NOTE:",
    "text": [{ "text": "Please read carefully before use." }]
  },
  "productImage": {
    "description": "cover product placeholder"
  },
  "model": [{ "text": "HKF50" }],
  "supportEmail": [{ "text": "seesii-tiktok-shop@outlook.com" }]
}
```

`productImage` 只表示封面产品图占位符，不包含图片路径。

## 7. catalog

`catalog` 只描述目录页固定文本内容。

必填字段：

- `title`
- `supportEmail`

示例：

```json
{
  "title": "TABLE OF CONTENTS",
  "supportEmail": [{ "text": "seesii-tiktok-shop@outlook.com" }]
}
```

实例 DSL 不写 `catalog.sections`。

目录由 Transformer 从 `content.chapters` 自动生成。

目录只包含：

- 一级标题：`chapter.title`
- 二级标题：`block.type = "h2"`

三级标题 `h3` 不进入目录。

## 8. content

`content` 描述正文。

必填字段：

- `chapters`

```json
{
  "chapters": []
}
```

## 9. chapter

每个 chapter 表示一个一级章节。

必填字段：

- `id`
- `title`
- `blocks`

可选字段：

- `toc`

示例：

```json
{
  "id": "sec-safety-warnings",
  "title": "1. SAFETY WARNINGS",
  "blocks": []
}
```

`id` 用于正文标题锚点和目录页码计算。

`toc: false` 表示该 chapter 不进入目录。

## 10. id 生成规则

AI 生成实例 DSL 时，所有 chapter、h2、h3 都必须提供唯一 `id`。

`id` 用于：

- 正文标题锚点。
- 目录 `href`。
- Vivliostyle 页码计算。

生成规则：

1. 去掉标题前面的章节编号，例如 `1.`、`1.1`、`1.1.1`。
2. 转为小写。
3. 删除特殊符号。
4. 将空格转换为短横线。
5. 添加 `sec-` 前缀。
6. 如果重复，追加 `-2`、`-3`。

示例：

| 标题 | id |
|---|---|
| `1. SAFETY WARNINGS` | `sec-safety-warnings` |
| `2. Functional Description and Specifications` | `sec-functional-description-and-specifications` |
| `1.1 General Safety Warnings` | `sec-general-safety-warnings` |
| `1.1.1 Electrical Safety` | `sec-electrical-safety` |

`id` 必须符合 `seesii.dsl.json` 中的格式：小写字母开头，只包含小写字母、数字和短横线。

## 11. block

`chapter.blocks[]` 是正文内容流。每个 block 必须有 `type`。

当前支持：

- `h2`
- `h3`
- `text`
- `warning`
- `image`
- `imagePair`
- `table`

block 的顺序就是正文渲染顺序。

## 12. h2 / h3

二级标题：

```json
{
  "type": "h2",
  "id": "sec-general-safety",
  "title": "1.1 General Safety"
}
```

三级标题：

```json
{
  "type": "h3",
  "id": "sec-electrical-safety",
  "title": "1.1.1 Electrical Safety"
}
```

`h2` 默认进入目录。

`h3` 不进入目录。

`toc: false` 可以让 `h2` 不进入目录。

## 13. text

```json
{
  "type": "text",
  "content": [
    { "text": "Keep all warnings and instructions for future reference." }
  ]
}
```

`content` 使用 richText。

## 14. warning

```json
{
  "type": "warning",
  "items": [
    {
      "icon": { "description": "warning icon placeholder" },
      "title": "CAUTION:",
      "content": [{ "text": "Always check the direction before operation." }]
    }
  ]
}
```

规则：

- `items` 至少一项。
- `content` 必填。
- `icon` 可选。
- `title` 可选。
- 一个 warning block 可以包含多个 warning item。
- 没有 icon 和 title 时，只渲染文本。

## 15. image

普通章节单图使用 `image`。

```json
{
  "type": "image",
  "image": {
    "description": "operation diagram placeholder",
    "intrinsicWidth": 1200,
    "intrinsicHeight": 800
  }
}
```

也可以使用比例：

```json
{
  "type": "image",
  "image": {
    "description": "operation diagram placeholder",
    "aspectRatio": 1.5
  }
}
```

`image` 不包含图片路径，只表示占位符。

单图必须提供：

- `intrinsicWidth` + `intrinsicHeight`

或：

- `aspectRatio`

## 16. imagePair

双图/单槽图使用 `imagePair`。

```json
{
  "type": "imagePair",
  "align": "left",
  "images": [
    { "description": "left placeholder" },
    { "description": "right placeholder" }
  ]
}
```

规则：

- `images` 可以有 1 到 2 项。
- 1 项时默认占左侧。
- `align: "right"` 时，1 项占右侧。
- 不包含图片路径。

## 17. table

表格只支持 hybrid 结构。

```json
{
  "type": "table",
  "table": {
    "structure": "hybrid",
    "sections": []
  }
}
```

支持的 table section：

- `kv`
- `matrix`
- `group`

### kv

```json
{
  "type": "kv",
  "rows": [
    ["Model", "HKF50"],
    ["Voltage", "21V"]
  ]
}
```

### matrix

```json
{
  "type": "matrix",
  "headers": ["Speed", "Torque"],
  "rows": [
    ["Low", "High"]
  ]
}
```

### group

```json
{
  "type": "group",
  "title": "US Charger",
  "rows": [
    ["Input", "100-240V"]
  ]
}
```

表格内容可以是字符串，也可以是 richText。

## 18. 模块间距

模块间距不属于实例 DSL 字段。

实例 DSL 不允许写：

- `spacing`
- `margin`
- `padding`
- `top`
- `left`
- `width`
- `height`
- `px`
- `mm`
- `pt`

具体模块间距规则见 `mapping.md` 的 `Module Spacing Rules`。

## 19. Vivliostyle

DSL 不直接控制 Vivliostyle。

Transformer 生成结构化 HTML 后，由 `style.CSS` 中的 Paged Media 规则完成分页。

正文中：

- chapter 可以跨页。
- text 可以自然跨页。
- warning / image / imagePair / table 尽量避免内部断页。
