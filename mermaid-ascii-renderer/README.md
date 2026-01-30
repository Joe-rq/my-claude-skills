# Claude Skills for beautiful-mermaid

此目录包含用于 beautiful-mermaid 项目的 Claude 技能。

## 可用技能

### mermaid-ascii-renderer

提供 beautiful-mermaid 项目 ASCII 渲染系统的完整指南，包括：

- **支持的图表类型**：流程图/状态图、序列图、类图、ER 图
- **核心工具系统**：画布、绘图、网格布局、A* 路径查找、边路由
- **渲染选项**：ASCII/Unicode 双模式、自定义间距
- **算法详解**：A* 启发式、分层布局、网格坐标系统
- **扩展指南**：添加新图表类型、优化工具函数
- **故障排查**：节点重叠、边穿过、标签位置错误

## 使用方式

当需要理解或使用 beautiful-mermaid 的 ASCII 渲染功能时，系统会自动加载此技能。

### 示例场景

1. **理解项目架构**
   - 查看项目结构（src/ascii/ 目录）
   - 了解各图表类型的渲染策略
   - 研究核心工具函数（draw.ts、canvas.ts、grid.ts）

2. **修改渲染逻辑**
   - 参考布局算法（A* 路径查找、分层排序）
   - 调整绘图操作（盒子、线条、箭头）
   - 优化边路由策略

3. **添加新图表类型**
   - 按照扩展指南创建解析器、布局和渲染器
   - 更新 src/ascii/index.ts 中的图表类型检测

4. **调试渲染问题**
   - 查看常见问题解决章节
   - 参考性能优化建议

## 技能目录结构

```
.claude/
└── skills/
    └── mermaid-ascii-renderer/
        ├── SKILL.md          # 技能主体文档
        ├── scripts/           # 可选：自动化脚本
        ├── references/        # 可选：参考文档
        └── assets/           # 可选：资源文件
```

## 技能加载方式

Claude 系统会自动检测并加载 `.claude/skills/` 目录下的技能，当请求涉及以下内容时触发：

- beautiful-mermaid 的 ASCII 渲染实现
- Mermaid 图表的布局和渲染算法
- Unicode/ASCII 字符集的使用
- 流程图、序列图、类图、ER 图的渲染策略

---

## 致谢与参考

本技能基于 [lukilabs/beautiful-mermaid](https://github.com/lukilabs/beautiful-mermaid) 项目创建。

**beautiful-mermaid** 是一个将 Mermaid 图表转换为精美 ASCII/Unicode 渲染的开源项目。感谢该项目作者提供了优秀的实现，使我们能够深入理解其内部架构和算法。

---

**创建时间**：2026-01-30
**项目**：beautiful-mermaid (lukilabs/beautiful-mermaid)
