---
name: adaptive-team-research
description: >-
  Adaptive multi-agent team for software project reviews. Auto-selects collaboration mode
  (centralized / domain-lead / peer) and runs a 3-round workflow (facts → debate → consensus)
  to produce actionable plans with cost estimates. Triggers: design review, architecture review,
  code audit, multi-perspective analysis.
metadata:
  author: joe
  version: 1.1.0
  title: 自适应多智能体研究团队
---

# 自适应多智能体研究团队

## 概述

编排自适应多智能体研究团队，根据任务特征选择最优协作模式，通过三轮结构化工作流（事实收集 → 交叉辩论 → 共识收敛）产出可操作的洞察和行动计划。

三种模式不是文字标签，而是在 agent 数量、prompt 内容、画布结构上有实质差异的执行路径。

## 适用场景

- 项目设计评审、架构评估
- 产品策略分析（需要多方博弈视角）
- 代码质量审计（需要跨职能交叉验证）
- 用户明确要求多角度或团队协作式分析的软件项目评审任务

## 三种模式的实质差异

| 维度 | 集中调度 | 领域主导 | 对等协作 |
|------|---------|---------|---------|
| Round 1 agent 数 | 3（并行） | 3（Lead 加深，其余精简） | 3（并行，对等） |
| Round 2 agent 数 | **1**（仅 Critic） | **2**（Lead 交叉 + Critic） | **4**（3 交叉 + Critic） |
| Round 2 交叉评审 | Team Lead 自行完成 | Lead 评审全部，其余不参与 | 三方互评 |
| Round 3 决策方式 | Team Lead 直接裁决 | Lead 综合裁决 | 投票矩阵 + 共识收敛 |
| 适用场景 | 时间紧、产品决策、不确定选哪个模式 | 某领域需要特别深入 | 跨职能复杂权衡 |

## 工作流

### Phase 0：模式选择

分析任务特征，选择协作模式。详见 `references/mode-selection.md`。

**集中调度模式**（默认，推荐用于产品决策、时间紧迫、不确定选哪个模式）
- Team Lead 协调并行研究，自行完成交叉验证和裁决
- 最快收敛，最高输出完整度
- Round 2 仅启动 Critic，交叉评审由 Team Lead 自行完成

**领域主导模式**（推荐用于某个维度需要特别深入的场景）
- 领域专家（PM/Designer/Engineer）主导，其余辅助
- Round 1 Lead 角色使用加深版 prompt，其余使用精简版
- Round 2 仅 Lead 进行交叉评审 + Critic

**对等协作模式**（推荐用于跨职能复杂权衡场景）
- 所有角色对等，结构化辩论
- Round 2 三方互评 + Critic，最大化交叉碰撞
- Round 3 使用完整投票矩阵

**用户确认点：** 模式选定后，向用户展示所选模式及选择理由，等待用户确认后再启动 Round 1。如果用户不同意，根据反馈重新选择。

### Phase 1：Round 1 — 事实收集（并行）

启动三个 Explore agent 并行执行，角色 prompt 从 `references/role-prompts.md` 加载。

**按模式区分：**

| 模式 | PM agent | Designer agent | Engineer agent |
|------|----------|---------------|----------------|
| 集中调度 | 标准 prompt | 标准 prompt | 标准 prompt |
| 领域主导 | Lead 加深版 / 精简版 | Lead 加深版 / 精简版 | Lead 加深版 / 精简版 |
| 对等协作 | 标准 prompt | 标准 prompt | 标准 prompt |

**关键约束：** 只写事实，不做评价，不给建议。禁用词："好"、"差"、"应该"、"建议"、"改进"。

**完成后 Team Lead 动作：**
1. 收集三份事实清单
2. 从 `assets/canvas-template.md` 复制画布模板到项目的 `reviews/` 目录（不存在则创建）
3. 替换模板变量：`{{PROJECT_NAME}}`、`{{DATE}}`、`{{REVIEW_TARGET}}`、`{{MODE}}`
4. 提炼关键事实写入画布 Round 1 区域

### Phase 2：Round 2 — 交叉辩论（按模式执行）

所有 agent 读取共享画布。角色 prompt 从 `references/role-prompts.md` 加载。

**集中调度模式（1 agent）：**
- 仅启动 Critic agent
- 交叉评审由 Team Lead 自行完成（不启动额外 agent）
- Team Lead 读取三方事实，自行写交叉评论
- Critic 对所有发现发起质询

**领域主导模式（2 agents）：**
- 启动 Lead 交叉评审 agent + Critic agent
- Lead 评审全部其他视角的发现
- 其余角色不参与 Round 2
- Critic 对所有发现发起质询

**对等协作模式（4 agents）：**
- 启动 PM 交叉评审 + Designer 交叉评审 + Engineer 交叉评审 + Critic，共 4 个 agent 并行
- 三方互评（每人评其他两方）
- Critic 对所有发现发起质询

**完成后 Team Lead 动作：**
1. 收集所有报告
2. 构建投票矩阵（对等协作使用完整矩阵；集中调度和领域主导使用简化版）
3. 提取独到发现和 Critic 质询
4. 写入画布 Round 2 区域

### Phase 3：Round 3 — 共识收敛（Team Lead 执行）

由编排 agent（Team Lead）直接执行，不委派给子 agent。

**按模式区分：**

**集中调度 / 领域主导：Team Lead（或 Lead）直接裁决**
1. 整理事实和 Critic 质询
2. 直接作出裁决，标注裁决依据
3. Critic 标记为"关键质询"的议题，裁决中必须逐条回应；回应不充分则归入"待定：需用户确认"
4. 对分歧项写"裁决理由"说明
5. 输出行动计划

**对等协作：投票矩阵 + 共识收敛**
1. 汇总投票矩阵（每个议题 PM/Designer/Engineer 三方的 +1/反驳）
2. 共识判定（Critic 不参与投票，仅质询）：
   - ✅ 共识：2/3 或 3/3 方同意 → 纳入行动计划
   - ⚡ 分歧：1/3 或更少方同意 → 记录各方立场和理由
   - 🚨 Critic 升级：Critic 标记为"关键质询"且三方未实质性回应的议题 → 自动归入"待定：需用户确认"，不得直接进入 P0/P1
3. 对分歧项写"建议决策者权衡"说明
4. 整理被低估的优势（Critic 提醒的）
5. 按 ROI 排序输出行动计划

**行动计划优先级：**
- P0：高价值、低成本（立即执行）
- P1：高价值、中等成本（本轮迭代）
- P2：中等价值、较高成本（下个周期规划）
- 待定：需用户确认

**成本估算要求：** 每个行动项必须包含 Engineer 的实现成本估算。

### Phase 4：交付

1. 将完整 Round 3 结果写入共享画布
2. 向用户呈现简明摘要：
   - 使用的模式及选择理由
   - 共识项和分歧项数量
   - Top 3 P0 行动项（附成本）
   - Critic 的关键洞察
3. 指引用户查看完整画布文件

## 示例

### 集中调度模式 — 项目设计评审

**用户输入：**
> 帮我做一个多视角评审，目标是 my-web-app 这个项目

**Phase 0 — 模式选择：**
> 分析任务特征：用户未指定特定领域深入需求，任务为综合项目评审。
> 选定模式：**集中调度**（默认最佳平衡）。
> 理由：综合评审无明确主导领域，集中调度可快速收敛且输出完整度最高。
>
> *等待用户确认...*

**用户确认后 → 执行 Round 1~3：**
- Round 1：并行启动 PM / Designer / Engineer 三个 Explore agent 收集事实
- Round 2：Team Lead 自行交叉验证 + 启动 Critic agent 质询
- Round 3：Team Lead 直接裁决，输出行动计划

**Phase 4 — 交付摘要：**
> **使用模式：** 集中调度
> **共识项：** 5 项 | **分歧项：** 2 项
> **Top 3 P0 行动：**
> 1. 补充缺失的 API 输入验证（成本：4h）
> 2. 统一组件库替代散落的原生元素（成本：8h）
> 3. 添加 loading 骨架屏覆盖 3 个关键页面（成本：3h）
>
> 完整画布文件：`reviews/my-web-app-review.md`

## 核心设计原则

1. **模式决定执行路径** — 不同模式的 agent 数量、prompt 内容、画布结构有实质差异
2. **事实先于观点** — Round 1 纯事实收集，观点仅在 Round 2 引入
3. **对等尊重** — 无论哪种模式，Critic 对所有角色一视同仁
4. **结构化分歧** — 每个反驳必须附带理由，不接受无依据的意见
5. **强制收敛** — Round 3 必须产出共识和行动计划，不允许无限辩论
6. **Critic 是催化剂** — Critic 不写自己的发现，只挑战他人的发现；不投票，但对关键质询有升级权
7. **成本感知** — Engineer 必须为每个改进建议估算实现成本

## 资源文件

- `references/role-prompts.md` — 各角色各轮次的详细 prompt 模板，按模式分别说明
- `references/mode-selection.md` — 模式选择指南：决策树、混合模式、中途切换
- `references/round-protocols.md` — 轮次协议详解：每轮按模式说明 agent 数量、类型、并行策略、完成标志
- `assets/canvas-template.md` — 共享画布模板，复制到项目的 `reviews/` 目录后替换变量使用
