# My Claude Skills

个人自定义的 [Claude Skills](https://support.claude.com/en/articles/12512176-what-are-skills) 集合。

## Skills 列表

| Skill | 描述 |
|-------|------|
| **writing-assistant-skill** | 自媒体写作助手，帮助完成从选题到发布的完整流程 |
| **excalidraw-prompt-engineer** | AI 驱动的 Excalidraw 图表生成系统，支持 23+ 图表类型 |
| **decision-card** | 结构化决策路径生成器，带评分和推荐选项 |
| **primitive-ir-compiler** | IR 编译器相关 skill |
| **source-tracing** | 信息溯源 skill |

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
