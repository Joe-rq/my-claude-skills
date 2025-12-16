---
name: decision-card
description: Turn a clarified goal into a structured, scored decision path with a recommended option, rationale, and a schedulable next step.
---
inputs:
  - id: goal_statement
    type: string
    required: true
    description: The clarified goal from the Intention Clarifier card.
  - id: context_factors
    type: object
    required: false
    description: Parameters that affect the decision (resources, time, constraints, risks, preferences, weights).
outputs:
  - id: decision_path
    type: markdown
    description: A structured list of options with steps, estimates, and milestones.
  - id: decision_rationale
    type: object
    description: Scores, trade-offs, and assumptions used to choose the option.
  - id: chosen_option
    type: object
    description: The recommended option with a concrete next step.
  - id: next_card_suggestion
    type: string
    description: Suggested next Structure Card to execute (e.g., action-planner, reflection-card).
examples:
  - goal_statement: "Establish a 3-week learning plan covering LLM fundamentals and prompt engineering."
    context_factors:
      available_time: "1h/day"
      resources: ["YouTube", "Coursera", "ChatGPT"]
      constraints: ["budget<$100", "3 weeks"]
      preference: ["structured pacing", "measurable progress"]
      weights: {goal_alignment:0.4, feasibility:0.3, risk:0.2, feedback_loop:0.1}
    decision_path: |
      - Option A: Coursera structured course + daily 1h
        - Milestones: Week1 LLM basics; Week2 Prompting; Week3 Mini-project
        - Estimate: cost $49, time 21h
      - Option B: YouTube playlist + daily summaries + weekly mini-project
        - Estimate: cost $0, time 21h
      - Option C: 1:1 mentor remote coaching
        - Estimate: cost >$200, time 15h
    decision_rationale:
      scores:
        A: {goal_alignment:0.9, feasibility:0.8, risk:0.2, feedback_loop:0.7, total:0.79}
        B: {goal_alignment:0.6, feasibility:0.9, risk:0.3, feedback_loop:0.5, total:0.66}
        C: {goal_alignment:0.8, feasibility:0.6, risk:0.4, feedback_loop:0.8, total:0.70}
      tradeoffs: "A offers pacing & assessment; B is free but demands high self-discipline; C exceeds budget."
      assumptions: ["User can allocate 1h/day", "Budget cap $100", "Willing to take quizzes"]
    chosen_option:
      id: "A"
      label: "Coursera structured course + daily 1h"
      next_step: "Enroll tonight; block 1h/day for 3 weeks; create weekly milestone checklist."
    next_card_suggestion: "action-planner"
