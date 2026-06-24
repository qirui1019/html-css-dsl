# SEESII DSL → HTML 映射规则

## 1. 目标

本文档定义实例 DSL 如何转换为当前项目的 `index.html` 结构。

映射必须遵守：

- 使用现有 HTML class。
- 保留现有 `data-*` 结构。
- 不引入新的页面结构。
- 不把 DSL 字段转换成 inline style。
- 不把图片占位符转换成真实图片路径。

渲染优先级：

```text
DSL value > template default value > empty state
```

## 2. Page Mapping

| DSL | HTML |
|---|---|
| `cover` | `.page.page--cover` |
| `catalog` | `.page.page--catalog` |
| `content.chapters` | `.page.page--content .chapter` |

`.page--cover`、`.page--catalog`、`.page--content` 的分页规则由 `style.CSS` 控制。

## 3. Cover Mapping

| DSL path | HTML target |
|---|---|
| `cover.productName` | `.product-name[data-bind="cover.productName"]` |
| `cover.manualType` | `.manual-type[data-bind="cover.manualType"]` |
| `cover.importantNote.title` | `.important-note-title[data-bind="cover.importantNote.title"]` |
| `cover.importantNote.text` | `.important-note-text[data-bind="cover.importantNote.text"]` |
| `cover.productImage` | `.product-image[data-bind="cover.productImage"]` 和 `.product-image__img[data-bind="cover.productImage"]` |
| `cover.model` | `.bottom-info-box--model[data-bind="cover.model"]` |
| `cover.supportEmail` | `.bottom-info-box--email[data-bind="cover.supportEmail"]` |

固定资源不由 DSL 控制：

- 封面 logo：`assets/logo.png`
- 封面左下角箭头：`assets/threeGreyArrow.png`

`cover.productImage` 是占位符，不渲染真实图片。

## 4. Catalog Mapping

| DSL path | HTML target |
|---|---|
| `catalog.title` | `.catalog-title[data-bind="catalog.title"]` |
| `catalog.supportEmail` | `.catalog-support-email[data-bind="catalog.supportEmail"]` |

固定资源不由 DSL 控制：

- 目录 logo：`assets/logo.png`
- 目录右上角箭头：`assets/rightTopArrow.png`

目录列表挂载点：

```html
<nav
  class="module catalog-list catalog-list--normal"
  data-module-type="catalogList"
  data-bind-generated="toc"
  data-toc-source="content.chapters"
></nav>
```

实例 DSL 不写 `catalog.sections`。

Transformer 从 `content.chapters` 生成目录 DOM。

目录生成规则：

- `chapter.id` + `chapter.title` 生成一级目录项。
- `block.type = "h2"` 时，`block.id` + `block.title` 生成二级目录项。
- `block.type = "h3"` 不生成目录项。
- `chapter.toc === false` 时，该 chapter 不生成目录项。
- `h2.toc === false` 时，该 h2 不生成目录项。
- 目录项 `href` 必须等于 `#` + 对应的 `chapter.id` 或 `block.id`。

生成的目录项结构：

```html
<div class="catalog-node">
  <a class="element catalog-item" data-element-type="catalogItem" href="#section-id">
    <span class="catalog-item__title">Section Title</span>
    <span class="catalog-item__leader"></span>
  </a>
  <div class="catalog-children"></div>
</div>
```

页码由 CSS 和 Vivliostyle 计算：

```css
.catalog-item::after {
  content: target-counter(attr(href), page, decimal-leading-zero);
}
```

## 5. ID Mapping

Transformer 必须使用实例 DSL 中已有的 `id`。

如果实例 DSL 由 AI 生成，AI 应按 `dsl.md` 的 id 生成规则生成 `id`。

映射规则：

| DSL path | HTML output |
|---|---|
| `chapter.id` | H1 的 `id` |
| `block.id`，当 `block.type = "h2"` | H2 的 `id` |
| `block.id`，当 `block.type = "h3"` | H3 的 `id` |
| `chapter.id` | 一级目录项 `href="#{chapter.id}"` |
| `h2 block.id` | 二级目录项 `href="#{block.id}"` |

Transformer 不应在渲染阶段重新生成另一套 id。

## 6. Content / Chapter Mapping

`content.chapters[]` 克隆 `.chapter[data-bind="content.chapters"]`。

| DSL path | HTML target |
|---|---|
| `chapter.title` | `.chapter-title[data-bind="chapter.title"]` |
| `chapter.id` | `.chapter-title[data-bind-id="chapter.id"]` |
| `chapter.blocks[]` | `.chapter-body[data-bind="chapter.blocks"]` |

chapter 标题是 H1。

Transformer 必须把 `chapter.id` 写成 H1 的 `id`。

## 7. Block Mapping

Transformer 按 `chapter.blocks[]` 顺序生成 block。

### h2

DSL:

```json
{
  "type": "h2",
  "id": "sec-general-safety",
  "title": "1.1 General Safety"
}
```

HTML:

```html
<section class="block block--h2" data-type="h2" data-bind="block">
  <div class="title-bg" aria-hidden="true"></div>
  <h2
    class="element title section-title"
    data-element-type="secondaryTitle"
    data-bind="block.title"
    data-bind-id="block.id"
  ></h2>
</section>
```

### h3

DSL:

```json
{
  "type": "h3",
  "id": "sec-electrical-safety",
  "title": "1.1.1 Electrical Safety"
}
```

HTML:

```html
<section class="block block--h3" data-type="h3" data-bind="block">
  <h3
    class="element subsection-title"
    data-element-type="tertiaryTitle"
    data-bind="block.title"
    data-bind-id="block.id"
  ></h3>
</section>
```

### text

DSL:

```json
{
  "type": "text",
  "content": [{ "text": "Text content." }]
}
```

HTML:

```html
<section class="block block--text" data-type="text" data-bind="block">
  <div class="element text-content" data-element-type="textContent" data-bind="block.content"></div>
</section>
```

### warning

DSL:

```json
{
  "type": "warning",
  "items": [
    {
      "icon": { "description": "warning icon placeholder" },
      "title": "CAUTION:",
      "content": [{ "text": "Warning text." }]
    }
  ]
}
```

HTML:

```html
<section class="block block--warning" data-type="warning" data-bind="block">
  <div class="warning-content" data-bind="block.items" data-role="warning-items">
    <div class="warning-item" data-bind="item">
      <div class="warning-heading" data-role="warning-heading">
        <span class="element warning-icon" data-element-type="warningIcon" data-bind="item.icon" aria-hidden="true"></span>
        <strong class="element warning-title" data-element-type="warningTitle" data-bind="item.title"></strong>
      </div>
      <div class="element warning-text" data-element-type="warningText" data-bind="item.content"></div>
    </div>
  </div>
</section>
```

规则：

- `item.icon` 为空时，不渲染 `.warning-icon`。
- `item.title` 为空时，不渲染 `.warning-title`。
- icon 和 title 都为空时，可使用 `.warning-item--text-only`。

### image

DSL:

```json
{
  "type": "image",
  "image": {
    "description": "single image placeholder",
    "aspectRatio": 1.5
  }
}
```

HTML:

```html
<section class="block block--image block--image-single" data-type="image" data-variant="single" data-bind="block">
  <div
    class="element image-content image-content--single"
    data-element-type="imageContent"
    data-bind="block.image"
    data-bind-intrinsic-width="block.image.intrinsicWidth"
    data-bind-intrinsic-height="block.image.intrinsicHeight"
    data-bind-aspect-ratio="block.image.aspectRatio"
    role="img"
  ></div>
</section>
```

规则：

- 不生成 `<img>`。
- 不读取图片路径。
- Layout Engine 可根据 `intrinsicWidth` / `intrinsicHeight` / `aspectRatio` 设置占位符尺寸变量。

### imagePair

DSL:

```json
{
  "type": "imagePair",
  "align": "left",
  "images": [
    { "description": "first placeholder" },
    { "description": "second placeholder" }
  ]
}
```

HTML:

```html
<section class="block block--image-pair" data-type="imagePair" data-variant="pair" data-bind="block">
  <div class="image-pair" data-bind="block.images">
    <figure class="image-pair__slot" data-bind="image">
      <div class="element image-content image-content--pair" data-element-type="imageContent" data-bind="image" role="img"></div>
    </figure>
  </div>
</section>
```

规则：

- `images.length` 可以是 1 或 2。
- `align: "right"` 时，Transformer 在 `.image-pair` 上写入 `data-align="right"`。

### table

DSL:

```json
{
  "type": "table",
  "table": {
    "structure": "hybrid",
    "sections": []
  }
}
```

HTML:

```html
<section class="block block--table block--hybrid" data-type="table" data-structure="hybrid" data-bind="block.table">
  <div class="element table-content" data-element-type="tableContent">
    <div class="table-sections" data-role="table-sections">
      <div class="table-section table-section--kv" data-role="table-section-kv"></div>
      <div class="table-section table-section--matrix" data-role="table-section-matrix"></div>
      <div class="table-section table-section--group" data-role="table-section-group"></div>
    </div>
  </div>
</section>
```

## 8. Table Rendering

### kv

DSL:

```json
{
  "type": "kv",
  "rows": [["Model", "HKF50"]]
}
```

Render:

```html
<table>
  <tbody>
    <tr>
      <th>Model</th>
      <td>HKF50</td>
    </tr>
  </tbody>
</table>
```

Mount:

```html
[data-role="table-section-kv"]
```

### matrix

DSL:

```json
{
  "type": "matrix",
  "headers": ["Speed", "Torque"],
  "rows": [["Low", "High"]]
}
```

Render:

```html
<table>
  <thead>
    <tr>
      <th>Speed</th>
      <th>Torque</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Low</td>
      <td>High</td>
    </tr>
  </tbody>
</table>
```

Mount:

```html
[data-role="table-section-matrix"]
```

### group

DSL:

```json
{
  "type": "group",
  "title": "US Charger",
  "rows": [["Input", "100-240V"]]
}
```

Render:

```html
<div class="table-section__title">US Charger</div>
<table>
  <tbody>
    <tr>
      <th>Input</th>
      <td>100-240V</td>
    </tr>
  </tbody>
</table>
```

Mount:

```html
[data-role="table-section-group"]
```

## 9. RichText Mapping

| DSL mark | HTML |
|---|---|
| `bold` | `<strong>` |
| `italic` | `<em>` |
| `underline` | `<span class="text-mark--underline">` |
| `regular` | `<span class="text-mark--regular">` |
| `nowrap` | `<span class="text-mark--nowrap">` |

RichText 可用于：

- `cover.importantNote.text`
- `cover.model`
- `cover.supportEmail`
- `catalog.supportEmail`
- `block.content`
- `warning.items[].content`
- table cell

## 10. Module Spacing Rules

模块间距由 Layout Engine / CSS 执行，不写入实例 DSL。

| 前一模块 | 当前模块 | 间距 |
|---|---|---|
| 任意普通模块 | H1 / `.block--h1` | `10mm` |
| H1 / `.block--h1` | 正文 / 表格 / 警示框 / 图片 | `5mm` |
| 任意普通模块 | H2 / `.block--h2` | `8mm` |
| H1 / `.block--h1` | H2 / `.block--h2` | `5mm` |
| H2 / `.block--h2` | 后续模块 | `4mm` |
| 文本 / 警示框 / 表格 / 图片之间的普通相邻关系 | 后续普通模块 | `4mm` |
| 文本与示意图构成注释或演绎组合 | 组合内部 | `3mm` |
| 两个“文本 + 图片”组合 | 组合之间 | `6mm` |

文本和示意图组合应尽量作为不可分割单元处理。

## 11. Density Rules

目录密度 class：

- `.catalog-list--compact`
- `.catalog-list--normal`
- `.catalog-list--loose`

正文密度 class：

- `.content-density--compact`
- `.content-density--normal`
- `.content-density--loose`

实例 DSL 不控制这些 class。Layout Engine 根据内容量选择。

## 12. Transformer 禁止行为

Transformer 不得：

- 生成 inline `style`。
- 从 DSL 生成任意 CSS class。
- 把 `spacing`、`margin`、`padding`、`width`、`height` 等字段当作合法 DSL。
- 把图片占位符渲染成真实图片路径。
- 用截图替代表格、文本或 warning。
- 引入 JavaScript 控制渲染。

实例 DSL 如果出现 Schema 未开放字段，应校验失败。
