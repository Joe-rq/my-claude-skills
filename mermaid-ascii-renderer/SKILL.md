---
name: mermaid-ascii-renderer
description: >-
  使用 beautiful-mermaid 项目将 Mermaid 图表渲染为 ASCII
  艺术。提供解析、布局算法和渲染操作的完整实现指南，支持流程图、状态图、序列图、类图和 ER 图表。当 Claude 需要理解
  beautiful-mermaid 的 ASCII 渲染系统或在项目中使用 ASCII 渲染功能时使用。
metadata:
  author: joe
  version: 1.0.0
  title: mermaid-ascii-rendere
  description_zh: >-
    使用 beautiful-mermaid 项目将 Mermaid 图表渲染为 ASCII
    艺术。提供解析、布局算法和渲染操作的完整实现指南，支持流程图、状态图、序列图、类图和 ER 图表。当 Claude 需要理解
    beautiful-mermaid 的 ASCII 渲染系统或在项目中使用 ASCII 渲染功能时使用。
---

# Mermaid ASCII 渲染器技能

用于在 beautiful-mermaid 项目中理解或使用 ASCII/Unicode 渲染（流程图、状态图、序列图、类图、ER 图）。

## 快速开始

```bash
bun run index.ts

import { renderMermaidAscii } from 'beautiful-mermaid'

const ascii = renderMermaidAscii(`graph LR\n  A --> B --> C`)
console.log(ascii)
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

## 高层结构

```
src/ascii/
  index.ts        API 入口
  types.ts        类型
  canvas.ts       画布
  draw.ts         绘图
  grid.ts         网格布局
  pathfinder.ts   A* 路由
  edge-routing.ts 边路由
  sequence.ts     序列图
  class-diagram.ts类图
  er-diagram.ts   ER 图
```

## 何时阅读参考文档

- 图表实现细节与布局算法：`references/diagrams.md`
- 低层渲染系统（类型/画布/绘图/路由）：`references/core-systems.md`
- Unicode/ASCII 字符表：`references/characters.md`
- 故障排查与性能建议：`references/troubleshooting.md`
- 外部链接与资料：`references/resources.md`
