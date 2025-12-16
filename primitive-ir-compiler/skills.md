---
name: primitive-ir-compiler
description: >
  Convert high-entropy natural language into low-entropy Primitive IR objects
  (Entity, Resource, Event, Obligation, Action, Policy, Ledger) with metadata.
  Designed for chaining into Structure Cards (Decision, Action Planner, Reflection).
---
inputs:
  - id: raw_text
    type: string
    required: true
    description: High-entropy natural language to parse.
  - id: language
    type: string
    required: false
    default: en
    description: ISO language code of input (en, zh-CN, etc.).
  - id: domain
    type: string
    required: false
    default: general
    description: Domain hints (general, finance, law, education...).
  - id: time_zone
    type: string
    required: false
    default: America/New_York
    description: IANA time zone for normalizing dates/times.
  - id: reference_time
    type: string
    required: false
    description: ISO-8601 timestamp to resolve relative times like “tomorrow”.
  - id: require_types
    type: list[string]
    required: false
    description: If present, only emit these primitive types (e.g., [”Entity”,”Action”]).
  - id: allow_partial
    type: boolean
    required: false
    default: true
    description: Emit best-effort primitives even if some fields are missing (raise entropy).
  - id: vocab_seeds
    type: object
    required: false
    description: Controlled vocab seeds (e.g., tickers, units, indicators) to anchor fields.

outputs:
  - id: primitives
    type: list[object]
    description: List of Primitive IR objects (schemas below).
  - id: clarifications
    type: list[string]
    description: Natural-language questions/assumptions made to reduce entropy.
  - id: meta
    type: object
    description: Extraction diagnostics {entropy, confidence, trace_id, notes}.

# -----------------------
# Primitive IR Schemas
# -----------------------
schemas:
  entity:
    type: object
    required: [type, name]
    properties:
      type: {const: Entity}
      name: {type: string}
      kind: {type: string, enum_hint: [person, organization, equity, account, dataset, location]}
      attrs: {type: object}
      meta: {$ref: “#/schemas/meta”}
  resource:
    type: object
    required: [type, name, unit]
    properties:
      type: {const: Resource}
      name: {type: string}
      unit: {type: string}      # ISO-4217 for currency if applicable
      quantity: {type: number}
      meta: {$ref: “#/schemas/meta”}
  event:
    type: object
    required: [type, verb, time]
    properties:
      type: {const: Event}
      verb: {type: string}
      time: {type: string}      # ISO-8601 with time zone
      actors: {type: list[string]}
      impact_hint: {type: string}
      meta: {$ref: “#/schemas/meta”}
  obligation:
    type: object
    required: [type, duty, holder]
    properties:
      type: {const: Obligation}
      duty: {type: string}
      holder: {type: string}
      condition: {type: string}
      meta: {$ref: “#/schemas/meta”}
  action:
    type: object
    required: [type, verb, actor]
    properties:
      type: {const: Action}
      verb: {type: string}
      actor: {type: string}
      object: {type: object}    # usually an Entity
      params: {type: object}
      meta: {$ref: “#/schemas/meta”}
  policy:
    type: object
    required: [type, rule]
    properties:
      type: {const: Policy}
      rule: {type: string}
      scope: {type: string}
      conditions: {type: list[object]}
      meta: {$ref: “#/schemas/meta”}
  ledger:
    type: object
    required: [type, entry, timestamp]
    properties:
      type: {const: Ledger}
      entry: {type: string}
      timestamp: {type: string} # ISO-8601 with time zone
      ref: {type: object}
      meta: {$ref: “#/schemas/meta”}
  meta:
    type: object
    required: [id, confidence, entropy]
    properties:
      id: {type: string}        # PRM-<Type>-<snowflake>
      lang: {type: string}
      source: {type: string}
      confidence: {type: number}# 0-1
      entropy: {type: number}   # 0-1 (higher = more ambiguous)
      trace: {type: list[string]}
      version: {type: number}

# -----------------------
# Skill Logic (Deterministic Guidance)
# -----------------------
skill_logic: |
  # Parsing & Normalization Pipeline
  1) Span: Segment the raw_text into candidate phrases (entities, numbers, dates, verbs).
  2) Type: Map each candidate to one of the seven primitive types (favor schedulability).
  3) Fill: Normalize fields:
     - Dates → ISO-8601 with time_zone (use reference_time for relative phrases).
     - Currency → ISO-4217 (USD, EUR...).
     - Tickers/indicators → resolve via vocab_seeds if provided.
  4) Score: For each primitive, set:
     - confidence (0-1) from extraction certainty.
     - entropy (0-1) from ambiguity/omissions; >0.4 triggers clarifications.
  5) Filter: If require_types is set, include only those types.
  6) Partial: If allow_partial=true, emit best-effort primitives with higher entropy.
  7) Clarify: Emit clarifications/questions for any ambiguity (“How many shares?”).
  8) Output: Return {primitives[], clarifications[], meta{entropy, confidence, trace_id}}.

  # Golden Rules
  - Do not invent facts. Prefer higher entropy + a clarification over guessing.
  - Keep one semantic axis per primitive. Split multi-intent sentences into multiple primitives.
  - Favor machine-readability (deterministic keys, controlled vocab).
  - Make outputs chainable into Decision / Action Planner / Reflection Cards.

validation:
  - Ensure every primitive matches its schema; drop ill-formed ones with a note.
  - Use ISO-8601 timestamps and explicit time_zone.
  - Currency amounts include both unit and quantity if stated; otherwise raise entropy.
  - Emit at least one clarification if any primitive’s entropy > 0.4.

# -----------------------
# Examples
# -----------------------
examples:
  - title: Finance — earnings pre-entry (English)
    input:
      raw_text: I want to buy a bit of Apple before earnings, maybe this Friday.
      language: en
      domain: finance
      time_zone: America/New_York
      reference_time: “2025-10-28T09:00:00-04:00”
      vocab_seeds:
        tickers: {Apple: AAPL}
    output: |
      primitives:
        - type: Entity
          name: AAPL
          kind: equity
          meta: {id: PRM-Entity-001, lang: en, source: raw_text, confidence: 0.95, entropy: 0.08}
        - type: Event
          verb: earnings_release
          time: “2025-10-31T16:00:00-04:00”
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
        - What exact quantity or notional size should “a bit” map to? (e.g., shares or USD)
        - Confirm earnings date/time for Apple—do you mean the upcoming Friday after the reference_time?
      meta:
        entropy: 0.42
        confidence: 0.86
        trace_id: MC01

  - title: Chinese input — explicit amounts (zh-CN)
    input:
      raw_text: 我想在财报前用两千美元在退休账户买入100股苹果。
      language: zh-CN
      domain: finance
      time_zone: America/New_York
      reference_time: “2025-10-28T09:00:00-04:00”
      vocab_seeds:
        tickers: {苹果: AAPL}
    output: |
      primitives:
        - type: Entity
          name: AAPL
          kind: equity
          meta: {id: PRM-Entity-010, lang: zh-CN, source: raw_text, confidence: 0.97, entropy: 0.06}
        - type: Resource
          name: cash
          unit: USD
          quantity: 2000
          meta: {id: PRM-Resource-011, confidence: 0.96, entropy: 0.05}
        - type: Event
          verb: 财报发布
          time: “2025-10-30T16:00:00-04:00”
          actors: [AAPL]
          meta: {id: PRM-Event-012, confidence: 0.92, entropy: 0.18}
        - type: Action
          verb: 买入
          actor: 用户
          object: {type: Entity, name: AAPL, kind: equity}
          params: {quantity: 100, account: 退休账户}
          meta: {id: PRM-Action-013, confidence: 0.94, entropy: 0.12}
      clarifications: []
      meta:
        entropy: 0.16
        confidence: 0.93
        trace_id: MC01

  - title: Policy & Obligation extraction
    input:
      raw_text: If price closes below the 20-day MA, trim half the position; settlement must be T+1.
      language: en
      domain: finance
    output: |
      primitives:
        - type: Policy
          rule: Close < MA20 → trim 50% position
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
      meta:
        entropy: 0.20
        confidence: 0.91
        trace_id: MC02

# -----------------------
# Postconditions (for chaining)
# -----------------------
postconditions:
  - primitives.length >= 1
  - meta.entropy <= 0.6
  - If any primitive.meta.entropy > 0.4, clarifications.length >= 1

notes:
  - This skill is the “Perception Layer”: understand before the Structure Cards act.
  - Pair with Decision / Action Planner skills. Output is designed to be machine-readable and auditable.