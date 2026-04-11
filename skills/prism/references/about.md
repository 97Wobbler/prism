# About Prism

Prism is a Claude Code plugin that ships a curated catalog of **analytical
instruments** for building domain-expert agents and skills. It does NOT
generate agents or skills itself — it supplies the structured instrument
files that Claude Code's native `/agents` flow (and your own skills) can
reference. The bundle ships 657 instruments across 50 domains, all indexed
by a single `catalog.yml` for fast triage.

## The 5 classes

Every instrument belongs to exactly one of these classes. See `CLASSES.md`
for the full taxonomy, decision tree, and file-format templates.

- **Lens (렌즈)** — a structured executable procedure with a defined input,
  a fixed output format, and a confidence signal per finding. All four
  criteria are required; if any is missing, it is not a lens.
- **Frame (분류 프레임)** — a taxonomy or classification matrix: categories
  plus discriminating criteria, with no built-in "next step" procedure.
- **Model (이론 모델)** — a descriptive or predictive theoretical model
  (variables, relationships, predictions) with no built-in application
  steps until you pair it with a procedure.
- **Stance (비판적 관점)** — an interpretive commitment about what is worth
  looking for in any artifact. Stances have guiding questions, not
  procedures.
- **Heuristic (원리 / 격언)** — a single-rule aphorism used as a check
  inside a lens or as a sanity gate before synthesis.

## 3-layer storage

Prism looks up instruments across three layers and merges them with
**precedence project > global > bundle**:

- **Bundle** — read-only, shipped with the plugin (`./library/`, 657 items).
- **Global** — your personal instruments at `~/.claude/prism/library/`,
  available across all projects.
- **Project** — repo-local instruments at `./.claude/prism/library/`,
  scoped to the current project.

You never write to the bundle. Global and project layers are both
writable; closer context wins on name collisions.

## How to use Prism

- **To see what already exists for a domain** → use the `catalog-browse`
  skill (it proactively surfaces instruments whenever you're building an
  agent or skill).
- **To create a new instrument** → run `/prism {framework or methodology
  name}`. If details are ambiguous, you'll get a short interview; if the
  request is specific, generation starts immediately.
- **To batch-generate many instruments at once** → use `python3
  scripts/prism_batch.py` (the batch pipeline, not this skill).

## Out of scope

Prism does not write agent configs, does not run meta-workflows, and does
not grow the bundle beyond ~700 items. Agent assembly belongs to Claude
Code's native `/agents` flow; Prism only provides the catalog that flow
reads from.

Try `/prism CVSS` or `/prism 칸반 방식` to see the interview in action.
