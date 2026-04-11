---
name: agent-creator
description: 도메인 전문가 분석 에이전트를 생성하는 스킬. "분석 에이전트 만들어줘", "보안 리뷰어 세팅", "전문가 렌즈로 평가 에이전트 생성", "expert agent", "analysis agent", "lens-based agent" 등의 요청에 트리거. 카탈로그를 탐색하고 필요한 렌즈·프레임·모델·스탠스·휴리스틱을 조립해 가벼운 에이전트 config을 산출한다.
---

# agent-creator

You are building a **framework-composing analysis agent** — a lightweight
agent that does rigorous domain-expert analysis by loading structured
frameworks, not by pretending to be a persona.

## Core principle

Persona prompts ("You are a security expert") change tone but not
reasoning depth. Structured frameworks (STRIDE, OWASP Top 10, Cynefin,
Prospect Theory, Occam's Razor, Bloom's Taxonomy, …) are the actual
instruments experts use. Load the right ones into an agent and it reasons
structurally, not vibes-first.

**v0.2 introduces 5 classes of framework.** Not every useful framework is
a "lens" in the strict sense. Your job is to pick the right *class* for
each framework you add, compose them into a minimal agent config, and
wire them into the appropriate workflow steps.

See `CLASSES.md` at the repo root for the full taxonomy and file formats.
Brief version:

- **Lens (렌즈)** — structured executable analytical procedure. Applied
  in the `analyze` step. E.g., STRIDE, OWASP Top 10, CVSS, Bloom's.
- **Frame (분류 프레임)** — taxonomy / classification matrix. Applied in
  the `classify` step to characterize the input before lenses run.
  E.g., Cynefin, Kano Model.
- **Model (이론 모델)** — descriptive / predictive theoretical model.
  Applied in the `model-interpret` step by mapping input onto variables
  and reading off predictions. E.g., Porter's Five Forces, Prospect
  Theory.
- **Stance (비판적 관점)** — interpretive position. Applied in the
  `critical-read` step to surface what other frameworks miss. E.g.,
  Marxist criticism, Foucauldian power/knowledge.
- **Heuristic (원리 / 격언)** — single rule of thumb. Bundled and
  applied in the `sanity-gate` step as a final check. E.g., Occam's,
  Hanlon's, Chesterton's Fence.

## Workflow

### Step 1 — Clarify domain and purpose

If the user's request already names the domain and the analysis target
clearly, skip this. Otherwise ask two things:

1. **What will this agent analyze?** (codebase? curriculum document?
   product spec? incident report? industry landscape?)
2. **What decision does the output support?** (ship/no-ship? revise?
   prioritize? escalate? enter market?)

Keep it to two questions max. Don't interrogate.

### Step 2 — Triage the catalog

Read the top-level `catalog.yml` — this is the index of every item
available, grouped by class. Do NOT read individual files yet. That's
the whole point: the catalog stays cheap, only selected items get
loaded.

For the user's domain, assemble a candidate set by walking the catalog
in this order:

1. **Domain-specific lenses** (e.g., `security/` for a security review)
   — these are the load-bearing instruments.
2. **Relevant frames** — does the input need to be classified or sorted
   first? Cynefin to pick a response mode; Kano to sort features by
   satisfaction curve; a domain-specific taxonomy if one exists.
3. **Relevant models** — is there a theoretical model whose variables
   the input can be mapped onto, producing predictions? Porter's Five
   Forces for an industry, Prospect Theory for a decision under risk,
   etc.
4. **Relevant stances** — will the agent benefit from an interpretive
   critical read? (Most engineering / analytical agents won't. Stances
   are usually right when the artifact is cultural, institutional, or
   discursive.)
5. **General lenses** as a meta-layer — `first-principles`,
   `rumsfeld-matrix`, `socratic-method`, etc.
6. **Heuristic bundles** — usually `library/heuristics/general.md` as a sanity
   gate, plus any domain-specific bundle if one exists.
7. **Reject items** whose `one_liner` clearly doesn't fit. Be decisive:
   3–6 primary items (lenses + frames + models) is a good target. More
   than 8 primary items means the agent will dilute.

Present the candidate set to the user **grouped by class**, with a
one-line rationale per item. Let them add/remove before you proceed.

### Step 3 — Fill gaps (generate missing items if needed)

If the user's domain lacks items that clearly should exist, offer to
generate them. **Pick the right class first.**

- The source framework has a procedure, an output template, and a
  confidence signal? → write it as a **lens** in `library/lenses/<domain>/`.
- The source is just a taxonomy with categories and classification
  criteria? → write it as a **frame** in `library/frames/<domain>/`.
- The source is a theoretical model (variables, relationships,
  predictions) without an "Analytical Procedure" of its own? → write
  it as a **model** in `library/models/<domain>/`, and include an Application
  Procedure that walks input → variables → predictions.
- The source is an interpretive commitment about what is worth looking
  for? → write it as a **stance** in `library/stances/<domain>/`. Do **not**
  convert stances into lenses.
- The source is a single-rule aphorism? → add it to an existing
  `library/heuristics/<domain>.md` bundle, or create one if none exists for
  that domain.

**Converting a frame, model, or heuristic into a lens** is a bigger
commitment (see CLASSES.md § "Converting other classes to lenses"). It
requires writing a procedure, output format, and confidence signal
around the source. **Ask the user before doing this** — they may prefer
the lighter-weight original class.

File format rules (see CLASSES.md for full templates):

- **Lens**: the `Analytical Procedure` is load-bearing and must be
  concrete enough that someone unfamiliar with the framework can
  execute it step by step. Vague instructions like "assess quality"
  are a failure. "For each endpoint, verify whether an authentication
  mechanism is declared and how it handles missing tokens" is the
  target.
- **Frame**: the `Classification Procedure` must give a reader a
  concrete way to decide which category an instance belongs in.
  Categories without discriminating criteria are not usable.
- **Model**: the `Application Procedure` is load-bearing. Mapping
  input onto model variables should be step-by-step, not hand-wavy.
  Always include `Boundary Conditions`.
- **Stance**: the `Guiding Questions` are the working instrument.
  Abstract manifestos are not enough.
- **Heuristic**: keep each entry tight; the bite is in `When It
  Applies`, `When It Misleads`, and `How Agents Use It`.

After creating a new item, register it in `catalog.yml` under the
correct class section. For a lens that was operationalized from
another class, set `underlying_class` in both the file frontmatter
and the catalog entry.

### Step 4 — Generate the agent config

Write a YAML config to `.claude/agents/<agent-name>.yml` in the current
project (not inside the plugin). Follow the **Agent config format**
below. Rules:

- `persona` is ONE sentence. Role + primary goal. No adjectives like
  "meticulous" or "rigorous" — those are vibes.
- Each item gets a `usage` field: `always`, `when-relevant`, or
  `on-request`.
- Items are grouped by class (`lenses:`, `frames:`, `models:`,
  `stances:`, `heuristics:`). Unused class sections can be omitted
  entirely.
- The `analysis_workflow` uses the 7-step pipeline (see below).
  Optional steps should be marked `optional: true` if the agent only
  uses them conditionally, or dropped entirely if the agent never
  uses them. Simple agents keep only `triage`, `analyze`, and
  `synthesize`.
- `output_format` matches a fixed template so outputs are consistent
  across agents.

### Step 5 — Review with user

Show the user:

1. The final item list, grouped by class, with one-line summaries
   (re-state so they can sanity-check).
2. The agent config file path.
3. Which workflow steps the agent actually runs (not every agent
   needs all 7).
4. One example invocation (how to actually run the agent on a sample
   input).

Ask if they want to tune anything before finalizing.

## Agent config format

```yaml
name: <agent-name>
persona: >
  <one sentence: role + primary goal>

# Each class section is optional. Omit sections the agent doesn't use.

lenses:
  - path: library/lenses/<domain>/<lens>.md
    usage: always        # apply unconditionally
  - path: library/lenses/<domain>/<lens>.md
    usage: when-relevant # apply only if triage flags it
  - path: library/lenses/general/<lens>.md
    usage: on-request    # apply only when user explicitly asks

frames:
  - path: library/frames/<domain>/<frame>.md
    usage: when-relevant

models:
  - path: library/models/<domain>/<model>.md
    usage: when-relevant

stances:
  - path: library/stances/<domain>/<stance>.md
    usage: on-request

heuristics:
  - path: library/heuristics/general.md
    usage: always        # typical default: run as sanity gate

analysis_workflow:
  - step: triage
    description: >
      Read catalog.yml and the input. Decide which items apply.
      `always` items run unconditionally. For `when-relevant` items,
      state whether they apply and why (or why not).

  - step: classify
    optional: true
    description: >
      If any frames are selected, apply them first to characterize
      the input. The classification may change which lenses run
      (e.g., Cynefin Complex → probe-oriented lenses, not prescriptive
      ones).

  - step: analyze
    description: >
      Apply each selected lens in order. Follow its Analytical
      Procedure step by step. Record findings in the lens's Output
      Format. Tag each finding with confidence: high | medium | low.

  - step: model-interpret
    optional: true
    description: >
      If any models are selected, instantiate each one: map the input
      (or earlier findings) onto the model's variables, read off
      predictions, and check boundary conditions. Findings here are
      explanations / predictions, not lens-style vulnerabilities.

  - step: critical-read
    optional: true
    description: >
      If any stances are selected, apply them to surface what lenses
      miss: what is naturalized, whose interests are served, what is
      occluded. Multiple stances can be applied and compared.

  - step: sanity-gate
    optional: true
    description: >
      Run the selected heuristic bundles on the top findings from
      earlier steps. For each heuristic, ask whether it changes the
      framing or the priority of any finding. Heuristics never create
      findings from nothing — they amend, confirm, or demote.

  - step: synthesize
    description: >
      Cross-check findings across all applied items. Issues confirmed
      by multiple items are high-confidence. Where items disagree,
      describe the tension explicitly. Produce a prioritized
      recommendation list.

output_format: |
  ## Triage
  <items selected by class + reasoning>

  ## Classification (if applied)
  <frame categorization of the input>

  ## Analysis
  ### <lens 1 name>
  <output matching lens Output Format>

  ### <lens 2 name>
  <output>

  ## Model interpretation (if applied)
  ### <model 1 name>
  <variables mapped + predictions + boundary-condition check>

  ## Critical read (if applied)
  ### <stance 1 name>
  <findings in stance's Output Format>

  ## Sanity gate (if applied)
  <heuristic bundle results: which findings survived, which were amended>

  ## Synthesis
  ### High-Priority Findings
  <issues confirmed by multiple items>

  ### Tensions
  <disagreements between items and what they mean>

  ### Recommendations
  <prioritized action items>
```

## What NOT to do

- Don't write persona prompts full of adjectives. The frameworks do
  the work.
- Don't dump every catalog item into one agent. Triage is there for
  a reason.
- Don't force a frame, model, stance, or heuristic into a lens file
  just because the v0.1 plugin only had lenses. Pick the correct
  class and use the matching file format.
- Don't create "lenses" that are just renamed checklists. A lens
  must encode a real analytical procedure experts recognize.
- Don't skip the `catalog.yml` update when adding a new item. Silent
  items are unusable.
- Don't enable every workflow step by default. A code-review agent
  almost certainly does not need `critical-read`; a strategy agent
  almost certainly does not need `classify` via Kano if it is
  reading an incident report. Pick the steps the agent actually
  needs.
