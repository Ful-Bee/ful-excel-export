# Ful-ExcelExport 开源版

高性能纯前端 Excel 导出库

High-Performance Pure Frontend Excel Export Library

## 0. 获取代码 / Get the Code

```bash
# 克隆仓库 / Clone the repository
git clone https://github.com/Ful-Bee/Ful-ExcelExport.git

# 进入项目目录 / Enter project directory
cd Ful-ExcelExport
```

![Ful-ExcelExport Demo](./assets/public/proDemoPic.png)

### 导出效果示例 / Export Result Example

![Excel Export Result](./assets/public/excel-temp1.png)

_支持样式、富文本、合并单元格、边框等完整 Excel 功能_

_Supports styles, rich text, merged cells, borders and full Excel features_

---

## 1. 本地启动 Demo / Run Demo Locally

### 方式一：使用 npx serve（推荐）

```bash
# 进入项目目录（如果还没进入）
cd Ful-ExcelExport

# 启动本地服务器
npx serve .

# 浏览器访问
# http://localhost:3000/demo.html     (中文)
# http://localhost:3000/demo.en.html  (English)
```

### 方式二：使用 VS Code Live Server

1. 安装 VS Code 插件 `Live Server`
2. 右键 `demo.html` → `Open with Live Server`

### 方式三：使用 Python

```bash
cd demo
python -m http.server 8080
# 访问 http://localhost:8080/demo.html
```

> ⚠️ 注意：由于使用 ES Module，必须通过 HTTP 服务器访问，不能直接双击 HTML 文件打开。

---

## 2. 在项目中引用 / Usage in Your Project

### ESM 模块引入（推荐）

```javascript
import FulExcelExport from "./ful-excel-export.esm.js";

const exporter = new FulExcelExport({
  fileName: "导出文件.xlsx",
});
```

### UMD 方式（script 标签）

```html
<script src="ful-excel-export.umd.js"></script>
<script>
  const FulExcelExport = window.FulExcelExport;
  const exporter = new FulExcelExport({ fileName: "export.xlsx" });
</script>
```

---

## 3. API 文档 / API Documentation

### 核心类和方法一览 / Core Classes & Methods

| 类 / Class       | 方法 / Method           | 说明 / Description                        |
| ---------------- | ----------------------- | ----------------------------------------- |
| `FulExcelExport` | `constructor(options)`  | 创建导出器实例 / Create exporter instance |
| `FulExcelExport` | `addWorkSheet(options)` | 添加工作表 / Add worksheet                |
| `FulExcelExport` | `export()`              | 执行导出 / Execute export                 |
| `WorkSheet`      | `addRows(rows)`         | 批量添加行 / Add rows in batch            |
| `WorkSheet`      | `addRow(row)`           | 添加单行 / Add single row                 |

### 构造函数参数 / Constructor Options

| 参数 / Parameter      | 类型 / Type | 必填 | 说明 / Description                                       |
| --------------------- | ----------- | ---- | -------------------------------------------------------- |
| `fileName`            | string      | ✅   | 导出文件名 / Export file name                            |
| `onProgress`          | function    | ❌   | 进度回调 / Progress callback（见下方详细说明）           |
| `workSheetMaxRowsNum` | number      | ❌   | 每 Sheet 最大行数，超过自动分页（默认 1000000）          |
| `useStreamDownload`   | boolean     | ❌   | 是否启用流式下载（默认 false，使用 Blob 模式）           |
| `streamMitmUrl`       | string      | ❌   | 自定义 Service Worker 中转页面地址（启用流式下载时使用） |

### onProgress 回调参数详解

```javascript
exporter.onProgress((percent, stage, message, memory) => {
  console.log(percent, stage, message, memory);
});
```

| 参数 / Parameter | 类型 / Type | 说明 / Description                                               |
| ---------------- | ----------- | ---------------------------------------------------------------- |
| `percent`        | number      | 进度百分比 0-100                                                 |
| `stage`          | string      | 当前阶段：`init` / `building` / `compressing` / `done` / `error` |
| `message`        | string      | 状态消息，如 "[Sheet1] 已写入 50,000 行"                         |
| `memory`         | object      | 内存信息对象（见下方）                                           |

**memory 对象字段：**

| 字段 / Field | 类型 / Type | 说明 / Description |
| ------------ | ----------- | ------------------ |
| `usedHeap`   | number      | 已使用堆内存 (MB)  |
| `totalHeap`  | number      | 总堆内存 (MB)      |
| `limit`      | number      | 堆内存限制 (MB)    |

**stage 阶段说明：**

| 阶段 / Stage  | 说明 / Description  |
| ------------- | ------------------- |
| `init`        | 初始化导出          |
| `building`    | 正在生成 Excel 数据 |
| `compressing` | 正在压缩 ZIP 文件   |
| `done`        | 导出完成            |
| `error`       | 导出出错            |

### addWorkSheet 参数 / addWorkSheet Options

| 参数 / Parameter | 类型 / Type | 必填 | 说明 / Description               |
| ---------------- | ----------- | ---- | -------------------------------- |
| `name`           | string      | ❌   | 工作表名称 / Sheet name          |
| `columnCount`    | number      | ❌   | 列数（可自动检测）/ Column count |

---

### 3.1 创建导出器 / Create Exporter

```javascript
const exporter = new FulExcelExport({
  fileName: "导出文件.xlsx", // 文件名 / File name
  onProgress: (info) => {
    // 进度回调 / Progress callback
    console.log(info.percent, info.message);
  },
});
```

### 3.2 添加工作表 / Add Worksheet

```javascript
const worksheet = exporter.addWorkSheet({
  name: "Sheet1", // 工作表名称 / Sheet name
  columnCount: 10, // 列数 / Column count
});
```

### 3.3 添加数据行 / Add Data Rows

```javascript
// 添加表头
await worksheet.addRows([["ID", "名称", "数量", "金额"]]);

// 分页添加数据（推荐用于大数据量）
for (let page = 0; page < totalPages; page++) {
  const pageData = generatePageData(page); // 你的数据生成函数
  await worksheet.addRows(pageData);
}
```

### 3.4 单元格样式 / Cell Styles

```javascript
// 带样式的单元格
const row = [
  { value: "标题", bold: true, fontSize: 14, fill: "#f0f0f0" },
  { value: "金额", color: "#ff0000", align: "right" },
  { value: "<b>粗体</b><i>斜体</i>", border: true },
  // 混合 HTML 标签 + style 属性
  {
    value:
      '<b>重要：</b><span style="color: #ff0000;">错误数 <b>5</b> 处</span>，<span style="color: #00aa00;">正确数 <b>10</b> 处</span>',
    border: true,
  },
];

await worksheet.addRows([row]);
```

**支持的样式属性 / Supported Style Properties:**

| 属性 / Property | 类型 / Type    | 说明 / Description         |
| --------------- | -------------- | -------------------------- |
| `value`         | string/number  | 单元格值 / Cell value      |
| `bold`          | boolean        | 粗体 / Bold                |
| `italic`        | boolean        | 斜体 / Italic              |
| `underline`     | boolean        | 下划线 / Underline         |
| `strikethrough` | boolean        | 删除线 / Strikethrough     |
| `fontSize`      | number         | 字号 / Font size           |
| `fontFamily`    | string         | 字体 / Font family         |
| `color`         | string         | 字体颜色 / Font color      |
| `fill`          | string         | 背景色 / Background color  |
| `align`         | string         | 水平对齐 left/center/right |
| `valign`        | string         | 垂直对齐 top/center/bottom |
| `border`        | boolean/object | 边框 / Border              |

### 3.5 富文本 HTML / Rich Text HTML

单元格的 `value` 支持 HTML 富文本，可以在一个单元格内混合多种样式。

The cell `value` supports HTML rich text, allowing mixed styles within a single cell.

**支持的 HTML 标签 / Supported HTML Tags:**

| 标签 / Tag        | 说明 / Description     |
| ----------------- | ---------------------- |
| `<b>`, `<strong>` | 粗体 / Bold            |
| `<i>`, `<em>`     | 斜体 / Italic          |
| `<u>`             | 下划线 / Underline     |
| `<s>`, `<strike>` | 删除线 / Strikethrough |
| `<br>`            | 换行 / Line break      |
| `<span>`, `<div>` | 容器（用于 style）     |
| `<td>`            | 表格单元格样式         |

**style 属性支持 / Supported style Properties:**

| CSS 属性 / Property | 示例 / Example               | 说明 / Description |
| ------------------- | ---------------------------- | ------------------ |
| `font-size`         | `font-size: 16pt`            | 字号（支持 pt/px） |
| `font-weight`       | `font-weight: bold`          | 粗体               |
| `font-style`        | `font-style: italic`         | 斜体               |
| `color`             | `color: #ff0000`             | 字体颜色           |
| `background-color`  | `background-color: #f0f0f0`  | 背景色             |
| `text-align`        | `text-align: center`         | 水平对齐           |
| `text-decoration`   | `text-decoration: underline` | 下划线/删除线      |
| `line-height`       | `line-height: 16.8pt`        | 行高               |

**富文本示例 / Rich Text Examples:**

```javascript
// 示例 1：基础 HTML 标签
{ value: "<b>粗体</b> <i>斜体</i> <u>下划线</u>", border: true }

// 示例 2：换行
{ value: "第一行<br>第二行<br>第三行", border: true }

// 示例 3：带 style 的 span
{
  value: '<span style="color: #ff0000; font-weight: bold;">红色粗体</span> 普通文字',
  border: true
}

// 示例 4：多种颜色混合
{
  value: '<span style="color: #ff0000;">红色</span>' +
         '<span style="color: #00ff00;">绿色</span>' +
         '<span style="color: #0000ff;">蓝色</span>',
  border: true
}

// 示例 5：带字号和背景色的 td 标签
{
  value: '<td style="font-size: 16pt; font-weight: bold; text-align: center; background-color: #e3f2d9;">标题</td>',
  border: true
}

// 示例 6：复杂富文本（多行 + 混合样式）
{
  value: '<div style="line-height: 16.8pt">项目名称：Excel导出插件</div>' +
         '<div style="line-height: 16.8pt">版本：<span style="font-weight: bold;">1.0.0</span></div>' +
         '<div style="line-height: 16.8pt">状态：<span style="color: #ff0000; font-weight: bold;">发布中</span></div>',
  border: true
}

// 示例 7：检查结果样式（实际业务场景）
{
  value: '<span style="font-weight: bold;">检查结果：</span>' +
         '共检查<span style="font-weight: bold;">10</span>项，' +
         '其中<span style="color: #ff0000; font-weight: bold;">3</span>项存疑',
  border: true,
  fill: "#fff9e3"  // 可同时设置单元格级别样式
}
```

### 3.5 执行导出 / Execute Export

```javascript
const result = await exporter.export();
console.log(result); // { success: true, rows: 100000 }
```

### 3.6 自动分页 / Auto Split Sheets

当数据量超过单个 Sheet 的最大行数时，会自动创建新的 Sheet。

When data exceeds the maximum rows per sheet, new sheets are created automatically.

```javascript
const exporter = new FulExcelExport({
  fileName: "大数据.xlsx",
  workSheetMaxRowsNum: 1000000, // 每 Sheet 最大行数（非必填，默认100万）
});

// 添加超过100万行的数据，会自动分成多个 Sheet
// Adding over 1M rows will auto-split into multiple sheets
```

### 3.7 多 Sheet 合并导出 / Multi-Sheet Export

将多个数据源合并到同一个 Excel 文件的不同 Sheet 中。

Merge multiple data sources into different sheets of the same Excel file.

```javascript
import FulExcelExport from "./ful-excel-export.esm.js";

async function exportMultipleSheets() {
  // 1. 创建导出器（只创建一次）
  const exporter = new FulExcelExport({
    fileName: "综合报表.xlsx",
    onProgress: (percent, stage, message, memory) => {
      console.log(`[${stage}] ${percent}% - ${message}`);
    },
  });

  // 2. 第一个 Sheet：员工数据
  const sheet1 = exporter.addWorkSheet({ name: "员工表" });
  await sheet1.addRows([
    [
      { value: "工号", bold: true, fill: "#4472C4", color: "#FFFFFF" },
      { value: "姓名", bold: true, fill: "#4472C4", color: "#FFFFFF" },
      { value: "部门", bold: true, fill: "#4472C4", color: "#FFFFFF" },
    ],
    ["E001", "张三", "技术部"],
    ["E002", "李四", "市场部"],
    ["E003", "王五", "财务部"],
  ]);

  // 3. 第二个 Sheet：部门数据
  const sheet2 = exporter.addWorkSheet({ name: "部门表" });
  await sheet2.addRows([
    [
      { value: "部门编号", bold: true, fill: "#70AD47", color: "#FFFFFF" },
      { value: "部门名称", bold: true, fill: "#70AD47", color: "#FFFFFF" },
      { value: "人数", bold: true, fill: "#70AD47", color: "#FFFFFF" },
    ],
    ["D001", "技术部", 50],
    ["D002", "市场部", 30],
    ["D003", "财务部", 10],
  ]);

  // 4. 第三个 Sheet：项目数据
  const sheet3 = exporter.addWorkSheet({ name: "项目表" });
  await sheet3.addRows([
    [
      { value: "项目编号", bold: true, fill: "#ED7D31", color: "#FFFFFF" },
      { value: "项目名称", bold: true, fill: "#ED7D31", color: "#FFFFFF" },
      { value: "状态", bold: true, fill: "#ED7D31", color: "#FFFFFF" },
    ],
    ["P001", "官网改版", "进行中"],
    ["P002", "移动APP开发", "已完成"],
    ["P003", "数据平台", "规划中"],
  ]);

  // 5. 执行导出（所有 Sheet 合并到一个文件）
  const result = await exporter.export();
  console.log(`导出完成！共 ${result.rows} 行，包含 3 个工作表`);
}
```

**要点说明 / Key Points:**

1. **单一 exporter 实例**：多个 Sheet 必须使用同一个 `FulExcelExport` 实例
2. **顺序调用 addWorkSheet**：每次调用 `addWorkSheet()` 创建新的工作表
3. **按顺序写入数据**：完成一个 Sheet 的数据写入后，再创建下一个 Sheet
4. **最后调用 export()**：所有 Sheet 数据写入完成后，调用 `export()` 生成文件

---

## 4. 完整示例 / Complete Example

```javascript
import FulExcelExport from "./ful-excel-export.esm.js";

async function exportData() {
  // 1. 创建导出器
  const exporter = new FulExcelExport({
    fileName: "销售报表.xlsx",
    onProgress: (info) => {
      console.log(`${info.percent}% - ${info.message}`);
    },
  });

  // 2. 添加工作表
  const worksheet = exporter.addWorkSheet({
    name: "销售数据",
    columnCount: 4,
  });

  // 3. 添加表头（带样式）
  await worksheet.addRows([
    [
      { value: "ID", bold: true, fill: "#4472C4", color: "#FFFFFF" },
      { value: "商品名称", bold: true, fill: "#4472C4", color: "#FFFFFF" },
      { value: "数量", bold: true, fill: "#4472C4", color: "#FFFFFF" },
      { value: "金额", bold: true, fill: "#4472C4", color: "#FFFFFF" },
    ],
  ]);

  // 4. 分页添加数据（适用于大数据量）
  const totalRows = 100000;
  const pageSize = 5000;
  const totalPages = Math.ceil(totalRows / pageSize);

  for (let page = 0; page < totalPages; page++) {
    const pageData = [];
    const start = page * pageSize;
    const end = Math.min(start + pageSize, totalRows);

    for (let i = start; i < end; i++) {
      pageData.push([
        i + 1,
        `商品${i + 1}`,
        Math.floor(Math.random() * 100),
        (Math.random() * 1000).toFixed(2),
      ]);
    }

    await worksheet.addRows(pageData);
  }

  // 5. 执行导出
  await exporter.export();
}

exportData();
```

---

## 5. Pro 专业版功能 / Pro Edition Features

> ⚠️ 以下功能仅在 Pro 专业版中可用
>
> ⚠️ The following features are only available in Pro Edition

### 5.1 合并单元格 / Merge Cells (Pro)

```javascript
// Pro 专用
const row = [
  {
    value: "合并区域标题",
    merge: { rowSpan: 2, colSpan: 3 }, // 跨2行3列
  },
  { value: "" }, // 被合并的单元格留空
  { value: "" },
];
```

### 5.2 自定义行高列宽 / Row Height & Column Width (Pro)

```javascript
// Pro 专用
const worksheet = exporter.addWorkSheet({
  name: "Sheet1",
  columns: [
    { width: 20 }, // 第1列宽度
    { width: 30 }, // 第2列宽度
    { width: 15 }, // 第3列宽度
  ],
});

// 设置行高
const row = [{ value: "高行", height: 40 }];
```

### 5.3 图片嵌入 / Image Embedding (Pro)

```javascript
// Pro 专用
const row = [
  {
    type: "image",
    value: "data:image/png;base64,iVBORw0KGgo...", // base64 图片
    width: 100,
    height: 80,
  },
];
```

### 5.4 超链接 / Hyperlinks (Pro)

```javascript
// Pro 专用
const row = [
  {
    value: "点击访问",
    link: "https://example.com",
  },
  {
    value: "发送邮件",
    link: "mailto:test@example.com",
  },
];
```

---

## 6. 功能对比 / Feature Comparison

如需以下高级功能，请联系获取 Pro 版：

For advanced features below, please contact us for Pro Edition:

| 功能 / Feature    | 开源版 / Open Source | Pro 版 / Pro |
| ----------------- | -------------------- | ------------ |
| 千万级数据导出    | ✅                   | ✅           |
| 单元格样式        | ✅                   | ✅           |
| 富文本 HTML       | ✅                   | ✅           |
| 合并单元格        | ❌                   | ✅           |
| 多 Sheet 自动分页 | ✅                   | ✅           |
| 自定义行高列宽    | ❌                   | ✅           |
| 图片嵌入          | ❌                   | ✅           |
| 超链接            | ❌                   | ✅           |
| 技术支持          | ❌                   | ✅           |

**联系方式 / Contact:** 634688344@qq.com

---

## 7. 重要事项 / Important Notes

### 7.1 关于内存占用与流式下载

本文档中提到的 **"内存仅占用 50MB"** 是建立在 **启用流式下载** 的基础上的。

#### 两种下载模式对比

| 模式 / Mode           | 内存占用 / Memory      | 说明 / Description                     |
| --------------------- | ---------------------- | -------------------------------------- |
| **Blob 模式（默认）** | 较高（与文件大小相关） | 文件完全生成后才触发下载，适合中小文件 |
| **流式下载**          | 低（约 50MB 稳定）     | 边生成边下载，适合大文件（千万级数据） |

#### 启用流式下载

```javascript
const exporter = new FulExcelExport({
  fileName: "大数据.xlsx",
  useStreamDownload: true, // 启用流式下载
  // streamMitmUrl: "https://your-domain.com/mitm.html", // 可选：自定义地址
});
```

#### ⚠️ 流式下载注意事项

1. **网络依赖**：默认的流式下载依赖外部 Service Worker 中转服务 (`jimmywarting.github.io`)。由于网络波动，可能导致导出失败或卡住。

2. **生产环境建议**：在生产环境中使用流式下载时，强烈建议在自己的服务器上部署 Service Worker 中转页面：

   ```javascript
   const exporter = new FulExcelExport({
     fileName: "export.xlsx",
     useStreamDownload: true,
     streamMitmUrl: "https://your-domain.com/mitm.html", // 部署在自己服务器
   });
   ```

3. **部署 mitm.html**：从 [StreamSaver.js](https://github.com/nicedoc/nicedoc-react/tree/master/public) 获取 `mitm.html` 和 `sw.js` 文件，部署到您的 HTTPS 服务器。

4. **HTTPS 要求**：Service Worker 仅在 HTTPS 环境下工作（localhost 除外）。

5. **推荐方案**：如果无法部署 Service Worker，建议保持默认的 Blob 模式，对于超大文件可考虑分批导出。

---

## 8. License

MIT License
