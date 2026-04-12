<!-- Rendering rule: Do not narrate routing or file-loading decisions to the user — render the content below directly, without meta commentary about which intent was classified or which file was opened. -->

# /prism — quick reference

## Usage

- `/prism` — show what Prism is and how to use it (loads `about.md`).
- `/prism {framework name}` — create a new instrument via a short
  interview. Example: `/prism CVSS`, `/prism 칸반 방식`, `/prism PASTA
  threat modeling`. If the request is specific, generation starts
  immediately; if it's ambiguous, expect up to 3 clarifying questions.
- `/prism help` — this page.

## For catalog browsing

Use `/prism search` for queries like "what lenses exist for security?"
or "show me frames for product strategy".

## For loading instruments into subagents

Use `/prism fetch <instrument-names>` to get a ready-made instruction
block for subagent prompts. Example: `/prism fetch stride owasp-top10`.

## For batch generation

That's outside this skill: `python3 scripts/prism_batch.py` handles
bulk generation from a prompt template.

## 5 classes at a glance

- **Lens (렌즈)** — executable procedure with input, output template, and
  confidence signal. All four criteria required.
- **Frame (분류 프레임)** — taxonomy or classification matrix: categories
  plus discriminating criteria, no "next step" procedure.
- **Model (이론 모델)** — theoretical or predictive model with variables,
  relationships, and predictions, but no built-in application steps.
- **Stance (비판적 관점)** — interpretive commitment expressed as guiding
  questions about what is worth looking for.
- **Heuristic (원리 / 격언)** — single-rule aphorism used as a check
  inside a lens or as a sanity gate before synthesis.

## Storage layers

Instruments live in 3 layers: bundle (read-only), global, project.
Precedence: project > global > bundle.
