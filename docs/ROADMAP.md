# Prism Roadmap

Prism is a Claude Code plugin that provides a catalog of analytical instruments
across 5 classes — lens, frame, model, stance, heuristic — for use when building
domain-expert agents and skills. This roadmap describes where the catalog stands
today and what each upcoming minor version changes.

## v0.1.0

- Renamed from lenslab. Same 657-instrument library across 50 domains.
- Plugin identity clarified: Prism is a catalog reference provider, not an agent
  or skill generator.
- Top-level class term unified as "instrument" (lens, frame, model, stance,
  heuristic are its five sub-classes).
- Breaking change from lenslab v0.3.0: name, plugin identifier, and package
  version all reset to 0.1.0 to mark a fresh start under the new identity.
- Existing `create-agent` slash command and `agent-creator` skill still ship in
  this release for continuity, but are deprecated and removed in v0.2.0.

## v0.2.0

Structural changes that align the plugin with its clarified identity.

- **New skills**
  - `catalog-browse` — read-only skill for exploring the instrument catalog by
    class, domain, or free-text query. Returns candidate instruments for
    whatever the caller is building (an agent, a skill, or a one-off prompt).
  - `make-instrument` — interview-style skill for creating a new instrument.
    Takes a minimal input (framework name, domain), asks targeted clarifying
    questions for any ambiguous fields, then auto-generates the instrument body
    using an LLM (Haiku-class model, following the same template pipeline
    already used for the v0.3 batch).
- **3-layer custom instrument storage**
  - Bundle (read-only): the 657 built-in instruments shipped with Prism.
  - Global: `~/.claude/prism/library/` — the user's personal instruments,
    available in any project.
  - Project: `<project>/.claude/prism/library/` — instruments scoped to a
    single repository.
  - Lookup precedence: project > global > bundle. Closer context overrides
    broader context.
- **Removed**
  - `create-agent` slash command and `agent-creator` skill. Agent and skill
    creation belong to Claude Code's native mechanisms — Prism only supplies
    the catalog they reference.
- **Restructured**
  - `agents/examples/*.yml` becomes `docs/cookbook/*.md` as composition
    recipes: narrative markdown explaining how a set of instruments can be
    combined for a given task, not an executable agent config.
- **Scripts**
  - `sync_catalog.py --source {bundle|global|project}` targets any layer of
    the 3-layer storage.

## v0.2.1

A consolidation pass on top of v0.2.0, focused on giving Prism a single
memorable user-facing entry point.

- **Router skill** — `make-instrument` has been folded into a new `/prism`
  router skill. The router itself is a thin classifier (~80 lines) that
  dispatches the request into one of three intents — about, make, help —
  and loads the corresponding reference file from `skills/prism/references/`
  (`about.md`, `make.md`, `help.md`). The full interview protocol and the
  5-class decision tree now live in `references/make.md`.
- **Catalog-browse unchanged in shape, stronger in framing** — still a
  separate skill, but its description has been rewritten to position it as
  a proactive recommender that should be consulted whenever the user is
  about to build an agent or skill, not just a passive browse tool.
- **Single memorable entry point** — `/prism` (explain), `/prism <name>`
  (create), and `/prism help` (quick reference) all flow through the same
  router, so users only have to remember one command.

## v0.3.0 (Current)

A 12-domain expansion driven by an external contributor batch. The
catalog grows from 658 to 712 instruments — about 8% — and the domain
count rises from 50 to 62. All new instrument bodies were generated
via the same `scripts/prism_batch.py` pipeline (Haiku 4.5 + class-
specific system prompts + golden few-shot examples) that authored the
original 656, ensuring tone and structural consistency with the
existing catalog.

- **12 new domains** — `systems-thinking`, `complexity-science`,
  `public-policy`, `media-studies`, `behavioral-finance`,
  `urban-planning`, `political-science`, `anthropology`,
  `neuroscience`, `applied-ethics`, `graph-theory`, `psychoanalysis`.
  Two of these (systems-thinking and complexity-science) shipped first
  in a quality-gated P1 commit because their concepts (causal loops,
  stocks/flows, archetypes, power laws, SOC, emergence, NP-hardness)
  are implicitly presupposed by other lenses.
- **5 existing-domain reinforcements** — Toulmin in `philosophy`;
  Stasis theory + Classical canons + Peirce icon-index-symbol in
  `linguistics` (rhetoric absorbed rather than given its own domain
  by deliberate decision); social psychology block (bystander effect,
  situationism, social proof, Milgram obedience) in `psychology`;
  Schelling focal points in `game-theory` (distinct from the existing
  `schelling-segregation-model` in sociology); 6 individual cognitive
  biases (availability, confirmation, anchoring, base-rate, sunk-cost,
  hindsight) in the `general` heuristics bundle.
- **Three load-bearing placement decisions** — NP-hardness lives under
  `complexity-science`, not `graph-theory` (which is reserved for the
  algorithmic layer: network analysis, spanning trees). Psychoanalysis
  (Freud / Lacan / Jung) is its own domain, separated from
  `psychology` for triage clarity, with future expansion expected
  toward Klein / Winnicott / Bion. `applied-ethics` covers EU AI Act,
  NIST AI RMF, GDPR, OECD AI Principles, bioethics 4 principles, and
  research ethics — ISO 42001 was deferred as too narrow in
  application context for this round.
- **Pipeline maturation** — 5 system prompts and 10 golden-example
  frontmatters that still carried v0.1's Korean `one_liner` convention
  were translated to English first (commits `4d4e184`, `1707b2f`),
  preserving the v0.2.3 catalog-as-LLM-surface invariant. An
  independent `Explore`-agent integrity audit caught 12 frontmatter
  defects in the generated batch (1 slug mismatch, 11 unquoted colons
  in `source:` fields where book titles contained colons), all fixed
  in commit `db58f78` before the version bump.
- **Instrument-count limit acknowledged** — the project's stated
  out-of-scope item ("growing the bundle beyond ~700 instruments")
  is now within ~10 of its ceiling. Future contributions should
  prefer reinforcing existing domains or replacing low-quality
  entries over net-new growth.

## v0.2.3

A catalog-only patch release. No user-facing surface or runtime behavior
changes — every adjustment lives in `catalog.yml` and the heuristics
library.

- **`xy-problem` heuristic added** — registered under
  `library/heuristics/general/xy-problem.md` and the `general` section of
  the catalog. Encodes the "answer the X behind the Y" anti-pattern as a
  one-rule heuristic for requirement-gathering, debugging, and code-review
  lenses to consume in embedded or sanity-gate mode.
- **All 658 catalog `one_liner` fields translated KO→EN** — the catalog
  is consumed by the LLM triage step (instrument selection / retrieval),
  and English summaries align better with the predominantly English
  instrument names and source terminology (`razor`, `dictum`, `chronotope`,
  `OODA`, …). User-facing surfaces — README, `/prism help`, and every
  instrument body `.md` file — remain Korean. Translation was performed
  via a four-step extract → translate → merge → verify pipeline that
  preserved comments, blank lines, indentation, and field order
  byte-for-byte (line-based regex substitution, no YAML roundtrip).

## v0.2.2

A first-contact UX pass on top of v0.2.1, tightening what `/prism` shows
on invocation.

- **about.md slimmed down** — the about reference was ~60 lines covering
  class definitions, 3-layer storage, and out-of-scope rules. v0.2.2 cuts
  it to ~25 lines: one-paragraph summary, a one-line-each reminder of the
  5 classes, and three "바로 시작" entry points. Full class semantics and
  storage architecture are now on-demand — the user asks, Claude reads
  `CLASSES.md` to answer.
- **No-narration rule** — all three `references/*.md` files and the
  router `SKILL.md` now carry an explicit "do not narrate routing or
  file-loading decisions" instruction. Claude should render the reference
  content directly on `/prism` invocation, without meta commentary about
  which intent was classified.

## v0.4.0 (Later)

Extensions on top of the v0.3 structure.

- `/prism contribute` — a new intent branch of the `/prism` router for
  upstreaming a local instrument. Walks through fork, branch, and PR against
  the Prism repo, using a new `references/contribute.md` reference file
  loaded by the router when the user's request matches the contribute
  intent.
- Enhanced catalog discovery: fuzzy name matching, class/domain graph
  navigation, and "instruments commonly used together" hints derived from
  cookbook compositions.
- Community curation workflow for accepted upstream contributions, including
  review criteria and a lightweight acceptance pipeline.

## Out of scope

Things Prism explicitly does not do, regardless of version.

- **Meta-workflows** (e.g., project-management pipelines, self-iterating dev
  loops). These are larger than a single instrument and belong in the user's
  own skills or a separate harness project.
- **Agent and skill generation.** Use Claude Code's native agent creation or
  the official `skill-creator` skill. Prism's role is to supply the catalog
  that those tools reference.
- **Growing the bundle beyond ~700 instruments.** The v0.3 batch (639 new
  files generated via Haiku) already demonstrated that batch instrument
  generation is a solved, reproducible problem. The project's value is in
  curation and retrieval, not volume.
