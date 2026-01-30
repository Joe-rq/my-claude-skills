# My Claude Skills

个人自定义的 [Claude Skills](https://support.claude.com/en/articles/12512176-what-are-skills) 集合，用于扩展 Claude 在特定领域的专业能力。

## Skills 列表

### 内容创作

| Skill | 描述 |
|-------|------|
| **writing-assistant-skill** | 自媒体写作助手，帮助完成从选题到发布的完整流程 |
| **excalidraw-prompt-engineer** | AI 驱动的 Excalidraw 图表生成系统，支持 23+ 图表类型 |

### 思维与决策

| Skill | 描述 |
|-------|------|
| **decision-card** | 结构化决策路径生成器，带评分和推荐选项 |
| **primitive-ir-compiler** | 将自然语言转换为低熵的 Primitive IR 对象 |

### 开发工具

| Skill | 描述 | 亮点 |
|-------|------|------|
| **mermaid-ascii-renderer** | beautiful-mermaid ASCII/Unicode 渲染系统完整指南 | ✅ 支持 5 种图表类型<br>✅ 详细的 API 文档<br>✅ 故障排查决策树<br>✅ 扩展开发指南 |
| **explain-code** | 使用视觉图表和类比解释代码，适合教学和文档编写 |
| **text-to-json** | 将自然语言描述转换为复杂的分层 JSON 结构 |

### 专业领域

| Skill | 描述 |
|-------|------|
| **medical-triage** | 智能医疗导诊助手，模拟分诊护士问诊流程 |
| **source-tracing** | 信息溯源 skill |

## 快速开始

### 安装 Skills

将对应 skill 目录复制到 Claude Desktop 的 skills 目录：

```bash
# macOS/Linux
cp -r <skill-name> ~/.claude/skills/

# Windows
xcopy <skill-name> %USERPROFILE%\.claude\skills\ /E /I
```

### 验证安装

重启 Claude Desktop 后，在对话中触发 skill：

- 测试 `mermaid-ascii-renderer`: "怎么用 beautiful-mermaid 生成 ASCII 图表？"
- 测试 `writing-assistant`: "帮我规划一篇关于 AI 的文章"

## 项目结构

```
my-claude-skills/
├── <skill-name>/              # 每个 skill 独立目录
│   ├── SKILL.md              # 技能主体文档（必需）
│   ├── README.md             # 目录导航（可选）
│   ├── references/           # 详细参考文档（可选）
│   │   ├── core-systems.md
│   │   ├── diagrams.md
│   │   └── ...
│   └── assets/               # 资源文件（可选）
├── README.md                 # 本文件
└── .gitignore               # Git 忽略配置
```

## Skill 开发规范

### 文档结构

每个 skill 应包含清晰的渐进式披露结构：

1. **SKILL.md** - 核心入口（<5k 字）
   - 快速开始
   - 核心 API
   - 何时阅读参考文档

2. **references/** - 详细参考（按需加载）
   - 实现细节
   - 故障排查
   - 扩展指南

### 命名规范

- Skill 目录名：kebab-case（如 `mermaid-ascii-renderer`）
- Frontmatter name：与目录名一致
- 版本号：语义化版本（如 `1.0.0`）

## 最近更新

| 日期 | Skill | 更新内容 |
|------|-------|---------|
| 2026-01-30 | mermaid-ascii-renderer | 全面优化：范围与限制、API 细节、多图示例、故障决策树、扩展指南 |
| 2026-01-30 | 全局 | 添加 .gitignore，忽略依赖和构建文件 |

## 贡献指南

欢迎提交改进：

1. Fork 本仓库
2. 创建 feature 分支
3. 提交更改（遵循现有 commit 风格）
4. 发起 Pull Request

### Commit 风格

```
<类型>: <描述>

docs: 更新文档
feat: 新增功能
fix: 修复问题
chore: 维护任务
```

## 相关资源

- [Claude Skills 官方文档](https://support.claude.com/en/articles/12512176-what-are-skills)
- [beautiful-mermaid](https://github.com/lukilabs/beautiful-mermaid) - mermaid-ascii-renderer 基于的开源项目

## License

MIT

---

**维护者**: Joe  
**创建时间**: 2024-12  
**最后更新**: 2026-01-30
