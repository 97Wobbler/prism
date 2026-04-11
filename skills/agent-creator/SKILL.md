---
name: agent-creator
description: 도메인 전문가 분석 에이전트를 생성하는 스킬. "분석 에이전트 만들어줘", "보안 리뷰어 세팅", "전문가 렌즈로 평가 에이전트 생성", "expert agent", "analysis agent", "lens-based agent" 등의 요청에 트리거. 렌즈 카탈로그를 탐색하고 필요한 렌즈를 조립해 가벼운 에이전트 config을 산출한다.
---

# agent-creator

You are building a **lens-based analysis agent** — a lightweight agent that does rigorous domain-expert analysis by loading structured analytical frameworks ("lenses"), not by pretending to be a persona.

## Core principle

Persona prompts ("You are a security expert") change tone but not reasoning depth. Lenses (STRIDE, OWASP Top 10, Bloom's Taxonomy, Eisenhower Matrix, etc.) are the actual procedures experts use. Load the right lenses into an agent and it reasons structurally, not vibes-first.

Your job is to pick or build the right lenses and wire them into a minimal agent config.

## Workflow

### Step 1 — Clarify domain and purpose

If the user's request already names the domain and the analysis target clearly, skip this. Otherwise ask two things:

1. **What will this agent analyze?** (codebase? curriculum document? product spec? incident report?)
2. **What decision does the output support?** (ship/no-ship? revise? prioritize? escalate?)

Keep it to two questions max. Don't interrogate.

### Step 2 — Triage the catalog

Read `lenses/catalog.yml` — this is the index of every lens available. Do NOT read individual lens files yet. That's the whole point: catalog stays cheap, only selected lenses get loaded.

For the user's domain, assemble a candidate set:

- **Domain-specific lenses** first (e.g., `security/` for a security review)
- **General lenses** second — many analyses benefit from `first-principles`, `rumsfeld-matrix`, `socratic-method`, etc. as a meta-layer
- **Reject lenses** whose `one_liner` clearly doesn't fit. Be decisive — 3-6 lenses is a good target. More than 8 means the agent will dilute.

Present the candidate set to the user with a one-line rationale per lens. Let them add/remove before you proceed.

### Step 3 — Fill gaps (generate missing lenses if needed)

If the user's domain lacks a lens that clearly should exist (e.g., they want a contract-review agent but `lenses/legal/` is empty), offer to generate it. Follow the **Lens file format** below. Key rule: the `Analytical Procedure` section must be concrete enough that someone unfamiliar with the framework can execute it step by step. Vague instructions like "assess quality" are a failure. "For each endpoint, verify whether an authentication mechanism is declared and how it handles missing tokens" is the target.

After creating a new lens, register it in `catalog.yml`.

### Step 4 — Generate the agent config

Write a YAML config to `.claude/agents/<agent-name>.yml` in the current project (not inside the plugin). Follow the **Agent config format** below. Rules:

- `persona` is ONE sentence. Role + primary goal. No adjectives like "meticulous" or "rigorous" — those are vibes.
- Each lens gets a `usage` field: `always`, `when-relevant`, or `on-request`.
- The `analysis_workflow` is fixed: triage → analyze → synthesize. Don't invent new steps unless the user asks.
- `output_format` matches the fixed template so outputs are consistent across agents.

### Step 5 — Review with user

Show the user:

1. The final lens list with one-line summaries (re-state so they can sanity-check).
2. The agent config file path.
3. One example invocation (how to actually run the agent on a sample input).

Ask if they want to tune anything before finalizing.

## Lens file format

Every lens file lives under `lenses/<domain>/<lens-name>.md` and follows this exact structure:

```markdown
---
name: <framework name>
domain: <general | security | education | ...>
source: <author, institution, year>
best_for: <one line — when this lens is most useful>
---

# <framework name>

## Overview
<2-3 sentences: what it is and why practitioners use it>

## Analytical Procedure
<numbered, step-by-step instructions. Each step must be directly executable.
Bad: "Assess the threat level"
Good: "For each data flow that crosses a trust boundary, list what identity information is attached and whether the receiving component re-validates it">

## Evaluation Criteria
<checklist, rubric, or scoring matrix the analyst fills in>

## Red Flags
<concrete patterns that, when observed, indicate a problem>

## Output Format
<template for structured output — headers, fields, scales>
```

The `Analytical Procedure` is the load-bearing section. If it's vague, the whole lens is vague.

## Agent config format

```yaml
name: <agent-name>
persona: >
  <one sentence: role + primary goal>

lenses:
  - path: lenses/<domain>/<lens>.md
    usage: always        # apply unconditionally
  - path: lenses/<domain>/<lens>.md
    usage: when-relevant # apply if triage flags it
  - path: lenses/general/<lens>.md
    usage: on-request    # apply only when user explicitly asks

analysis_workflow:
  - step: triage
    description: >
      Read catalog.yml and the input. Decide which lenses apply.
      `always` lenses run unconditionally. For `when-relevant` lenses,
      state whether they apply and why (or why not).

  - step: analyze
    description: >
      Apply each selected lens in order. Follow its Analytical Procedure
      step by step. Record findings in the lens's Output Format.
      Tag each finding with confidence: high | medium | low.

  - step: synthesize
    description: >
      Cross-check findings across lenses. Issues flagged by multiple
      lenses are high-confidence. Where lenses disagree, describe the
      tension explicitly. Produce a prioritized recommendation list.

output_format: |
  ## Triage
  <lenses selected + reasoning>

  ## Analysis
  ### <lens 1 name>
  <output matching lens Output Format>

  ### <lens 2 name>
  <output>

  ## Synthesis
  ### High-Priority Findings
  <issues confirmed by multiple lenses>

  ### Tensions
  <disagreements between lenses and what they mean>

  ### Recommendations
  <prioritized action items>
```

## What NOT to do

- Don't write persona prompts full of adjectives. The lens does the work.
- Don't dump every lens into one agent. Triage is there for a reason.
- Don't create lenses that are just renamed checklists. A lens must encode a real analytical procedure experts recognize.
- Don't skip the `catalog.yml` update when adding a new lens. Silent lenses are unusable.
