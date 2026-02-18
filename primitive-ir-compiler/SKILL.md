---
name: primitive-ir-compiler
description: >
  Convert high-entropy natural language into low-entropy Primitive IR objects
  (Entity, Resource, Event, Obligation, Action, Policy, Ledger) with metadata.
  Designed for chaining into Structure Cards (Decision, Action Planner, Reflection).
---

# Primitive IR Compiler

将高熵自然语言转换为低熵的 Primitive IR 对象，支持 7 种原语类型，可链式接入决策卡、行动规划等下游 skill。

## 输入参数

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `raw_text` | string | 是 | — | 待解析的自然语言文本 |
| `language` | string | 否 | `en` | ISO 语言代码（en, zh-CN 等） |
| `domain` | string | 否 | `general` | 领域提示（general, finance, law, education...） |
| `time_zone` | string | 否 | `America/New_York` | IANA 时区，用于日期/时间标准化 |
| `reference_time` | string | 否 | — | ISO-8601 时间戳，解析"明天"等相对时间 |
| `require_types` | list[string] | 否 | — | 仅输出指定类型（如 `["Entity","Action"]`） |
| `allow_partial` | boolean | 否 | `true` | 允许输出不完整但尽力而为的原语（熵值较高） |
| `vocab_seeds` | object | 否 | — | 受控词汇种子（如 tickers、units），用于锚定字段 |

## 输出

| 字段 | 类型 | 说明 |
|------|------|------|
| `primitives` | list[object] | Primitive IR 对象列表（schema 见下方） |
| `clarifications` | list[string] | 自然语言形式的澄清问题/假设 |
| `meta` | object | 提取诊断信息 `{entropy, confidence, trace_id, notes}` |

## 原语 Schema（7 种类型）

### Entity

```yaml
type: {const: Entity}
name: {type: string}           # 必填
kind: {type: string}           # person, organization, equity, account, dataset, location
attrs: {type: object}
meta: {$ref: meta}
```

### Resource

```yaml
type: {const: Resource}
name: {type: string}           # 必填
unit: {type: string}           # 必填，货币用 ISO-4217
quantity: {type: number}
meta: {$ref: meta}
```

### Event

```yaml
type: {const: Event}
verb: {type: string}           # 必填
time: {type: string}           # 必填，ISO-8601 含时区
actors: {type: list[string]}
impact_hint: {type: string}
meta: {$ref: meta}
```

### Obligation

```yaml
type: {const: Obligation}
duty: {type: string}           # 必填
holder: {type: string}         # 必填
condition: {type: string}
meta: {$ref: meta}
```

### Action

```yaml
type: {const: Action}
verb: {type: string}           # 必填
actor: {type: string}          # 必填
object: {type: object}         # 通常是 Entity
params: {type: object}
meta: {$ref: meta}
```

### Policy

```yaml
type: {const: Policy}
rule: {type: string}           # 必填
scope: {type: string}
conditions: {type: list[object]}
meta: {$ref: meta}
```

### Ledger

```yaml
type: {const: Ledger}
entry: {type: string}          # 必填
timestamp: {type: string}      # 必填，ISO-8601 含时区
ref: {type: object}
meta: {$ref: meta}
```

### Meta（共享）

```yaml
id: {type: string}             # PRM-<Type>-<snowflake>
lang: {type: string}
source: {type: string}
confidence: {type: number}     # 0-1
entropy: {type: number}        # 0-1，越高越模糊
trace: {type: list[string]}
version: {type: number}
```

## 解析流水线

1. **Span** — 将 raw_text 分割为候选短语（实体、数字、日期、动词）
2. **Type** — 将候选映射到 7 种原语类型（优先可调度性）
3. **Fill** — 标准化字段：日期 → ISO-8601 含时区；货币 → ISO-4217；ticker/指标 → 通过 vocab_seeds 解析
4. **Score** — 设置 confidence（0-1）和 entropy（0-1）；entropy > 0.4 触发澄清
5. **Filter** — 若设置了 require_types，只保留指定类型
6. **Partial** — 若 allow_partial=true，输出尽力而为的原语（熵值较高）
7. **Clarify** — 为任何模糊点输出澄清问题
8. **Output** — 返回 `{primitives[], clarifications[], meta{}}`

## 核心规则

- 不捏造事实。宁可提高熵值 + 附带澄清问题，也不猜测
- 每个原语保持单一语义轴。多意图句子拆分为多个原语
- 优先机器可读性（确定性键名、受控词汇）
- 输出设计为可链式接入 Decision / Action Planner / Reflection Card

## 校验规则

- 每个原语必须匹配其 schema；不合格的丢弃并附注
- 时间戳使用 ISO-8601 并包含明确时区
- 货币金额同时包含 unit 和 quantity（缺失则提高熵值）
- 若任何原语 entropy > 0.4，至少输出一条澄清

## 后置条件（链式调用）

- `primitives.length >= 1`
- `meta.entropy <= 0.6`
- 若任何 `primitive.meta.entropy > 0.4`，则 `clarifications.length >= 1`

## 示例

### 示例 1：金融 — 财报前买入（英文）

**输入：**
```
raw_text: "I want to buy a bit of Apple before earnings, maybe this Friday."
language: en
domain: finance
time_zone: America/New_York
reference_time: "2025-10-28T09:00:00-04:00"
vocab_seeds: {tickers: {Apple: AAPL}}
```

**输出：**
```yaml
primitives:
  - type: Entity
    name: AAPL
    kind: equity
    meta: {id: PRM-Entity-001, lang: en, confidence: 0.95, entropy: 0.08}
  - type: Event
    verb: earnings_release
    time: "2025-10-31T16:00:00-04:00"
    actors: [AAPL]
    impact_hint: volatility_up
    meta: {id: PRM-Event-002, confidence: 0.88, entropy: 0.22}
  - type: Action
    verb: buy
    actor: user
    object: {type: Entity, name: AAPL, kind: equity}
    params: {quantity: null}
    meta: {id: PRM-Action-003, confidence: 0.72, entropy: 0.55}
clarifications:
  - 'What exact quantity should "a bit" map to? (shares or USD)'
  - 'Confirm earnings date — do you mean the upcoming Friday?'
meta: {entropy: 0.42, confidence: 0.86, trace_id: MC01}
```

### 示例 2：中文输入 — 明确金额

**输入：**
```
raw_text: "我想在财报前用两千美元在退休账户买入100股苹果。"
language: zh-CN
domain: finance
vocab_seeds: {tickers: {苹果: AAPL}}
```

**输出：**
```yaml
primitives:
  - type: Entity
    name: AAPL
    kind: equity
    meta: {id: PRM-Entity-010, lang: zh-CN, confidence: 0.97, entropy: 0.06}
  - type: Resource
    name: cash
    unit: USD
    quantity: 2000
    meta: {id: PRM-Resource-011, confidence: 0.96, entropy: 0.05}
  - type: Action
    verb: 买入
    actor: 用户
    object: {type: Entity, name: AAPL, kind: equity}
    params: {quantity: 100, account: 退休账户}
    meta: {id: PRM-Action-013, confidence: 0.94, entropy: 0.12}
clarifications: []
meta: {entropy: 0.16, confidence: 0.93, trace_id: MC01}
```

### 示例 3：策略与义务提取

**输入：**
```
raw_text: "If price closes below the 20-day MA, trim half the position; settlement must be T+1."
language: en
domain: finance
```

**输出：**
```yaml
primitives:
  - type: Policy
    rule: "Close < MA20 → trim 50% position"
    scope: target_instruments
    conditions:
      - {indicator: MA, window: 20, relation: below, target: close}
    meta: {id: PRM-Policy-020, confidence: 0.9, entropy: 0.18}
  - type: Obligation
    duty: settlement
    holder: user
    condition: T+1
    meta: {id: PRM-Obl-021, confidence: 0.88, entropy: 0.12}
clarifications: []
meta: {entropy: 0.20, confidence: 0.91, trace_id: MC02}
```

## 备注

- 本 skill 是"感知层"：先理解，再由 Structure Card 执行
- 搭配 Decision / Action Planner skill 使用。输出设计为机器可读且可审计
