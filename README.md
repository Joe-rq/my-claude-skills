# My Claude Skills

个人自定义的 [Claude Skills](https://support.claude.com/en/articles/12512176-what-are-skills) 集合。

## Skills 列表

| Skill | 描述 |
|-------|------|
| **writing-assistant-skill** | 自媒体写作助手，帮助完成从选题到发布的完整流程 |
| **excalidraw-prompt-engineer** | AI 驱动的 Excalidraw 图表生成系统，支持 23+ 图表类型 |
| **decision-card** | 结构化决策路径生成器，带评分和推荐选项 |
| **primitive-ir-compiler** | 将自然语言转换为低熵的 Primitive IR 对象 |
| **source-tracing** | 信息溯源 skill |
| **medical-triage** | 智能医疗导诊助手，模拟分诊护士问诊流程 |
| **explain-code** | 使用视觉图表和类比解释代码，适合教学和文档编写 |
| **mermaid-ascii-renderer** | beautiful-mermaid 项目的 ASCII 渲染系统完整指南 |
| **text-to-json** | 将自然语言描述转换为复杂的分层 JSON 结构 |

## 使用方法

将对应 skill 目录复制到 Claude Desktop 的 skills 目录：

```bash
# macOS/Linux
cp -r <skill-name> ~/.claude/skills/

# Windows
xcopy <skill-name> %USERPROFILE%\.claude\skills\ /E /I
```

## License

MIT
