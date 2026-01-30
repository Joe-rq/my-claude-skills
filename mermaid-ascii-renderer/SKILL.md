---
name: mermaid-ascii-renderer
description: >-
  使用 beautiful-mermaid 项目将 Mermaid 图表渲染为 ASCII 艺术。提供解析、布局算法和渲染操作的完整实现指南，支持流程图、状态图、序列图、类图和 ER 图表。当用户请求涉及以下场景时使用：（1）beautiful-mermaid 的 ASCII 渲染系统实现；（2）在终端/控制台输出 Mermaid 图表；（3）理解 A* 路径查找、网格布局、边路由等算法；（4）添加新的图表类型支持；（5）调试 ASCII 渲染问题（节点重叠、边穿过、标签位置）；（6）修改渲染逻辑或布局策略。
metadata:
  author: joe
  version: 1.0.0
  title: mermaid-ascii-renderer
  description_zh: >-
    使用 beautiful-mermaid 项目将 Mermaid 图表渲染为 ASCII
    艺术。提供解析、布局算法和渲染操作的完整实现指南，支持流程图、状态图、序列图、类图和 ER 图表。当 Claude 需要理解
    beautiful-mermaid 的 ASCII 渲染系统或在项目中使用 ASCII 渲染功能时使用。
---

# Mermaid ASCII 渲染器技能

用于在 beautiful-mermaid 项目中理解或使用 ASCII/Unicode 渲染（流程图、状态图、序列图、类图、ER 图）。

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

## 核心 API

```typescript
import { renderMermaidAscii } from 'beautiful-mermaid'

const ascii = renderMermaidAscii(diagram, {
  useAscii: false,      // true: ASCII, false: Unicode
  paddingX: 5,
  paddingY: 5,
  boxBorderPadding: 1,
})
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
