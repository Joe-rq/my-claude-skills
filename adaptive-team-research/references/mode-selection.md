# Mode Selection Guide

## Overview

This skill supports three collaboration modes. Choose the mode that best fits
the task characteristics to optimize for speed, depth, or innovation.

## Quick Decision Matrix

| Task Characteristic | Recommended Mode | Reasoning |
|--------------------|------------------|-----------|
| Time pressure / Deadline | Centralized Dispatch | Fastest convergence |
| Product decision needed | Centralized Dispatch | Clear ownership, actionable |
| Experience optimization | Domain-Led (UX) | Deep UX expertise |
| Technical architecture deep-dive | Domain-Led (Engineer) | Deep technical expertise |
| Strategy/planning | Domain-Led (PM) | Deep product expertise |
| Exploring unknown territory | Peer Collaboration | Maximum creativity |
| Innovation required | Peer Collaboration | Cross-pollination |
| Research / Discovery | Peer Collaboration | Multiple angles |
| Unsure / Mixed | Centralized Dispatch | Best default balance |

## Centralized Dispatch Mode

### When to Use

- **Time-sensitive decisions**: Deadlines are tight, need fast results
- **Product decisions**: Need clear ownership and actionable output
- **Task uncertainty**: When unsure which mode fits best
- **Default choice**: When no strong indicators for other modes

### Characteristics

**Structure:**
```
User Request
    ↓
Team-Lead (creates team, coordinates)
    ↓
┌───────────┬───────────┬───────────┐
│ PM        │ Designer  │ Engineer  │ (parallel execution)
└───────────┴───────────┴───────────┘
    ↓
Team-Lead (synthesizes, decides)
    ↓
Final Report
```

**Pros:**
- Fastest execution time
- Highest output completeness
- Clear accountability
- Minimal coordination overhead

**Cons:**
- Less cross-pollination
- May miss creative solutions
- Single point of decision bias

**Best for:** Product decisions, time-sensitive tasks, executive summaries

## Domain-Led Mode

### When to Use

- **Experience optimization**: Deep UX analysis needed
- **Technical deep-dive**: Complex architecture assessment
- **Strategy work**: Product positioning, market analysis
- **Specialist expertise**: One dimension requires exceptional depth

### Characteristics

**Structure:**
```
User Request
    ↓
Domain Expert (PM/Designer/Engineer) leads
    ↓
┌─────────────────────────────────────┐
│ Lead coordinates supporting roles   │
│ - Lead defines direction            │
│ - Others provide input to lead      │
└─────────────────────────────────────┘
    ↓
Domain Expert synthesizes
    ↓
Final Report (domain-deep)
```

**Pros:**
- Exceptional depth in chosen domain
- Leverages specialist expertise
- Focused output

**Cons:**
- May under-weight other perspectives
- Requires clear domain identification
- Less balanced output

**Best for:** UX reviews, architecture assessments, strategy documents

### Sub-variants

**UX-Led**: Designer leads, PM and Engineer support
- Use when: User experience is primary concern
- Output: Deep UX analysis with product and technical context

**Engineer-Led**: Engineer leads, PM and Designer support
- Use when: Technical feasibility is primary concern
- Output: Deep technical analysis with product and UX context

**PM-Led**: PM leads, Designer and Engineer support
- Use when: Product strategy is primary concern
- Output: Deep product analysis with UX and technical context

## Peer Collaboration Mode

### When to Use

- **Exploratory tasks**: Unknown territory, need broad exploration
- **Innovation required**: Need creative solutions
- **Research tasks**: Multiple angles needed
- **Complex tradeoffs**: No clear right answer

### Characteristics

**Structure:**
```
User Request
    ↓
Shared Canvas (all roles read/write)
    ↓
┌─────────┬──────────┬──────────┐
│ PM      │ Designer │ Engineer │ (parallel, equal)
└────┬────┴────┬─────┴────┬─────┘
     └─────────┼──────────┘
               ↓
         Critic (challenges all)
               ↓
     Consensus or Vote
               ↓
         Final Report
```

**Pros:**
- Maximum creativity
- Cross-pollination of ideas
- Balanced perspectives
- Discovers overlooked insights

**Cons:**
- Slower convergence
- Requires structured debate protocol
- May produce conflicting recommendations

**Best for:** Research, innovation, complex problem-solving, discovery

## Mode Selection Decision Tree

```
Is there a clear domain that needs exceptional depth?
├── YES → Domain-Led Mode
│         └── Which domain?
│               ├── UX → UX-Led
│               ├── Technical → Engineer-Led
│               └── Strategy → PM-Led
│
└── NO → Is time a critical constraint?
          ├── YES → Centralized Dispatch Mode
          │
          └── NO → Is exploration/innovation the goal?
                    ├── YES → Peer Collaboration Mode
                    └── NO  → Centralized Dispatch Mode (default)
```

## Hybrid Approaches

### Start with Peer, Converge with Centralized

For complex tasks:
1. Start with Peer Collaboration for exploration (Round 1-2)
2. Switch to Centralized Dispatch for decision (Round 3)

Benefits: Creative exploration + fast decision-making

### Domain-Led with Peer Elements

For deep analysis tasks:
1. Domain expert leads overall direction
2. Use peer-style debate for specific sub-questions
3. Domain expert synthesizes

Benefits: Depth + balanced input on key questions

## Switching Modes Mid-Task

It's acceptable to switch modes if the task characteristics change:

- **Peer → Centralized**: When exploration is done, decision needed
- **Centralized → Domain-Led**: When depth needed in specific area
- **Domain-Led → Peer**: When tradeoffs emerge needing multi-angle input

Document mode switches in the shared canvas for transparency.
