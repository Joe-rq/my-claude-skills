---
name: multi-perspective-review
description: "This skill orchestrates a multi-agent design review using the Peer Collaboration + Shared Canvas pattern. It should be used when the user requests a comprehensive project review, design audit, architecture assessment, or any task requiring structured multi-perspective analysis. Triggers include phrases like review this project, audit the design, multi-angle analysis, 多视角审查, 设计评审, 架构评估. The skill coordinates PM, Designer, Engineer, and Critic agents through three structured rounds: fact-gathering, cross-debate, and consensus convergence, producing an actionable prioritized improvement plan."
---

# Multi-Perspective Review

## Overview

Orchestrate a structured multi-agent design review that produces actionable insights through
peer collaboration and structured debate. Four roles (PM, Designer, Engineer, Critic) work
through three rounds (Facts → Debate → Consensus) using a shared canvas document as the
coordination mechanism.

## When to Use

- Project design reviews or architecture assessments
- Product strategy analysis requiring multiple viewpoints
- Code quality audits needing cross-functional perspective
- Any task where the user explicitly requests multi-angle or team-based analysis

## Workflow

### Phase 0: Setup

1. Identify the project directory and review target from user context
2. Copy the canvas template from `assets/canvas-template.md` to the project's working directory
   (e.g., `.sisyphus/review-canvas.md` or a location the user specifies)
3. Replace template variables: `{{PROJECT_NAME}}`, `{{DATE}}`, `{{REVIEW_TARGET}}`

### Phase 1: Round 1 — Fact Gathering (Parallel)

Launch three Explore agents in parallel using `run_in_background: true`. Load role-specific
prompts from `references/role-prompts.md` and customize with project context.

**Agent configuration:**
- Agent type: `Explore` (read-only, sufficient for fact gathering)
- All three run in parallel to minimize total time
- Each agent outputs a structured fact list with source references (file:line)

**Key constraint communicated to each agent:** Only write facts. No evaluations, no suggestions,
no subjective language ("good", "bad", "should").

**On completion:**
1. Collect all three fact reports
2. Distill key facts into the shared canvas Round 1 section
3. Ensure no critical facts are lost but avoid redundancy

### Phase 2: Round 2 — Cross-Debate (Parallel)

Launch four Explore agents in parallel, all reading the shared canvas:

1. **PM cross-reviewer** — Comments on Designer and Engineer findings from product perspective
2. **Designer cross-reviewer** — Comments on PM and Engineer findings from UX perspective
3. **Engineer cross-reviewer** — Comments on PM and Designer findings from technical feasibility
   perspective, including cost estimates for each suggested improvement
4. **Critic** — Challenges weak arguments, questions missing evidence, identifies contradictions,
   and highlights overlooked strengths

**Output format for cross-reviewers:**
```
[Role → Target] Re: topic — +1 / Rebuttal, reasoning...
[Role cross-finding] Overlooked issue spanning multiple perspectives...
```

**Output format for Critic:**
```
[Critic → All] Challenge N: statement
  Evidence: ...
  Response requested from: PM / Designer / Engineer
```

**On completion:**
1. Collect all four reports
2. Build a voting matrix (topic × role → +1/rebuttal)
3. Extract unique cross-findings and Critic challenges
4. Write Round 2 summary to the shared canvas

### Phase 3: Round 3 — Consensus Convergence (Team Lead)

Performed directly by the orchestrating agent (not delegated to sub-agents).

**Steps:**
1. Build consensus from the voting matrix:
   - **Consensus (✅):** 3/4 or 4/4 agree → include in action plan
   - **Disagreement (⚡):** 2/4 or fewer agree → document all positions with reasoning
2. For each disagreement, write a "decision-maker weighing" note
3. List underestimated strengths (from Critic's challenges)
4. Produce a prioritized action plan sorted by ROI:
   - P0: High value, low cost (do immediately)
   - P1: High value, medium cost (do this sprint)
   - P2: Medium value, higher cost (plan for next cycle)
   - Pending: Needs user validation before committing

### Phase 4: Delivery

1. Write the complete Round 3 results to the shared canvas
2. Present a concise summary to the user with:
   - Number of consensus items and disagreements
   - Top 3 P0 actions
   - Key insight from the Critic
3. Point the user to the full canvas file for details

## Critical Design Principles

1. **Facts before opinions** — Round 1 is pure fact-gathering. Opinions only enter in Round 2.
2. **Peer equality** — No role outranks another. The Critic challenges all equally.
3. **Structured disagreement** — Every rebuttal must include reasoning. No unsupported opinions.
4. **Forced convergence** — Round 3 must produce consensus and disagreements. Endless debate is
   not allowed.
5. **Critic as catalyst, not judge** — The Critic does not write their own findings. They only
   challenge others' findings.
6. **Cost awareness** — Engineer must estimate implementation cost for every suggested change.

## Resources

- `references/role-prompts.md` — Detailed prompt templates for all roles across both rounds.
  Load and customize these when launching agents.
- `references/round-protocols.md` — Detailed protocols for each round including agent type
  selection, performance optimization, and completion criteria.
- `assets/canvas-template.md` — Shared canvas template to copy into the project directory.
  Replace `{{PROJECT_NAME}}`, `{{DATE}}`, and `{{REVIEW_TARGET}}` before use.
