# my-claude-skills — 自适应多智能体研究画布

> 协作模式：集中调度
> 日期：2026-02-18
> 审查目标：项目整体审查

---

## 模式选择

**选定模式：** 集中调度

**选择理由：**
通用项目审查，无特定领域需要深入，非探索性任务。集中调度模式最适合。

---

## Round 1：事实层（只写事实，不评价）

### PM — 产品视角

**产品定位：**
- 自我描述："个人自定义的 Claude Skills 集合，用于扩展 Claude 在特定领域的专业能力"（README:3）
- MIT 许可证，维护者 Joe，创建于 2024-12（README:137）
- 基于 Claude 官方 Skills 框架，无独立运行时

**功能完成度：**
- README 列出 10 个 skill，实际存在 11 个目录（agent-skills-doc 未收录）
- 8/11 skill 有完整 SKILL.md 实现
- 2 个 skill 使用非标准 `skills.md`（primitive-ir-compiler、source-tracing）
- explain-code 仅 21 行框架级定义，无详细指导
- excalidraw-prompt-engineer 有 2 个生产级 JS 脚本（箭头优化、JSON 修复）

**竞品功能对比：**
- 同类项目（awesome-claude-skills 等社区集合）通常有统一模板和自动化安装
- 本项目无自动化安装脚本、无 skill 验证机制、无版本管理

**目标用户线索：**
- 纯文本交互，无 UI 界面
- 手动 `cp -r` 安装，需技术背景
- 无 API/SDK、无多租户、无权限系统
- 目标用户：开发者 / Claude 高级用户

### Designer — 用户视角

**页面清单与结构：**
- README 列出 4 个分类：内容创作(2)、思维与决策(3)、开发工具(3)、专业领域(2)
- 实际 11 个 skill 目录 + 1 个未收录的 agent-skills-doc
- 4/11 skill 有 README.md（explain-code、medical-triage、mermaid-ascii-renderer、writing-assistant-skill）
- 3/11 有 references/ 目录
- 2/11 有 assets/ 目录

**交互模式统计：**
- 3 种触发方式：明确触发词列表（adaptive-team-research）、场景描述触发（writing-assistant）、结构化输入（decision-card）
- 3 种 frontmatter 格式：标准 3 字段、扩展含 metadata、结构化含 inputs/outputs
- 无统一的触发机制规范

**组件一致性事实：**
- 入口文件命名不一致：8 个用 SKILL.md，2 个用 skills.md
- 版本号覆盖率：2/11（mermaid-ascii-renderer v1.1.0, explain-code v1.0.0）
- 作者字段：2/11 有（mermaid-ascii-renderer: joe, explain-code: user4363）
- 语言分布：6 中文、3 英文、2 中英混合

**无障碍事实：**
- 无 i18n 机制，语言由作者个人习惯决定
- 安装说明重复出现在 4 处（README + 3 个 skill 的 README.md），内容一致

### Engineer — 技术视角

**存储层事实：**
- 201 个文件（含 .git），54 个 .md 文件，~8400 行
- 2 个 .js 脚本（excalidraw-prompt-engineer/scripts/）
- 1 个 .json 配置（.claude/settings.local.json）
- 无 package.json、pyproject.toml、tsconfig.json

**核心引擎事实：**
- 无运行时引擎，纯文档驱动（Claude Skills 框架消费 Markdown）
- 模板变量系统仅在 adaptive-team-research 的 role-prompts.md 中使用（{{PROJECT_NAME}} 等 5 个变量）
- 唯一可执行代码：json-repair.js（JSON 闭包修复）和 optimizeArrows.js（箭头优化算法）

**API/协议事实：**
- 无 REST/GraphQL API
- 无 SDK
- 交互协议完全依赖 Claude Skills 的 frontmatter + Markdown 约定

**安全机制事实：**
- .claude/settings.local.json 限制权限：仅允许 git add/commit/push 和 gh run list
- 无敏感信息泄露（.gitignore 覆盖 .env、node_modules 等）
- medical-triage 包含免责声明

**依赖和模块耦合事实：**
- 各 skill 完全独立，零跨 skill 引用
- 20 个 git commit，1 个贡献者（Joe-rq），单一 master 分支
- 无 git tags、无 CI/CD、无自动化测试
- 提交模式呈爆发式：2026-01-30 单日 11 个 commit（占总量 55%）

---

## Round 2：辩论层

### 交叉评论投票矩阵

#### 简化投票矩阵（集中调度模式）

| 议题 | 裁决者立场 | Critic 质询 | 最终裁决 |
|------|-----------|------------|---------|
| README 与目录不同步 | agent-skills-doc 未收录 | agent-skills-doc 不是 skill，是参考资料。实际 skill 数=10，README 也列 10 个，数量是对的 | Critic 胜：根因是参考资料混在 skill 同层级，需命名区分而非收录 |
| 入口文件命名不统一 | skills.md 违反规范 | 同意违规，但追问：这两个文件的 inputs/outputs 字段也非 Claude 标准，可能源自其他框架迁移残留 | 共识：需重命名 + 清理非标准字段，但需确认字段用途 |
| 版本管理缺失 | 无 tags 无 release | prompt 集合不等于软件包，语义化版本对 prompt 无共识定义，git log 已是版本追踪 | Critic 胜：单人 prompt 仓库无需完整 release 流程，git log 足够 |
| 语言碎片化 | 6中+3英+2混 | 语言选择与 skill 目标语境对齐，强行统一成本高收益低 | Critic 胜：维持按场景选语言，但在 README 中标注每个 skill 的语言 |
| Frontmatter 格式分裂 | 3 种格式无 schema | Claude 只认 name/description 等官方字段，其他字段是给人读的文档。强行统一可能丢失人类接口文档 | 部分共识：统一 Claude 消费的官方字段，保留人类文档字段但移至正文 |
| 自动化为零 | 无 CI/CD/测试 | 纯 markdown 仓库能跑什么 CI？且自动安装可能绕过 medical-triage 免责声明 | Critic 胜：暂不需要 CI，但可考虑 frontmatter lint |
| 文档深度极不均 | 21 行 vs 662 行 | explain-code 是官方 demo 副本，不应与工程级 skill 比行数。深度应匹配任务复杂度 | Critic 胜：深度不均是合理的，关键是每个 skill 是否匹配自身复杂度 |

### 独到发现

1. **`skills.md` 的 inputs/outputs 字段不被 Claude 识别** — 这些 skill 可能无法被 Claude Code 正确加载，是功能性缺陷而非仅仅是命名问题
2. **手动安装流程意外形成了安全屏障** — 强制用户进入目录查看内容，medical-triage 的免责声明因此被自然暴露
3. **agent-skills-doc 是开发时上下文注入材料** — 不是遗漏，是维护者给 Claude Code 的本地参考文档

### Critic 质询

1. [Critic → Team Lead] "README 不同步"的诊断是错的。10 个 skill 对 10 个列表项，数量一致。真正的问题是参考资料目录（agent-skills-doc）与 skill 目录混在同一层级，无命名约定区分。
   - 裁决：接受。根因已修正。

2. [Critic → PM] 竞品对比中"有统一模板和自动化安装"被当作差距，但未论证自动化安装对个人 prompt 仓库的收益。手动安装的"缺陷"恰好是安全特性。
   - 裁决：接受。自动安装降级为可选项。

3. [Critic → Engineer] 报告遗漏了关键问题：哪些 skill 能被 Claude Code 实际加载？`skills.md`（非 `SKILL.md`）的 skill 可能根本不工作。
   - 裁决：接受。这是最高优先级修复项。

4. [Critic → Designer] "均匀 = 健康"的隐含假设不成立。prompt 工程中指令精简度比文档厚度更重要。
   - 裁决：接受。不再追求文档行数均匀化。

---

## Round 3：共识层

### ✅ 共识项

1. **`skills.md` → `SKILL.md` 重命名**
   **结论：** primitive-ir-compiler 和 source-tracing 的入口文件必须重命名为 SKILL.md，否则 Claude Code 无法加载
   **行动：** 重命名文件 + 验证 Claude Code 能否识别
   **成本估算：** 极低（改文件名 + 调整 frontmatter 为标准格式）
   **优先级：** P0

2. **参考资料目录与 skill 目录分离**
   **结论：** agent-skills-doc 不是 skill，需通过命名约定区分（如改名为 `_docs` 或移至项目根 `docs/`）
   **行动：** 重命名 agent-skills-doc 为 `docs/` 或加 `_` 前缀
   **成本估算：** 极低
   **优先级：** P0

3. **非标准 Frontmatter 字段清理**
   **结论：** Claude 只消费 name/description 等官方字段。inputs/outputs 等自定义字段对 Claude 无意义，但可能对人类有文档价值
   **行动：** 保留 name + description 在 frontmatter 中（Claude 消费），将 inputs/outputs/examples 移至 SKILL.md 正文（人类消费）
   **成本估算：** 低（2 个文件需重构 frontmatter）
   **优先级：** P1

4. **README 补充 skill 语言标注**
   **结论：** 维持按场景选语言的策略，但在 README 表格中增加"语言"列
   **行动：** README Skills 列表表格加一列
   **成本估算：** 极低
   **优先级：** P1

### ⚡ 分歧项

1. **是否需要 Frontmatter schema 校验**
   **各方立场：**
   - PM/Designer：统一格式有利于一致性
   - Engineer：纯 markdown 仓库搭 CI 成本过高
   - Critic：frontmatter lint 脚本足够，无需完整 CI
   **裁决：** 暂不搭建 CI，可选择性加入本地 lint 脚本（如 markdown-lint + 自定义 frontmatter 检查）
   **决策依据：** 单人维护仓库，CI 的维护成本可能超过收益

2. **是否统一所有 skill 的文档深度**
   **各方立场：**
   - Designer：最低深度标准有助于用户体验
   - Critic：深度应匹配任务复杂度，explain-code 是 demo 不需要膨胀
   **裁决：** 不设最低行数标准，但每个 skill 需包含：至少 1 个使用示例 + 触发方式说明
   **决策依据：** explain-code 当前确实缺少使用示例，但解法是加示例而非膨胀文档

---

## 被低估的优势

1. **完全独立的 skill 架构** — 零跨 skill 引用不是缺陷，是与 Claude skill 加载机制完美对齐的设计。每个 skill 可独立安装、独立删除，无隐藏依赖
2. **内嵌的知识基线** — agent-skills-doc 目录收录了 Claude 官方 skill 开发文档的本地副本，使维护者在离线环境下也能参考标准规范
3. **手动安装作为安全屏障** — 强制用户进入 skill 目录操作，自然暴露 medical-triage 等敏感 skill 的免责声明

---

## 行动计划（按 ROI 排序）

| 优先级 | 行动 | 成本 | 价值 | 负责 | 状态 |
|--------|------|------|------|------|------|
| P0 | 重命名 skills.md → SKILL.md（primitive-ir-compiler, source-tracing） | 极低 | 高（功能性修复） | 维护者 | 待执行 |
| P0 | 重命名 agent-skills-doc → docs/ 或 _references/ | 极低 | 高（消除结构歧义） | 维护者 | 待执行 |
| P1 | 清理非标准 frontmatter（inputs/outputs 移至正文） | 低 | 中（对齐官方规范） | 维护者 | 待执行 |
| P1 | README Skills 表格增加"语言"列 | 极低 | 中（提升可发现性） | 维护者 | 待执行 |
| P2 | 为 explain-code 补充使用示例 | 低 | 中（提升实用性） | 维护者 | 待执行 |
| 待定 | 本地 frontmatter lint 脚本 | 中 | 低（单人仓库） | 维护者 | 需确认 |

### P0：立即执行（高价值、低成本）

1. `primitive-ir-compiler/skills.md` → `primitive-ir-compiler/SKILL.md`，同时清理 frontmatter 为 Claude 标准格式
2. `source-tracing/skills.md` → `source-tracing/SKILL.md`，同上
3. `agent-skills-doc/` → `docs/`（或 `_references/`），与 skill 目录在命名层面区分

### P1：本轮迭代（高价值、中等成本）

1. primitive-ir-compiler 和 source-tracing 的 inputs/outputs/examples 从 frontmatter 移至 SKILL.md 正文
2. README Skills 列表表格增加"语言"列（中文/英文/双语）

### P2：下个周期规划（中等价值、较高成本）

1. 为 explain-code 补充 2-3 个使用示例（代码解释场景）

### 待定：需用户确认

1. 是否需要本地 frontmatter lint 脚本？（成本中等，对单人仓库收益有限）

---

## 摘要

**使用模式：** 集中调度

**关键共识：**
- 4 项达成共识

**关键分歧：**
- 2 项需要决策者介入（schema 校验、文档深度标准）

**Top 3 P0 行动：**
1. 重命名 skills.md → SKILL.md（2 个文件），修复 Claude Code 无法加载的功能性缺陷
2. 重命名 agent-skills-doc → docs/，消除参考资料与 skill 目录的结构歧义
3. 清理非标准 frontmatter 字段，对齐 Claude 官方规范

**Critic 关键洞察：**
- 7 个议题中 Critic 推翻了 4 个的前提假设（版本管理、语言碎片化、自动化缺失、文档深度不均）
- 最有价值的发现：`skills.md` 命名问题不仅是规范违反，更是功能性缺陷（Claude Code 不识别）
- 手动安装流程被错误归类为缺陷，实际是安全特性

---

## 附录：模式切换日志

无
