# 用技能（Skills）扩展 Claude 的能力

> 在 Claude Code 中创建、管理和共享技能，从而扩展 Claude 的能力，包括自定义斜杠命令。

技能让 Claude 能做更多事情。创建一个包含指令的 `SKILL.md` 文件，Claude 就会把它加入自己的工具箱。Claude 会在相关时使用这些技能，你也可以直接通过 `/skill-name` 来调用它们。

<Note>
  对于 `/help` 和 `/compact` 等内置命令，请参阅[交互模式](/en/interactive-mode#built-in-commands)。

  **自定义斜杠命令已经合并到技能中。** `.claude/commands/review.md` 文件和 `.claude/skills/review/SKILL.md` 技能都能创建 `/review`，工作方式也相同。你现有的 `.claude/commands/` 文件会继续正常工作。技能增加了一些可选功能：用于存放辅助文件的目录、用于[控制由谁调用技能](#control-who-invokes-a-skill)的前置元数据（frontmatter，**一种在文件开头用特殊标记包裹的元数据格式，通常用 `---` 分隔**），以及 Claude 在相关时自动加载技能的能力。
</Note>

Claude Code 的技能遵循 [Agent Skills](https://agentskills.io) 开放标准，该标准适用于多种 AI 工具。Claude Code 在此基础上增加了额外功能，如[调用控制](#control-who-invokes-a-skill)、[子智能体执行](#run-skills-in-a-subagent)和[动态上下文注入](#inject-dynamic-context)。

## 快速开始

### 创建你的第一个技能

这个例子会创建一个技能，教 Claude 用视觉图表和类比来解释代码。因为它使用默认的前置元数据，所以当你问某些东西怎么工作时，Claude 可以自动加载它，你也可以直接用 `/explain-code` 调用它。

<Steps>
  <Step title="创建技能目录">
    在你的个人技能文件夹中为这个技能创建一个目录。个人技能在你的所有项目中都可用。

    ```bash  theme={null}
    mkdir -p ~/.claude/skills/explain-code
    ```
  </Step>

  <Step title="编写 SKILL.md">
    每个技能都需要一个 `SKILL.md` 文件，包含两个部分：YAML 前置元数据（在 `---` 标记之间）告诉 Claude 何时使用该技能，以及 Markdown 内容包含技能被调用时 Claude 要遵循的指令。`name` 字段会变成 `/斜杠命令`，`description` 帮助 Claude 决定何时自动加载它。

    创建 `~/.claude/skills/explain-code/SKILL.md`：

    ```yaml  theme={null}
    ---
    name: explain-code
    description: Explains code with visual diagrams and analogies. Use when explaining how code works, teaching about a codebase, or when the user asks "how does this work?"
    ---

    When explaining code, always include:

    1. **Start with an analogy**: Compare the code to something from everyday life
    2. **Draw a diagram**: Use ASCII art to show the flow, structure, or relationships
    3. **Walk through the code**: Explain step-by-step what happens
    4. **Highlight a gotcha**: What's a common mistake or misconception?

    Keep explanations conversational. For complex concepts, use multiple analogies.
    ```
  </Step>

  <Step title="测试技能">
    你可以用两种方式测试：

    **让 Claude 自动调用它**，只要问一些与描述匹配的问题：

    ```
    How does this code work?
    ```

    **或者直接用技能名调用它**：

    ```
    /explain-code src/auth/login.ts
    ```

    无论哪种方式，Claude 都应该在解释中包含类比和 ASCII 图表。
  </Step>
</Steps>

### 技能存放在哪里

你把技能存放在哪里决定了谁能使用它：

| 位置   | 路径                                             | 适用于                       |
| :--------- | :----------------------------------------------- | :--------------------------- |
| 企业级 | 请参阅[托管设置](/en/iam#managed-settings) | 组织内的所有用户              |
| 个人级   | `~/.claude/skills/<skill-name>/SKILL.md`         | 你的所有项目                 |
| 项目级    | `.claude/skills/<skill-name>/SKILL.md`           | 仅此项目                     |
| 插件     | `<plugin>/skills/<skill-name>/SKILL.md`          | 插件启用的地方              |

当不同层级的技能使用相同名称时，优先级高的位置胜出：企业级 > 个人级 > 项目级。插件技能使用 `plugin-name:skill-name` 命名空间（**命名空间是一种用于避免命名冲突的机制，通过在名称前加上前缀来区分不同的来源或模块**），所以不会与其他层级冲突。如果你在 `.claude/commands/` 中有文件，它们的工作方式相同，但如果技能和命令共享同一个名称，技能优先。

#### 从嵌套目录自动发现

当你在子目录中处理文件时，Claude Code 会自动从嵌套的 `.claude/skills/` 目录中发现技能。例如，如果你在编辑 `packages/frontend/` 中的文件，Claude Code 也会在 `packages/frontend/.claude/skills/` 中查找技能。这支持 monorepo（**单体仓库：一种将多个相关项目或包放在同一个代码仓库中的管理方式**）设置，其中各个包可以有自己的技能。

每个技能是一个以 `SKILL.md` 为入口点的目录：

```
my-skill/
├── SKILL.md           # 主指令（必需）
├── template.md        # Claude 需要填写的模板
├── examples/
│   └── sample.md      # 示例输出，展示预期格式
└── scripts/
    └── validate.sh    # Claude 可以执行的脚本
```

`SKILL.md` 包含主指令，是必需的。其他文件是可选的，让你能构建更强大的技能：供 Claude 填写的模板、展示预期格式的示例输出、Claude 可以执行的脚本，或详细的参考文档。在 `SKILL.md` 中引用这些文件，这样 Claude 就知道它们包含什么内容以及何时加载它们。更多细节请参阅[添加辅助文件](#add-supporting-files)。

<Note>
  `.claude/commands/` 中的文件仍然有效，支持相同的[前置元数据](#frontmatter-reference)。建议使用技能，因为它们支持额外的功能，比如辅助文件。
</Note>

## 配置技能

技能通过 `SKILL.md` 顶部的 YAML 前置元数据和随后的 Markdown 内容进行配置。

### 技能内容的类型

技能文件可以包含任何指令，但思考如何调用它们有助于决定包含什么内容：

**参考内容（Reference content）**添加 Claude 应用于当前工作的知识。约定、模式、风格指南、领域知识。这种内容是内联运行的，所以 Claude 可以在对话上下文旁边使用它。

```yaml  theme={null}
---
name: api-conventions
description: API design patterns for this codebase
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
- Include request validation
```

**任务内容（Task content）**为特定操作给 Claude 逐步指令，比如部署、提交或代码生成。这些通常是你想直接用 `/skill-name` 调用的操作，而不是让 Claude 决定何时运行它们。添加 `disable-model-invocation: true` 来防止 Claude 自动触发它。

```yaml  theme={null}
---
name: deploy
description: Deploy the application to production
context: fork
disable-model-invocation: true
---

Deploy the application:
1. Run the test suite
2. Build the application
3. Push to the deployment target
```

你的 `SKILL.md` 可以包含任何内容，但思考你希望如何调用技能（由你、由 Claude，或两者）、以及希望它在哪里运行（内联或在子智能体中），有助于决定包含什么。对于复杂的技能，你也可以[添加辅助文件](#add-supporting-files)以保持主技能的聚焦。

### 前置元数据参考

除了 Markdown 内容，你还可以使用 `SKILL.md` 文件顶部 `---` 标记之间的 YAML 前置元数据字段来配置技能行为：

```yaml  theme={null}
---
name: my-skill
description: What this skill does
disable-model-invocation: true
allowed-tools: Read, Grep
---

Your skill instructions here...
```

所有字段都是可选的。只有 `description` 是推荐的，这样 Claude 就知道何时使用该技能。

| 字段                      | 是否必需    | 描述                                                                                                                                           |
| :------------------------- | :---------- | :---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                     | 否          | 技能的显示名称。如果省略，使用目录名。仅限小写字母、数字和连字符（最多 64 个字符）。                    |
| `description`              | 推荐       | 技能做什么以及何时使用它。Claude 使用此信息来决定何时应用该技能。如果省略，使用 Markdown 内容的第一段。 |
| `argument-hint`            | 否          | 在自动完成时显示的提示，指示预期的参数。例如：`[issue-number]` 或 `[filename] [format]`。                                    |
| `disable-model-invocation` | 否          | 设置为 `true` 以防止 Claude 自动加载此技能。用于你想用 `/name` 手动触发的工作流。默认值：`false`。 |
| `user-invocable`           | 否          | 设置为 `false` 以从 `/` 菜单中隐藏。用于用户不应该直接调用的后台知识。默认值：`true`。                              |
| `allowed-tools`            | 否          | 当此技能处于活动状态时，Claude 无需请求权限即可使用的工具。                                                                             |
| `model`                    | 否          | 当此技能处于活动状态时使用的模型。                                                                                                               |
| `context`                  | 否          | 设置为 `fork` 以在分叉的子智能体上下文中运行。                                                                                                    |
| `agent`                    | 否          | 当设置了 `context: fork` 时使用的子智能体类型。                                                                                               |
| `hooks`                    | 否          | 作用域为此技能生命周期的钩子（**hook：在特定事件发生时自动执行的回调函数**）。有关配置格式，请参阅[钩子](/en/hooks)。                                                              |

#### 可用的字符串替换

技能支持在技能内容中对动态值进行字符串替换：

| 变量               | 描述                                                                                                                                  |
| :--------------------- | :------------------------------------------------------------------------------------------------------------------------------------------- |
| `$ARGUMENTS`           | 调用技能时传递的所有参数。如果 `$ARGUMENTS` 不存在于内容中，参数会作为 `ARGUMENTS: <value>` 附加在后面。 |
| `$ARGUMENTS[N]`        | 通过从 0 开始的索引访问特定参数，例如 `$ARGUMENTS[0]` 表示第一个参数。                                                 |
| `$N`                   | `$ARGUMENTS[N]` 的简写，例如 `$0` 表示第一个参数，`$1` 表示第二个参数。                                                   |
| `${CLAUDE_SESSION_ID}` | 当前的会话 ID。适用于日志记录、创建特定于会话的文件，或将技能输出与会话关联。                      |

**使用替换的示例：**

```yaml  theme={null}
---
name: session-logger
description: Log activity for this session
---

Log the following to logs/${CLAUDE_SESSION_ID}.log:

$ARGUMENTS
```

### 添加辅助文件

技能可以在其目录中包含多个文件。这使得 `SKILL.md` 专注于基本内容，同时让 Claude 仅在需要时访问详细的参考资料。大型参考文档、API 规范或示例集合不需要在每次技能运行时都加载到上下文中。

```
my-skill/
├── SKILL.md （必需 - 概述和导航）
├── reference.md （详细的 API 文档 - 需要时加载）
├── examples.md （使用示例 - 需要时加载）
└── scripts/
    └── helper.py （实用脚本 - 被执行，不被加载）
```

在 `SKILL.md` 中引用辅助文件，这样 Claude 就知道每个文件包含什么内容以及何时加载它：

```markdown  theme={null}
## Additional resources

- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)
```

<Tip>保持 `SKILL.md` 在 500 行以下。将详细的参考资料移动到单独的文件中。</Tip>

### 控制由谁调用技能

默认情况下，你和 Claude 都可以调用任何技能。你可以输入 `/skill-name` 直接调用它，Claude 也可以在相关时自动加载它。两个前置元数据字段允许你限制这种权限：

* **`disable-model-invocation: true`**：只有你可以调用该技能。将其用于有副作用（side effect，**指函数执行时对程序状态产生可观察的影响，如修改外部数据**）或你想控制时机的操作流程，比如 `/commit`、`/deploy` 或 `/send-slack-message`。你不想让 Claude 因为你的代码看起来准备好了就决定部署。

* **`user-invocable: false`**：只有 Claude 可以调用该技能。将其用于作为命令不可操作的背景知识。`legacy-system-context` 技能解释旧系统是如何工作的。Claude 在相关时应该知道这一点，但 `/legacy-system-context` 不是用户可以采取的有意义的操作。

这个例子创建了一个只有你能触发的部署技能。`disable-model-invocation: true` 字段防止 Claude 自动运行它：

```yaml  theme={null}
---
name: deploy
description: Deploy the application to production
disable-model-invocation: true
---

Deploy $ARGUMENTS to production:

1. Run the test suite
2. Build the application
3. Push to the deployment target
4. Verify the deployment succeeded
```

这两个字段如何影响调用和上下文加载：

| 前置元数据                      | 你可以调用 | Claude 可以调用 | 何时加载到上下文                                     |
| :------------------------------- | :------------- | :---------------- | :----------------------------------------------------------- |
| （默认）                        | 是            | 是               | 描述始终在上下文中，完整技能在调用时加载 |
| `disable-model-invocation: true` | 是            | 否                | 描述不在上下文中，完整技能在你调用时加载 |
| `user-invocable: false`          | 否             | 是               | 描述始终在上下文中，完整技能在调用时加载 |

<Note>
  在常规会话中，技能描述会加载到上下文中，这样 Claude 就知道有什么可用，但只有调用时才会加载完整的技能内容。[预加载技能的子智能体](/en/sub-agents#preload-skills-into-subagents)的工作方式不同：完整的技能内容在启动时就被注入。
</Note>

### 限制工具访问

使用 `allowed-tools` 字段来限制当技能处于活动状态时 Claude 可以使用哪些工具。这个技能创建了一个只读模式，Claude 可以探索文件但不能修改它们：

```yaml  theme={null}
---
name: safe-reader
description: Read files without making changes
allowed-tools: Read, Grep, Glob
---
```

### 向技能传递参数

你和 Claude 都可以在调用技能时传递参数。参数通过 `$ARGUMENTS` 占位符可用。

这个技能通过编号修复 GitHub 问题。`$ARGUMENTS` 占位符会被跟随在技能名后面的内容替换：

```yaml  theme={null}
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---

Fix GitHub issue $ARGUMENTS following our coding standards.

1. Read the issue description
2. Understand the requirements
3. Implement the fix
4. Write tests
5. Create a commit
```

当你运行 `/fix-issue 123` 时，Claude 会收到 "Fix GitHub issue 123 following our coding standards..."

如果你使用参数调用技能，但技能没有包含 `$ARGUMENTS`，Claude Code 会将 `ARGUMENTS: <your input>` 附加到技能内容的末尾，这样 Claude 仍然能看到你输入的内容。

要按位置访问单个参数，使用 `$ARGUMENTS[N]` 或更简短的 `$N`：

```yaml  theme={null}
---
name: migrate-component
description: Migrate a component from one framework to another
---

Migrate the $ARGUMENTS[0] component from $ARGUMENTS[1] to $ARGUMENTS[2].
Preserve all existing behavior and tests.
```

运行 `/migrate-component SearchBar React Vue` 会将 `$ARGUMENTS[0]` 替换为 `SearchBar`，`$ARGUMENTS[1]` 替换为 `React`，`$ARGUMENTS[2]` 替换为 `Vue`。使用 `$N` 简写的相同技能：

```yaml  theme={null}
---
name: migrate-component
description: Migrate a component from one framework to another
---

Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```

## 高级模式

### 注入动态上下文

`!`command\`` 语法在技能内容发送给 Claude 之前运行 Shell 命令。命令输出会替换占位符，因此 Claude 接收到的是实际数据，而不是命令本身。

这个技能通过使用 GitHub CLI 获取实时 PR 数据来总结拉取请求（pull request，**PR**）。`!`gh pr diff\`` 和其他命令首先运行，它们的输出会被插入到提示词中：

```yaml  theme={null}
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Pull request context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Your task
Summarize this pull request...
```

当此技能运行时：

1. 每个 `!`command\`` 立即执行（在 Claude 看到任何内容之前）
2. 输出会替换技能内容中的占位符
3. Claude 接收到带有实际 PR 数据的完全渲染的提示词

这是预处理，而不是 Claude 执行的东西。Claude 只看到最终结果。

<Tip>
  要在技能中启用[扩展思考](/en/common-workflows#use-extended-thinking-thinking-mode)，请在技能内容的任何位置包含单词 "ultrathink"。
</Tip>

### 在子智能体中运行技能

当你希望技能在隔离环境中运行时，在前置元数据中添加 `context: fork`。技能内容成为驱动子智能体的提示词。它无法访问你的对话历史。

<Warning>
  `context: fork` 只对包含显式指令的技能有意义。如果你的技能包含"使用这些 API 约定"等指南而没有任务，子智能体收到指南但没有可操作的提示词，返回时就没有有意义的输出。
</Warning>

技能和[子智能体](/en/sub-agents)在两个方向上协同工作：

| 方法                     | 系统提示词                             | 任务                        | 也加载                   |
| :--------------------------- | :---------------------------------------- | :-------------------------- | :--------------------------- |
| 带有 `context: fork` 的技能   | 来自智能体类型（`Explore`、`Plan` 等） | SKILL.md 内容            | CLAUDE.md                    |
| 带有 `skills` 字段的子智能体 | 子智能体的 Markdown 主体                  | Claude 的委托消息 | 预加载的技能 + CLAUDE.md |

使用 `context: fork`，你在技能中编写任务并选择一个智能体类型来执行它。对于相反的情况（定义一个使用技能作为参考资料的自定义子智能体），请参阅[子智能体](/en/sub-agents#preload-skills-into-subagents)。

#### 示例：使用 Explore 智能体进行研究技能

这个技能在分叉的 Explore 智能体中运行研究。技能内容成为任务，智能体提供针对代码库探索优化的只读工具：

```yaml  theme={null}
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:

1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings with specific file references
```

当此技能运行时：

1. 创建一个新的隔离上下文
2. 子智能体将技能内容作为其提示词接收（"Research \$ARGUMENTS thoroughly..."）
3. `agent` 字段确定执行环境（模型、工具和权限）
4. 结果被总结并返回到你的主对话

`agent` 字段指定使用哪个子智能体配置。选项包括内置智能体（`Explore`、`Plan`、`general-purpose`）或 `.claude/agents/` 中的任何自定义子智能体。如果省略，使用 `general-purpose`。

### 限制 Claude 的技能访问

默认情况下，Claude 可以调用任何未设置 `disable-model-invocation: true` 的技能。定义 `allowed-tools` 的技能在技能处于活动状态时授予 Claude 对这些工具的访问权限，无需每次使用都批准。你的[权限设置](/en/iam)仍然管理所有其他工具的基准批准行为。像 `/compact` 和 `/init` 这样的内置命令无法通过 Skill 工具使用。

控制 Claude 可以调用哪些技能的三种方法：

**禁用所有技能**，通过在 `/permissions` 中拒绝 Skill 工具：

```
# 添加到拒绝规则：
Skill
```

**允许或拒绝特定技能**，使用[权限规则](/en/iam)：

```
# 仅允许特定技能
Skill(commit)
Skill(review-pr *)

# 拒绝特定技能
Skill(deploy *)
```

权限语法：`Skill(name)` 用于精确匹配，`Skill(name *)` 用于带有任何参数的前缀匹配。

**隐藏单个技能**，通过在其前置元数据中添加 `disable-model-invocation: true`。这会完全从 Claude 的上下文中移除该技能。

<Note>
  `user-invocable` 字段只控制菜单可见性，不控制 Skill 工具访问。使用 `disable-model-invocation: true` 来阻止程序化调用。
</Note>

## 共享技能

技能可以根据你的受众在不同范围内分发：

* **项目技能**：将 `.claude/skills/` 提交到版本控制
* **插件**：在你的[插件](/en/plugins)中创建 `skills/` 目录
* **托管**：通过[托管设置](/en/iam#managed-settings)在整个组织范围内部署

### 生成视觉输出

技能可以捆绑并运行任何语言的脚本，赋予 Claude 超出单个提示词可能范围的能力。一个强大的模式是生成视觉输出：交互式 HTML 文件，在浏览器中打开以探索数据、调试或创建报告。

这个例子创建了一个代码库浏览器：一个交互式树形视图，你可以展开和折叠目录，一目了然地看到文件大小，并通过颜色识别文件类型。

创建技能目录：

```bash  theme={null}
mkdir -p ~/.claude/skills/codebase-visualizer/scripts
```

创建 `~/.claude/skills/codebase-visualizer/SKILL.md`。描述告诉 Claude 何时激活此技能，指令告诉 Claude 运行捆绑的脚本：

````yaml  theme={null}
---
name: codebase-visualizer
description: Generate an interactive collapsible tree visualization of your codebase. Use when exploring a new repo, understanding project structure, or identifying large files.
allowed-tools: Bash(python *)
---

# Codebase Visualizer

Generate an interactive HTML tree view that shows your project's file structure with collapsible directories.

## Usage

Run the visualization script from your project root:

```bash
python ~/.claude/skills/codebase-visualizer/scripts/visualize.py .
```

This creates `codebase-map.html` in the current directory and opens it in your default browser.

## What the visualization shows

- **Collapsible directories**: Click folders to expand/collapse
- **File sizes**: Displayed next to each file
- **Colors**: Different colors for different file types
- **Directory totals**: Shows aggregate size of each folder
````

创建 `~/.claude/skills/codebase-visualizer/scripts/visualize.py`。此脚本扫描目录树并生成一个自包含的 HTML 文件，包含：

* 一个**摘要侧边栏**，显示文件数、目录数、总大小和文件类型数量
* 一个**条形图**，按文件类型分解代码库（按大小前 8 名）
* 一个**可折叠的树**，你可以展开和折叠目录，带有颜色编码的文件类型指示器

该脚本需要 Python，但只使用内置库，因此无需安装任何包：

```python expandable theme={null}
#!/usr/bin/env python3
"""Generate an interactive collapsible tree visualization of a codebase."""

import json
import sys
import webbrowser
from pathlib import Path
from collections import Counter

IGNORE = {'.git', 'node_modules', '__pycache__', '.venv', 'venv', 'dist', 'build'}

def scan(path: Path, stats: dict) -> dict:
    result = {"name": path.name, "children": [], "size": 0}
    try:
        for item in sorted(path.iterdir()):
            if item.name in IGNORE or item.name.startswith('.'):
                continue
            if item.is_file():
                size = item.stat().st_size
                ext = item.suffix.lower() or '(no ext)'
                result["children"].append({"name": item.name, "size": size, "ext": ext})
                result["size"] += size
                stats["files"] += 1
                stats["extensions"][ext] += 1
                stats["ext_sizes"][ext] += size
            elif item.is_dir():
                stats["dirs"] += 1
                child = scan(item, stats)
                if child["children"]:
                    result["children"].append(child)
                    result["size"] += child["size"]
    except PermissionError:
        pass
    return result

def generate_html(data: dict, stats: dict, output: Path) -> None:
    ext_sizes = stats["ext_sizes"]
    total_size = sum(ext_sizes.values()) or 1
    sorted_exts = sorted(ext_sizes.items(), key=lambda x: -x[1])[:8]
    colors = {
        '.js': '#f7df1e', '.ts': '#3178c6', '.py': '#3776ab', '.go': '#00add8',
        '.rs': '#dea584', '.rb': '#cc342d', '.css': '#264de4', '.html': '#e34c26',
        '.json': '#6b7280', '.md': '#083fa1', '.yaml': '#cb171e', '.yml': '#cb171e',
        '.mdx': '#083fa1', '.tsx': '#3178c6', '.jsx': '#61dafb', '.sh': '#4eaa25',
    }
    lang_bars = "".join(
        f'<div class="bar-row"><span class="bar-label">{ext}</span>'
        f'<div class="bar" style="width:{(size/total_size)*100}%;background:{colors.get(ext,"#6b7280")}"></div>'
        f'<span class="bar-pct">{(size/total_size)*100:.1f}%</span></div>'
        for ext, size in sorted_exts
    )
    def fmt(b):
        if b < 1024: return f"{b} B"
        if b < 1048576: return f"{b/1024:.1f} KB"
        return f"{b/1048576:.1f} MB"

    html = f'''<!DOCTYPE html>
<html><head>
  <meta charset="utf-8"><title>Codebase Explorer</title>
  <style>
    body {{ font: 14px/1.5 system-ui, sans-serif; margin: 0; background: #1a1a2e; color: #eee; }}
    .container {{ display: flex; height: 100vh; }}
    .sidebar {{ width: 280px; background: #252542; padding: 20px; border-right: 1px solid #3d3d5c; overflow-y: auto; flex-shrink: 0; }}
    .main {{ flex: 1; padding: 20px; overflow-y: auto; }}
    h1 {{ margin: 0 0 10px 0; font-size: 18px; }}
    h2 {{ margin: 20px 0 10px 0; font-size: 14px; color: #888; text-transform: uppercase; }}
    .stat {{ display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #3d3d5c; }}
    .stat-value {{ font-weight: bold; }}
    .bar-row {{ display: flex; align-items: center; margin: 6px 0; }}
    .bar-label {{ width: 55px; font-size: 12px; color: #aaa; }}
    .bar {{ height: 18px; border-radius: 3px; }}
    .bar-pct {{ margin-left: 8px; font-size: 12px; color: #666; }}
    .tree {{ list-style: none; padding-left: 20px; }}
    details {{ cursor: pointer; }}
    summary {{ padding: 4px 8px; border-radius: 4px; }}
    summary:hover {{ background: #2d2d44; }}
    .folder {{ color: #ffd700; }}
    .file {{ display: flex; align-items: center; padding: 4px 8px; border-radius: 4px; }}
    .file:hover {{ background: #2d2d44; }}
    .size {{ color: #888; margin-left: auto; font-size: 12px; }}
    .dot {{ width: 8px; height: 8px; border-radius: 50%; margin-right: 8px; }}
  </style>
</head><body>
  <div class="container">
    <div class="sidebar">
      <h1>📊 Summary</h1>
      <div class="stat"><span>Files</span><span class="stat-value">{stats["files"]:,}</span></div>
      <div class="stat"><span>Directories</span><span class="stat-value">{stats["dirs"]:,}</span></div>
      <div class="stat"><span>Total size</span><span class="stat-value">{fmt(data["size"])}</span></div>
      <div class="stat"><span>File types</span><span class="stat-value">{len(stats["extensions"])}</span></div>
      <h2>By file type</h2>
      {lang_bars}
    </div>
    <div class="main">
      <h1>📁 {data["name"]}</h1>
      <ul class="tree" id="root"></ul>
    </div>
  </div>
  <script>
    const data = {json.dumps(data)};
    const colors = {json.dumps(colors)};
    function fmt(b) {{ if (b < 1024) return b + ' B'; if (b < 1048576) return (b/1024).toFixed(1) + ' KB'; return (b/1048576).toFixed(1) + ' MB'; }}
    function render(node, parent) {{
      if (node.children) {{
        const det = document.createElement('details');
        det.open = parent === document.getElementById('root');
        det.innerHTML = `<summary><span class="folder">📁 ${{node.name}}</span><span class="size">${{fmt(node.size)}}</span></summary>`;
        const ul = document.createElement('ul'); ul.className = 'tree';
        node.children.sort((a,b) => (b.children?1:0)-(a.children?1:0) || a.name.localeCompare(b.name));
        node.children.forEach(c => render(c, ul));
        det.appendChild(ul);
        const li = document.createElement('li'); li.appendChild(det); parent.appendChild(li);
      }} else {{
        const li = document.createElement('li'); li.className = 'file';
        li.innerHTML = `<span class="dot" style="background:${{colors[node.ext]||'#6b7280'}}"></span>${{node.name}}<span class="size">${{fmt(node.size)}}</span>`;
        parent.appendChild(li);
      }}
    }}
    data.children.forEach(c => render(c, document.getElementById('root')));
  </script>
</body></html>'''
    output.write_text(html)

if __name__ == '__main__':
    target = Path(sys.argv[1] if len(sys.argv) > 1 else '.').resolve()
    stats = {"files": 0, "dirs": 0, "extensions": Counter(), "ext_sizes": Counter()}
    data = scan(target, stats)
    out = Path('codebase-map.html')
    generate_html(data, stats, out)
    print(f'Generated {out.absolute()}')
    webbrowser.open(f'file://{out.absolute()}')
```

要测试，在任何项目中打开 Claude Code 并问"可视化这个代码库。"Claude 运行脚本，生成 `codebase-map.html`，并在你的浏览器中打开它。

此模式适用于任何视觉输出：依赖图、测试覆盖率报告、API 文档或数据库架构可视化。捆绑脚本完成繁重的工作，而 Claude 负责编排。

## 故障排除

### 技能没有触发

如果 Claude 在预期时没有使用你的技能：

1. 检查描述是否包含用户自然会说出的关键词
2. 验证技能是否出现在"有哪些技能可用？"中
3. 尝试重新表达你的请求，使其更接近描述
4. 如果技能是用户可调用的，直接用 `/skill-name` 调用它

### 技能触发过于频繁

如果 Claude 在你不希望的时候使用你的技能：

1. 使描述更具体
2. 如果你只想手动调用，添加 `disable-model-invocation: true`

### Claude 看不到我所有的技能

技能描述会被加载到上下文中，这样 Claude 就知道有什么可用。如果你有很多技能，它们可能会超出字符预算（默认 15,000 个字符）。运行 `/context` 检查是否有关于被排除技能的警告。

要增加限制，请设置 `SLASH_COMMAND_TOOL_CHAR_BUDGET` 环境变量。

## 相关资源

* **[子智能体](/en/sub-agents)**：将任务委托给专门的智能体
* **[插件](/en/plugins)**：与其他扩展一起打包和分发技能
* **[钩子](/en/hooks)**：围绕工具事件自动化工作流
* **[记忆](/en/memory)**：管理 CLAUDE.md 文件以持久化上下文
* **[交互模式](/en/interactive-mode#built-in-commands)**：内置命令和快捷键
* **[权限](/en/iam)**：控制工具和技能访问
