---
name: adaptive-team-research
description: "Orchestrate adaptive multi-agent research teams with automatic mode selection and structured three-round workflow. Use when the user needs comprehensive multi-perspective analysis, design reviews, architecture assessments, or any task requiring structured team collaboration. Triggers include phrases like 'research this', 'review from multiple angles', 'analyze comprehensively', 'multi-perspective analysis', 'design review', 'architecture assessment'. The skill automatically selects the optimal collaboration mode (centralized dispatch, domain-led, or peer collaboration) based on task characteristics, then executes a structured Facts → Debate → Consensus workflow with cost-aware actionable outputs."
---

# Adaptive Team Research

## Overview

Orchestrate intelligent multi-agent research teams that adapt to task requirements.
This skill combines automatic mode selection (centralized/domain-led/peer) with
a structured three-round workflow (Facts → Debate → Consensus) to produce
high-quality, actionable insights.

## When to Use

Use this skill for:
- Comprehensive project or design reviews requiring multiple perspectives
- Product strategy analysis with conflicting viewpoints
- Architecture assessments needing cross-functional input
- Any task where the user explicitly requests multi-angle or team-based analysis
- Tasks requiring both exploration AND concrete action plans

## Workflow

### Phase 0: Mode Selection

Analyze the task and select the optimal collaboration mode:

**Centralized Dispatch Mode** (recommended for product decisions, time-sensitive tasks)
- Team-lead coordinates parallel independent research
- Fastest convergence, highest output completeness
- Use when: deadlines are tight, clear ownership is needed

**Domain-Led Mode** (recommended for experience optimization, professional depth)
- Domain expert (UX/Engineer/PM) leads with supporting roles
- Deep expertise in specific area
- Use when: one dimension requires exceptional depth

**Peer Collaboration Mode** (recommended for exploratory/innovative tasks)
- All roles equal, shared canvas, structured debate
- Maximum creativity and cross-pollination
- Use when: exploring new territory, innovation required

Default: Centralized Dispatch Mode (best balance of speed and quality)

### Phase 1: Round 1 — Fact Gathering (Parallel)

Launch three Explore agents in parallel:

1. **PM Agent** — Product perspective
   - Load prompt from `references/role-prompts.md` section "PM Fact Gatherer"
   - Research: product positioning, feature completeness, competitive landscape
   - Output: Structured fact list with source references

2. **Designer Agent** — User experience perspective
   - Load prompt from `references/role-prompts.md` section "Designer Fact Gatherer"
   - Research: UX patterns, interaction flows, accessibility, consistency
   - Output: Structured fact list with source references

3. **Engineer Agent** — Technical perspective
   - Load prompt from `references/role-prompts.md` section "Engineer Fact Gatherer"
   - Research: architecture, dependencies, security, performance
   - Output: Structured fact list with source references

**Key constraint:** Only facts, no evaluations or suggestions.
**Parallel execution:** All three agents run simultaneously.

**On completion:**
1. Collect all three fact reports
2. Distill key facts into the shared canvas Round 1 section
3. Copy canvas template from `assets/canvas-template.md` to project directory
4. Fill in PROJECT_NAME, DATE, REVIEW_TARGET variables

### Phase 2: Round 2 — Cross-Debate (Parallel)

Launch four Explore agents in parallel, all reading the shared canvas:

1. **PM Cross-Reviewer** — Reviews Designer and Engineer findings
2. **Designer Cross-Reviewer** — Reviews PM and Engineer findings from UX perspective
3. **Engineer Cross-Reviewer** — Reviews PM and Designer findings with cost estimates
4. **Critic** — Challenges weak arguments, identifies contradictions, highlights overlooked strengths

Load detailed prompts from `references/role-prompts.md` section "Round 2 Roles".

**Output format:**
```
[Role → Target] Re: topic — +1 / Rebuttal, reasoning...
[Role cross-finding] Overlooked issue spanning perspectives...
[Critic → All] Challenge N: statement
  Evidence: ...
  Response requested from: PM / Designer / Engineer
```

**On completion:**
1. Collect all four reports
2. Build voting matrix (topic × role → +1/rebuttal)
3. Extract unique cross-findings and Critic challenges
4. Write Round 2 summary to shared canvas

### Phase 3: Round 3 — Consensus Convergence (Team Lead)

Performed by the orchestrating agent (not delegated).

**Steps:**
1. Build consensus from voting matrix:
   - **Consensus (✅):** 3/4 or 4/4 agree → include in action plan
   - **Disagreement (⚡):** 2/4 or fewer agree → document all positions
2. For each disagreement, write "decision-maker weighing" note
3. List underestimated strengths (from Critic's challenges)
4. Produce prioritized action plan sorted by ROI:
   - P0: High value, low cost (immediate)
   - P1: High value, medium cost (this sprint)
   - P2: Medium value, higher cost (next cycle)
   - Pending: Needs user validation

**Cost estimation requirement:** Every action must include implementation cost estimate from Engineer.

### Phase 4: Delivery

1. Write complete Round 3 results to shared canvas
2. Present concise summary to user:
   - Number of consensus items and disagreements
   - Top 3 P0 actions with costs
   - Key insight from Critic
3. Point user to full canvas file for details

## Critical Design Principles

1. **Adaptive mode selection** — Match collaboration structure to task needs
2. **Facts before opinions** — Round 1 is pure fact-gathering
3. **Peer equality** — No role outranks another; Critic challenges all
4. **Structured disagreement** — Every rebuttal must include reasoning
5. **Forced convergence** — Round 3 must produce consensus and action plan
6. **Cost awareness** — Engineer estimates cost for every suggested change
7. **Critic as catalyst** — Critic does not write findings, only challenges others

## Resources

### references/role-prompts.md
Detailed prompt templates for all roles across both rounds. Load and customize when launching agents.

### references/mode-selection.md
Guidelines for selecting collaboration mode based on task characteristics.

### assets/canvas-template.md
Shared canvas template. Copy to project directory, replace variables before use.

## Mode Selection Reference

See `references/mode-selection.md` for detailed guidance on choosing between:
- Centralized Dispatch (speed + completeness)
- Domain-Led (depth in specific area)
- Peer Collaboration (innovation + creativity)

Quick guide:
- Product decision + time pressure → Centralized
- Experience optimization needed → Domain-Led (UX)
- Exploring unknown territory → Peer Collaboration
- Unsure → Default to Centralized
