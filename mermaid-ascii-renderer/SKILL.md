---
name: mermaid-ascii-renderer
description: >-
  使用 beautiful-mermaid 项目将 Mermaid 图表渲染为 ASCII 艺术。提供解析、布局算法和渲染操作的完整实现指南，支持流程图、状态图、序列图、类图和 ER 图表。当用户请求涉及以下场景时使用：（1）beautiful-mermaid 的 ASCII 渲染系统实现；（2）在终端/控制台输出 Mermaid 图表；（3）理解 A* 路径查找、网格布局、边路由等算法；（4）添加新的图表类型支持；（5）调试 ASCII 渲染问题（节点重叠、边穿过、标签位置）；（6）修改渲染逻辑或布局策略。
metadata:
  author: joe
  version: 1.1.0
  title: mermaid-ascii-renderer
  description_zh: >-
    使用 beautiful-mermaid 项目将 Mermaid 图表渲染为 ASCII
    艺术。提供解析、布局算法和渲染操作的完整实现指南，支持流程图、状态图、序列图、类图和 ER 图表。当 Claude 需要理解
    beautiful-mermaid 的 ASCII 渲染系统或在项目中使用 ASCII 渲染功能时使用。
---

# Mermaid ASCII 渲染器技能

用于在 beautiful-mermaid 项目中理解或使用 ASCII/Unicode 渲染（流程图、状态图、序列图、类图、ER 图）。

## 范围与限制

### 支持的图表类型

| 图表类型 | 支持状态 | 说明 |
|---------|---------|------|
| **Flowchart** (流程图) | ✅ 完全支持 | graph TD/LR/BT/RL, 子图, 多种节点形状 |
| **State Diagram** (状态图) | ✅ 完全支持 | stateDiagram-v2, [*] 起始/结束状态 |
| **Sequence Diagram** (序列图) | ✅ 完全支持 | 参与者、消息、循环/条件块、注释 |
| **Class Diagram** (类图) | ✅ 完全支持 | 类定义、属性、方法、继承/组合/关联关系 |
| **ER Diagram** (实体关系图) | ✅ 完全支持 | 实体、属性、Crow's Foot 关系标记 |

### 不支持的图表类型

- Gantt (甘特图)
- Pie Chart (饼图)
- Mindmap (思维导图)
- Timeline (时间线)
- User Journey (用户旅程图)
- Git Graph (Git 图表)
- C4Context/C4Container (C4 模型)

### 版本兼容性

- **基于版本**: beautiful-mermaid v0.1.x
- **Node.js**: >= 18.0.0
- **Bun**: >= 1.0.0
- **TypeScript**: >= 5.0.0

### 输出限制与边界条件

| 场景 | ASCII 模式 | Unicode 模式 |
|------|-----------|-------------|
| **长标签** (>20字符) | 自动扩展节点宽度，可能影响布局 | 同左 |
| **密集图** (>20节点) | 渲染时间增加，建议增大 padding | 同左 |
| **复杂路由** | 可能产生锐角，可调整 heuristic | 同左 |
| **终端兼容性** | 最佳兼容性，所有终端支持 | 需 UTF-8 支持终端 |
| **子图嵌套** | 支持一层子图，多层嵌套可能重叠 | 同左 |

## 快速开始

### 安装

```bash
npm install beautiful-mermaid
# 或
bun add beautiful-mermaid
# 或
pnpm add beautiful-mermaid
```

### 基本使用

```typescript
import { renderMermaidAscii } from 'beautiful-mermaid'

const ascii = renderMermaidAscii(`graph LR\n  A --> B --> C`)
console.log(ascii)
```

**输出:**
```
┌───┐     ┌───┐     ┌───┐
│   │     │   │     │   │
│ A │────►│ B │────►│ C │
│   │     │   │     │   │
└───┘     └───┘     └───┘
```

### 不同环境的使用方式

#### Node.js / Bun

```typescript
import { renderMermaidAscii } from 'beautiful-mermaid'
// 或 CommonJS
const { renderMermaidAscii } = require('beautiful-mermaid')

const ascii = renderMermaidAscii(`graph LR\n  A --> B`)
console.log(ascii)
```

#### 浏览器 (CDN)

```html
<!-- 通过 unpkg -->
<script src="https://unpkg.com/beautiful-mermaid/dist/beautiful-mermaid.browser.global.js"></script>

<!-- 或 jsDelivr -->
<script src="https://cdn.jsdelivr.net/npm/beautiful-mermaid/dist/beautiful-mermaid.browser.global.js"></script>

<script>
  const { renderMermaidAscii } = beautifulMermaid
  const ascii = renderMermaidAscii('graph LR; A-->B')
  console.log(ascii)
</script>
```

#### 同步 vs 异步

```typescript
// ASCII 渲染 - 同步，立即返回结果
const ascii = renderMermaidAscii(`graph LR\n  A --> B`)
console.log(ascii)  // 直接输出字符串

// SVG 渲染 - 异步，返回 Promise
import { renderMermaid } from 'beautiful-mermaid'
const svg = await renderMermaid(`graph LR\n  A --> B`)
console.log(svg)  // SVG 字符串
```

## 核心 API

### renderMermaidAscii

```typescript
function renderMermaidAscii(
  diagram: string,
  options?: AsciiRenderOptions
): string
```

将 Mermaid 图表渲染为 ASCII/Unicode 字符串（同步执行）。

**参数说明：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `diagram` | `string` | ✅ | Mermaid 图表语法字符串 |
| `options` | `AsciiRenderOptions` | ❌ | 渲染选项配置 |

**AsciiRenderOptions 详细说明：**

| 选项 | 类型 | 默认值 | 取值范围 | 影响说明 |
|------|------|--------|----------|----------|
| `useAscii` | `boolean` | `false` | `true` / `false` | 字符集选择：`true` 使用纯 ASCII (`+---+`)，`false` 使用 Unicode 框线 (`┌───┐`) |
| `paddingX` | `number` | `5` | `0` - `20` | 节点间水平间距（字符数）。值越大，图表越宽，边路由越清晰 |
| `paddingY` | `number` | `5` | `0` - `20` | 节点间垂直间距（字符数）。值越大，图表越高，适合复杂布局 |
| `boxBorderPadding` | `number` | `1` | `0` - `3` | 节点框内边距（字符数）。影响标签与边框的距离 |

**使用示例：**

```typescript
import { renderMermaidAscii } from 'beautiful-mermaid'

// 基础用法 - Unicode 模式（推荐）
const unicode = renderMermaidAscii(`graph LR\n  A --> B`)

// ASCII 模式 - 最大兼容性
const ascii = renderMermaidAscii(`graph LR\n  A --> B`, {
  useAscii: true
})

// 调整间距 - 适合复杂图表
const spaced = renderMermaidAscii(`graph TD\n  A --> B --> C`, {
  useAscii: false,
  paddingX: 8,        // 增大水平间距
  paddingY: 6,        // 增大垂直间距
  boxBorderPadding: 1 // 默认内边距
})
```

## SVG vs ASCII 渲染选择指南

beautiful-mermaid 提供两种渲染模式，适用于不同场景：

### API 对比

| 特性 | `renderMermaid()` (SVG) | `renderMermaidAscii()` (ASCII) |
|------|------------------------|-------------------------------|
| **输出格式** | SVG 矢量图形 | 纯文本字符串 |
| **执行方式** | 异步 (`Promise<string>`) | 同步 (`string`) |
| **主题支持** | ✅ 完整主题系统 (15+ 内置主题) | ❌ 无主题，仅字符样式 |
| **交互性** | ✅ 可交互 (点击、悬停) | ❌ 静态文本 |
| **可缩放性** | ✅ 无限缩放不失真 | ❌ 固定字符尺寸 |
| **文件大小** | 较大 (含矢量路径) | 极小 (纯文本) |
| **终端显示** | ❌ 需图形界面 | ✅ 完美支持 |

### 选择建议

**选择 SVG (`renderMermaid`) 当：**
- 在 Web 应用/浏览器中显示
- 需要丰富的主题和配色
- 需要交互功能（点击节点等）
- 对输出文件大小不敏感
- 需要打印或导出为图片

**选择 ASCII (`renderMermaidAscii`) 当：**
- 在终端/控制台/CLI 工具中显示
- 需要纯文本输出（日志、文档、邮件）
- 追求零依赖、轻量级
- 需要快速同步渲染
- 在代码注释或 Markdown 中嵌入

### 混合使用示例

```typescript
import { renderMermaid, renderMermaidAscii } from 'beautiful-mermaid'

// 根据环境自动选择
async function renderDiagram(diagram: string, format: 'svg' | 'ascii') {
  if (format === 'svg') {
    // 异步获取 SVG
    return await renderMermaid(diagram, {
      bg: '#1a1b26',
      fg: '#a9b1d6'
    })
  } else {
    // 同步获取 ASCII
    return renderMermaidAscii(diagram, {
      useAscii: false,
      paddingX: 5
    })
  }
}
```

## 主题与配色系统（SVG 渲染）

> 虽然 ASCII 渲染不使用主题系统，但 beautiful-mermaid 的 SVG 渲染支持强大的主题功能。了解主题系统有助于理解整个项目的架构。

### Two-Color Foundation（双色基础）

SVG 渲染的核心设计：仅需两种颜色即可生成完整图表主题。

```typescript
import { renderMermaid } from 'beautiful-mermaid'

// Mono Mode: 仅需 bg 和 fg
const svg = await renderMermaid(diagram, {
  bg: '#1a1b26',  // 背景色
  fg: '#a9b1d6',  // 前景色
})
```

**自动推导的元素颜色**:

| 元素 | 推导方式 | 示例值 |
|------|---------|--------|
| 文本 | 100% fg | `#a9b1d6` |
| 次要文本 | 60% fg + 40% bg | `#7a819d` |
| 边标签 | 40% fg + 60% bg | `#5c6270` |
| 连接线 | 30% fg + 70% bg | `#4e5462` |
| 箭头 | 50% fg + 50% bg | `#8490a3` |
| 节点填充 | 3% fg + 97% bg | `#1b1c27` |
| 节点边框 | 20% fg + 80% bg | `#3a3f4f` |

### Enriched Mode（丰富模式）

如需更多控制，可指定额外的丰富颜色：

```typescript
const svg = await renderMermaid(diagram, {
  bg: '#1a1b26',
  fg: '#a9b1d6',
  line: '#3d59a1',     // 边/连接线颜色
  accent: '#7aa2f7',   // 箭头、高亮
  muted: '#565f89',    // 次要文本、标签
  surface: '#292e42',  // 节点填充色调
  border: '#3d59a1',   // 节点边框
})
```

### CSS 自定义属性

所有颜色通过 CSS 自定义属性 (--bg, --fg 等) 应用到 SVG，支持**实时主题切换**而无需重新渲染：

```javascript
// 浏览器中切换主题
const svgElement = document.querySelector('svg')
svgElement.style.setProperty('--bg', '#282a36')
svgElement.style.setProperty('--fg', '#f8f8f2')
// 图表立即更新，无需重新调用 renderMermaid
```

### 内置主题

```typescript
import { renderMermaid, THEMES } from 'beautiful-mermaid'

// 使用内置主题
const svg = await renderMermaid(diagram, THEMES['tokyo-night'])

// 可用主题: zinc-light, zinc-dark, tokyo-night, catppuccin-mocha, 
//           nord, dracula, github-light, github-dark, solarized-light, 
//           solarized-dark, one-dark 等 15+ 主题
```

## 多图类型示例

### Sequence Diagram (序列图)

```typescript
const sequence = renderMermaidAscii(`
sequenceDiagram
  Alice->>Bob: Hello Bob!
  Bob-->>Alice: Hi Alice!
  Alice->>Bob: How are you?
`)
```

**输出:**
```
┌──────┐      ┌────┐
│Alice │      │Bob │
└──┬───┘      └─┬──┘
   │            │
   │ Hello Bob! │
   │───────────>│
   │            │
   │ Hi Alice!  │
   │<───────────│
   │            │
   │ How are you│
   │───────────>│
```

### Class Diagram (类图)

```typescript
const classDiagram = renderMermaidAscii(`
classDiagram
  Animal <|-- Duck
  Animal: +int age
  Animal: +String gender
  Animal: +isMammal()
  Duck: +String beakColor
  Duck: +swim()
`)
```

**输出:**
```
┌─────────────────┐
│     Animal      │
├─────────────────┤
│+ int age        │
│+ String gender  │
├─────────────────┤
│+ isMammal()     │
└────────┬────────┘
         │
         △
         │
┌────────┴────────┐
│      Duck       │
├─────────────────┤
│+ String beakColo│
├─────────────────┤
│+ swim()         │
└─────────────────┘
```

### ER Diagram (实体关系图)

```typescript
const erDiagram = renderMermaidAscii(`
erDiagram
  CUSTOMER ||--o{ ORDER : places
  ORDER ||--|{ LINE_ITEM : contains
`)
```

**输出:**
```
┌──────────┐           ┌───────┐
│ CUSTOMER │           │ ORDER │
├──────────┤           ├───────┤
│          │           │       │
└────┬─────┘           └───┬───┘
     │                     │
     │  places             │
     │  ════════o{         │
     └─────────────────────┘
```

### 带方向的 Flowchart

```typescript
// 从上到下 (TD)
const td = renderMermaidAscii(`graph TD\n  A[Start] --> B{Decision}\n  B -->|Yes| C[Process]\n  B -->|No| D[End]`)

// 从左到右 (LR)
const lr = renderMermaidAscii(`graph LR\n  A[Input] --> B[Process] --> C[Output]`)

// 从下到上 (BT)
const bt = renderMermaidAscii(`graph BT\n  A[Result] --> B[Step 2] --> C[Step 1]`)
```

## 项目结构参考

**beautiful-mermaid 源码结构** (简化版):
```
src/
  ascii/              # ASCII 渲染引擎
    index.ts          # ASCII 渲染 API 入口
    types.ts          # 类型定义
    canvas.ts         # 画布操作
    draw.ts          # 绘图原语
    grid.ts          # 网格布局
    pathfinder.ts    # A* 路径查找
    edge-routing.ts   # 边路由
    sequence.ts      # 序列图实现
    class-diagram.ts # 类图实现
    er-diagram.ts   # ER 图实现
    converter.ts     # 转换器
  index.ts           # 主 API 入口 (导出 renderMermaid, renderMermaidAscii)
  renderer.ts       # SVG 渲染
  parser.ts         # Mermaid 语法解析
  layout.ts         # 布局引擎
  theme.ts          # 主题系统
```

**完整细节请参考**: `references/core-systems.md`（低层系统）、`references/diagrams.md`（图表实现）

## 何时阅读参考文档

- 图表实现细节与布局算法：`references/diagrams.md`
- 低层渲染系统（类型/画布/绘图/路由）：`references/core-systems.md`
- Unicode/ASCII 字符表：`references/characters.md`
- 故障排查与性能建议：`references/troubleshooting.md`
- 外部链接与资料：`references/resources.md`
