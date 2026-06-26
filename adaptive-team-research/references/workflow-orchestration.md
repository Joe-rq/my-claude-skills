# Dynamic Workflow 编排协议

> 本文件规定 Round 1（事实收集）与 Round 2（交叉辩论）如何由 **Workflow 工具**确定性执行。
> Phase 0（模式选择）与 Round 3（共识收敛）仍由主会话执行——这是不可妥协的边界，原因见下文。

---

## 1. 为什么用 Dynamic Workflow

旧版编排完全靠 SKILL.md 的自然语言描述、由主会话即兴调用 agent。这恰好是这个技能要防御的三种失效模式的温床：

| 失效模式 | 旧版（prompt 编排）的表现 | Workflow 如何结构性消除 |
|---------|--------------------------|------------------------|
| Agentic laziness | Team Lead 可能 Round 1 只跑 2 个 agent 就"觉得够了" | `parallel()` 的 barrier **强制**等齐全部 agent 才进下一轮 |
| Goal drift | 三轮长 context 中丢失"只写事实/禁用词"约束 | 每个 agent 独立 context，约束写死在 prompt + schema |
| Self-preferential bias | Team Lead 自己整合易偏爱自己视角 | Round 2 独立 Critic agent 对抗质询，输出经 schema 结构化 |

同时，schema 化输出让 Round 1 事实清单天然带 `文件路径:行号` 来源字段，比旧版"自然语言事实清单 + 禁用词检查"更硬。

---

## 2. 三段拆分（核心架构）

```
[主会话] Phase 0：模式选择 + 强制门禁（用户确认）   ← 必须留在外部
   ↓   mode / project / canvas_path 作为 args 传入
[Workflow] Round 1：parallel fan-out 三角色事实收集  ← barrier 等齐
[Workflow] Round 2：按 mode 分支 fan-out 交叉 + Critic
   ↓   结构化结果返回主会话
[主会话] Round 3：Team Lead 综合裁决 + 写画布 + 交付  ← 必须留在外部
```

### 为什么 Phase 0 和 Round 3 必须在 Workflow 外

- **Phase 0 的强制门禁**要求"模式选定后停下等用户确认，禁止自动进 Round 1"。Workflow 是后台一次性跑完的确定性脚本，**没有"中途停下来等用户输入"的机制**（只能暂停/停止）。把 Phase 0 塞进 Workflow，门禁就没了——这会让评测 exp-1（确认后才进 Round 1）退化。
- **Round 3** 本就规定由 Team Lead 在主会话执行、不委派 agent、且要写画布文件。留在主会话是顺水推舟，不是妥协。

→ Workflow 只接管 Round 1 + Round 2 这两段"真正需要并行 + 对抗"的部分。拆分干净。

---

## 3. 主会话职责

### Phase 0（模式选择 + 强制门禁）

1. 按 `mode-selection.md` 的决策树分析任务特征，选定模式（`centralized` / `domain-lead` / `peer`；领域主导还需定 Lead 角色）
2. 向用户展示所选模式、理由、备选模式，**停下等待确认**
3. 用户确认后，按 §5 组装各角色 prompt 文本，连同 mode/project/canvas_path 打包成 `args`，调用 Workflow 工具

> ⚠️ 强制门禁原义不变：用户确认前**禁止**调用 Workflow。Workflow 一旦启动即会跑完 Round 1-2。

### Round 3（共识收敛 + 交付）

1. 接收 Workflow 返回的 `{ mode, round1, round2 }` 结构化结果
2. 从 `assets/canvas-template.md` 复制画布到项目 `reviews/{project}-review.md`，替换模板变量
3. 将 Round 1 事实、Round 2 投票矩阵/Critic 质询写入画布对应区域
4. 按 `round-protocols.md` Round 3 的模式规则做裁决（集中/领域=直接裁决；对等=投票矩阵 + 共识判定）
5. Critic 关键质询逐条回应，不充分则归入"待定：需用户确认"
6. 输出 P0/P1/P2/待定 行动计划（每项含 Engineer 成本 + 负责角色），写入画布并交付摘要

---

## 4. Workflow 职责（Round 1 + Round 2）

### 输入 `args`（由主会话组装）

```jsonc
{
  "mode": "centralized | domain-lead | peer",
  "leadRole": "PM | Designer | Engineer",   // 仅 domain-lead 需要
  "projectName": "my-web-app",
  "projectDir": "/path/to/project",
  "canvasPath": "/path/to/reviews/my-web-app-review.md",
  "prompts": {
    "pm_r1": "...", "designer_r1": "...", "engineer_r1": "...",
    "pm_cross": "...", "designer_cross": "...", "engineer_cross": "...",
    "lead_cross": "...", "critic": "..."
  }
}
```

> prompt 是**文本字符串**，由主会话按 §5 从 `role-prompts-r1.md` / `role-prompts-r2.md` 组装后传入。Workflow 脚本内**不能读文件**（无 fs），所以 prompt 必须经 args 传入，不能在脚本里 `Read()`。
>
> **Round 1 prompt** 是完整文本。**Round 2 prompt 是模板**（只含角色评论/质询内容）——脚本会在 Round 1 完成后自动把三角色结构化事实注入（见 §6.2），因此 Round 2 prompt **不要**包含 Round 1 事实。Round 2 agent 不再读取画布文件；画布降级为纯交付物，由主会话在 Round 3 一次性生成。

### 输出（return 给主会话）

```jsonc
{
  "mode": "centralized",
  "round1": [ {role, facts: [{claim, source}]}, ... ],   // 三角色事实
  "round2": { critic: {...}, cross: [...] }              // 按模式结构不同
}
```

---

## 5. Prompt 映射规则

### Round 1（mode → 各角色用哪个版本）

| 模式 | PM | Designer | Engineer |
|------|----|----------|---------|
| 集中调度 | 标准版 | 标准版 | 标准版 |
| 领域主导 | Lead→加深版 / 非 Lead→精简版 | 同左 | 同左 |
| 对等协作 | 标准版 | 标准版 | 标准版 |

### Round 2（mode → 启动哪些 agent）

| 模式 | 启动 agent | 数量 |
|------|-----------|------|
| 集中调度 | 仅 Critic（交叉评审由主会话 Round 3 做） | 1 |
| 领域主导 | Lead 交叉评审 + Critic | 2 |
| 对等协作 | PM/Designer/Engineer 交叉评论者 + Critic | 4 |

> 这张表直接对应评测 exp-3（Round 2 agent 数）。Workflow 按模式分支起 agent，让 agent 数量比旧版 prompt 编排**更确定**。

### Prompt 组装方式（主会话执行）

每个 prompt = 对应通用规则（Round 1 / Round 2 通用规则）+ 角色身份声明 + 调研/评论内容，变量（`{{PROJECT_NAME}}` 等）替换为实际值。详见 `role-prompts-r1.md` / `role-prompts-r2.md` 末尾的"组装示例"。

---

## 6. Workflow 脚本骨架

> **这是模板，不是死脚本。** 主会话应根据具体任务调整 prompt 内容、agent 数量与 schema 字段。Wiki 最佳实践明确："通过 skill 分发工作流时，把 workflow 当模板而非逐字照搬的脚本，留出灵活性。"

### 6.1 schema 定义

```js
// Round 1 事实清单 —— 强制带来源，结构性落实"只写事实"
const FACT_SCHEMA = {
  type: 'object',
  properties: {
    role: { type: 'string', enum: ['PM', 'Designer', 'Engineer'] },
    facts: {
      type: 'array',
      items: {
        type: 'object',
        properties: {
          claim: { type: 'string', description: '纯事实陈述，禁用评价词' },
          source: { type: 'string', description: '文件路径:行号；无来源留空' },
        },
        required: ['claim'],
      },
    },
  },
  required: ['role', 'facts'],
}

// Round 2 Critic 质询
const CRITIC_SCHEMA = {
  type: 'object',
  properties: {
    inquiries: {
      type: 'array',
      items: {
        type: 'object',
        properties: {
          target: { type: 'string', description: 'PM / Designer / Engineer / 全体' },
          question: { type: 'string' },
          evidence: { type: 'string' },
          is_critical: { type: 'boolean', description: '关键质询（最多 3 条）' },
        },
        required: ['target', 'question', 'evidence'],
      },
    },
  },
  required: ['inquiries'],
}

// Round 2 交叉评论（对等 / 领域主导 Lead 交叉）
const CROSS_SCHEMA = {
  type: 'object',
  properties: {
    reviewer: { type: 'string' },
    comments: {
      type: 'array',
      items: {
        type: 'object',
        properties: {
          target_role: { type: 'string' },
          topic: { type: 'string' },
          stance: { type: 'string', enum: ['+1', '反驳'] },
          reason: { type: 'string' },
          cost_hours: { type: 'number', description: '仅 Engineer 交叉评论必填' },
        },
        required: ['target_role', 'topic', 'stance', 'reason'],
      },
    },
    cross_findings: { type: 'array', items: { type: 'string' } },
  },
  required: ['reviewer', 'comments'],
}
```

### 6.2 三模式统一骨架

```js
export const meta = {
  name: 'adaptive-team-round12',
  description: '自适应团队评审 Round1-2：按模式 fan-out 事实收集与交叉辩论',
  phases: [
    { title: 'Round 1', detail: '并行事实收集（PM/Designer/Engineer）' },
    { title: 'Round 2', detail: '按模式交叉辩论 + Critic 质询' },
  ],
}

const { mode, leadRole, projectDir, canvasPath, prompts } = JSON.parse(args)  // args 运行时是 JSON 字符串，需解析（见 §7）

// ---- Round 1：三角色并行事实收集（barrier 等齐）----
phase('Round 1')
const facts = (await parallel([
  () => agent(prompts.pm_r1,        { label: 'PM-R1',        phase: 'Round 1', agentType: 'Explore', schema: FACT_SCHEMA }),
  () => agent(prompts.designer_r1,  { label: 'Designer-R1',  phase: 'Round 1', agentType: 'Explore', schema: FACT_SCHEMA }),
  () => agent(prompts.engineer_r1,  { label: 'Engineer-R1',  phase: 'Round 1', agentType: 'Explore', schema: FACT_SCHEMA }),
])).filter(Boolean)   // 任一 agent 失败不拖垮整体

// ---- 把 Round 1 结构化事实序列化，注入 Round 2 各 agent 的 prompt ----
// 替代旧版"Round 2 agent 读画布文件"；画布由主会话在 Round 3 生成
const factsBlock = '\n\n【Round 1 已收集事实（仅供评论/质询参考，不得照抄或评价）】\n' +
  facts.map(f => `### ${f.role}\n` +
    f.facts.map(x => `- ${x.claim}${x.source ? `（来源：${x.source}）` : ''}`).join('\n')
  ).join('\n\n')

// ---- Round 2：按模式分支 ----
phase('Round 2')
let round2

if (mode === 'centralized') {
  // 仅 Critic × 1；交叉评审由主会话在 Round 3 完成
  const critic = await agent(prompts.critic + factsBlock, { label: 'Critic', phase: 'Round 2', agentType: 'Explore', schema: CRITIC_SCHEMA })
  round2 = { critic, cross: [] }

} else if (mode === 'domain-lead') {
  // Lead 交叉评审 + Critic × 2
  const [leadCross, critic] = (await parallel([
    () => agent(prompts.lead_cross + factsBlock, { label: 'Lead交叉', phase: 'Round 2', agentType: 'Explore', schema: CROSS_SCHEMA }),
    () => agent(prompts.critic + factsBlock,     { label: 'Critic',    phase: 'Round 2', agentType: 'Explore', schema: CRITIC_SCHEMA }),
  ])).filter(Boolean)
  round2 = { critic, cross: leadCross ? [leadCross] : [] }

} else {  // peer
  // 3 交叉评论者 + Critic × 4
  const [pmCross, designerCross, engineerCross, critic] = (await parallel([
    () => agent(prompts.pm_cross + factsBlock,        { label: 'PM交叉',        phase: 'Round 2', agentType: 'Explore', schema: CROSS_SCHEMA }),
    () => agent(prompts.designer_cross + factsBlock,  { label: 'Designer交叉',  phase: 'Round 2', agentType: 'Explore', schema: CROSS_SCHEMA }),
    () => agent(prompts.engineer_cross + factsBlock,  { label: 'Engineer交叉',  phase: 'Round 2', agentType: 'Explore', schema: CROSS_SCHEMA }),
    () => agent(prompts.critic + factsBlock,          { label: 'Critic',        phase: 'Round 2', agentType: 'Explore', schema: CRITIC_SCHEMA }),
  ])).filter(Boolean)
  round2 = { critic, cross: [pmCross, designerCross, engineerCross].filter(Boolean) }
}

return { mode, round1: facts, round2 }
```

---

## 7. Workflow 脚本技术约束（避免踩坑）

主会话生成脚本时务必遵守：

1. **纯 JavaScript，不是 TypeScript** —— 无类型注解、无 interface、无泛型，会解析失败。
2. **`meta` 必须是纯字面量** —— 不能用变量、函数调用、模板插值、展开运算符。
3. **禁用 `Date.now()` / `Math.random()` / 无参 `new Date()`** —— 会抛异常（破坏可恢复性）。需要时间戳由主会话通过 args 传入；需要随机性则按 agent 索引变化 prompt/label。
4. **无文件系统 / Node API** —— 不能 `Read()` / `fs` / `require`。读画布、读项目代码都由 `agentType: 'Explore'` 的子 agent 在自己的工具上下文里完成；写画布由主会话在 Round 3 完成。
5. **`parallel()` 是 barrier** —— 任一 thunk 抛错返回 `null`，需 `.filter(Boolean)` 清理后再用。
6. **`args` 运行时是 JSON 字符串（实测）** —— 本仓库 v1.4.0 端到端验证发现，Workflow 工具传入的 args 在脚本里 `typeof args === 'string'`（被序列化），**不是对象**。直接 `const {mode} = args` 会解构出 undefined 导致崩溃。必须 `JSON.parse(args)` 后再解构：`const { mode, prompts } = JSON.parse(args)`。若 prompts 体量较大，也可直接内联进脚本（实测内联稳定，绕开解析）。
7. **`return` 的值就是主会话收到的结果** —— 无需 `log()` 输出给用户；`log()` 只用于进度播报。

---

## 8. 模式切换与混合模式的处理

`mode-selection.md` 中的"混合模式"与"中途切换"在 Workflow 化后按如下方式落实：

- **模式在 Phase 0 由主会话确定为单一 `mode`** 传入 Workflow。Workflow 运行中**不支持**动态改拓扑（agent 数、prompt 版本已按该 mode 固化）。
- **中途切换** = 主会话重新评估任务特征 → 重选 mode → 在画布"模式切换日志"记录 → 重新组装 args 调用 Workflow。代价是 Round 1-2 重跑，但中途切换本就是低频高级特性，可接受。
- **混合方案**（如"先对等探索、后集中裁决"）：对等模式跑完 Round 1-2 拿到结构化结果后，主会话在 Round 3 用集中式裁决逻辑收敛——无需重启 Workflow，Round 3 本就在主会话，可自由选择裁决风格。

---

## 9. 何时回归 prompt 编排（不用 Workflow）

不是所有评审都值得起 Workflow（启动成本 + token 是单 agent 的 10-100 倍）。回归条件：

- 被评审项目极小（单文件 / 几十行）—— fan-out 收益低于开销
- 用户明确要"快速看看"，不需要完整三轮
- 只需单一视角（直接用 architect-reviewer 更高效）

判断口径：问自己"这真的需要并行 + 对抗吗？"不需要就退回主会话直接分析，并在交付时说明为何省略 Workflow。

---

## 10. 与其他协议文件的关系

| 文件 | 角色 | 本文件的关系 |
|------|------|-------------|
| `SKILL.md` | 总纲 | §工作流章节引用本文件作为 Round 1-2 执行细节 |
| `round-protocols.md` | 轮次协议（agent 数/类型/并行策略） | 本文件是其 Round 1-2 部分的**执行引擎实现**；拓扑（agent 数、依赖关系）以 round-protocols 为准 |
| `role-prompts-r1.md` / `role-prompts-r2.md` | prompt 内容库 | 本文件 §5 规定如何从中选版本并组装进 `agent()` |
| `mode-selection.md` | 模式选择 | 本文件 §8 规定其混合/切换语义在 Workflow 下的落实 |
| `assets/canvas-template.md` | 画布模板 | 由主会话在 Round 3 填充，不经 Workflow |
