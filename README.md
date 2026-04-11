# lenslab

**Frameworks over personas.** A Claude Code plugin for building
domain-expert analysis agents that actually reason like experts — not by
pretending to be one, but by loading the structured analytical
frameworks experts actually use.

v0.3 ships a library of **657 framework files** across 5 classes and 50
domains, all indexed by a single `catalog.yml` triage file.

## The problem with persona prompts

Telling an LLM "You are a senior security engineer" changes its tone but
not its reasoning depth. It will write more assertive sentences and
sprinkle in domain vocabulary, but the underlying analysis remains
ad-hoc. Ask it to review an API and you'll get a confident-sounding list
of suggestions that misses the same things an untrained reviewer would
miss.

Real security engineers don't run on vibes. They walk checklists: STRIDE
for threat modeling, OWASP Top 10 for API review, CVSS for severity.
These checklists are the reason their analysis is deep — they force
coverage, they surface blind spots, and they produce findings that
compare across reviews.

The same is true in every domain. Curriculum reviewers use Bloom's
Taxonomy and achievement-standard alignment. Strategists use Porter's
Five Forces and pre-mortems. Decision scientists use Prospect Theory and
Occam's Razor. Cultural critics read through Marxist and Foucauldian
stances.

**lenslab treats these frameworks as first-class, reusable assets.** You
build agents by composing frameworks, not by writing adjective-heavy
persona prompts.

## The 5 classes

In v0.1, lenslab called every framework a "lens" — which was imprecise.
Real frameworks come in different shapes, and flattening them all into
one template loses information. v0.2 introduced an explicit 5-class
taxonomy; v0.3 scales it out to 657 files. See `CLASSES.md` for the full
reference.

| Class | What it is | Examples |
|---|---|---|
| **Lens (렌즈)** | Structured executable analytical procedure with input, steps, output format, and confidence signal. | STRIDE, OWASP API Top 10, CVSS, Bloom's Taxonomy, First Principles, Pre-mortem |
| **Frame (분류 프레임)** | Taxonomy or classification matrix — categories with discriminating criteria, no built-in procedure. | Cynefin, Kano Model, BCG Matrix, Webb's DOK, MoSCoW |
| **Model (이론 모델)** | Descriptive or predictive theoretical model — variables, relationships, predictions. | Porter's Five Forces, Prospect Theory, Nash Equilibrium, Lotka-Volterra |
| **Stance (비판적 관점)** | Interpretive position — commitments about what is worth looking for. | Marxist criticism, Foucauldian power/knowledge, Deconstruction, Bourdieu's Habitus |
| **Heuristic (원리 / 격언)** | Single rule of thumb — too small to be a lens, used as a check. | Occam's Razor, Hanlon's Razor, Chesterton's Fence, Hickam's Dictum |

The **strict definition of a lens** is specific: it must have a defined
input type, an executable procedure, a structured output format, AND a
confidence evaluation. If any of the four is missing, the framework
belongs to one of the four sister classes.

## Agent workflow

Agents run a 7-step pipeline. Simple agents use only steps 1, 3, and 7;
richer agents enable the optional middle steps as needed.

```
1. Triage          — read catalog.yml; select items of any class
2. Classify        — apply FRAMES to sort the input               [optional]
3. Analyze         — apply LENSES in sequence                     [core]
4. Model-interpret — instantiate MODELS on input or findings      [optional]
5. Critical-read   — apply STANCES to surface hidden structures   [optional]
6. Sanity-gate     — run HEURISTIC bundles as final checks        [optional]
7. Synthesize      — cross-check across all items, prioritize
```

Each class maps to exactly one workflow step. The agent config declares
which items to load and what `usage` tag each has (`always`,
`when-relevant`, `on-request`).

## Core ideas

- **Frameworks > personas.** An agent's analytical depth lives in the
  framework files. The persona is a one-line framing; the frameworks do
  the work.
- **Frameworks are reusable.** Once written, the same `stride.md`
  powers a security-reviewer agent, a DevSecOps pipeline agent, and a
  threat-modeling workshop facilitator.
- **Catalog first, load on demand.** Every item lives in `catalog.yml`
  at the repo root as a triage index. Agents read the catalog first,
  select what applies, and only then load the full files. This keeps
  context small.
- **Agents stay thin.** An agent config is ~40 lines: a one-sentence
  persona, per-class lists of item references with usage tags, and the
  workflow steps the agent actually uses.

## Installation

```
/plugin install 97Wobbler/lenslab
```

Or clone and reference locally:

```bash
git clone https://github.com/97Wobbler/lenslab.git
# Then follow your plugin loader's instructions for local plugins.
```

## Usage

### Create an agent with the slash command

```
/lenslab:create-agent security "Review our public API for design-time vulnerabilities"
```

The `agent-creator` skill will:

1. Parse the domain and purpose.
2. Read `catalog.yml` (top level) and propose a candidate item set
   grouped by class.
3. Let you add or remove items before finalizing.
4. Generate any missing items, picking the correct class for each
   source framework (it will ask before converting a frame or model
   into a lens).
5. Write the agent config to `.claude/agents/<agent-name>.yml` in the
   current project.

### Invoke the agent

Once the config is saved, invoke the agent like any other Claude Code
agent. It will walk the 7-step workflow, running only the steps its
config enables.

### Example agents

See `agents/examples/`:

- **security-analyst.yml** — STRIDE + OWASP API Top 10 + CVSS as always
  lenses, rumsfeld-matrix as a when-relevant meta lens, Cynefin as a
  when-relevant triage frame, and the general heuristics bundle as a
  sanity gate.
- **curriculum-reviewer.yml** — Achievement Standard Alignment + Bloom's
  Taxonomy as always lenses, Cognitive Load Theory as when-relevant,
  Cynefin as a when-relevant classification frame, and the general
  heuristics bundle as a sanity gate.

## Included items

The v0.3 library ships with **657 framework files** across 5 classes and
50 domains. The v0.2 plugin shipped with 18 hand-written files; the jump
to 657 was made possible by a class-specific batch generation pipeline
that runs Haiku 4.5 in parallel against a curated seed catalog (see
`scripts/PHASE3_RUN.md` and the `v0.3: add … generated … files` commits
for the full batch history).

Use `catalog.yml` as the single triage index — the `agent-creator` skill
reads it first and only loads individual files when the domain matches.

### By class

| Class | Count | Example entries |
|---|---|---|
| Lens | 312 | `stride`, `first-principles`, `pre-mortem`, `swot`, `business-model-canvas`, `socratic-method` |
| Frame | 96 | `cynefin`, `kano-model`, `2-2-matrix`, `pestel`, `bcg-growth-share-matrix`, `wardley-map` |
| Model | 115 | `prospect-theory`, `coase-theorem`, `comparative-advantage`, `efficient-market-hypothesis`, `solow-growth-model` |
| Stance | 71 | `marxist-criticism`, `foucauldian-power-knowledge` |
| Heuristic | 63 + 1 bundle | `chestertons-fence`, `hanlons-razor`, `pareto-80-20`, `parkinsons-law`, `pigeonhole-principle` |

The heuristics class also ships a curated `library/heuristics/general.md`
bundle (Occam's Razor, Hanlon's Razor, Chesterton's Fence, Hickam's
Dictum, Pigeonhole Principle, Pareto 80/20, Parkinson's Law, Mise en
Place) for use as a single sanity-gate input.

### Domain coverage

The 50 domains include: security, strategy, product-management, ux,
law, medicine, ai, data, chemistry, physics, biology, ecology, economics,
finance, philosophy, theology, linguistics, literary-theory, music-theory,
film-theory, history, sociology, psychology, manufacturing, operations,
education, agile, okr, negotiation, marketing, communication,
decision, information-analysis, meta-science, military-strategy,
storytelling, sports, cooking, geology, astronomy, mathematics,
game-theory, productivity, personal-knowledge-management-pkm, and
several others.

Not every domain is populated in every class — lenses are the most
broadly covered (47 domains); frames, models, stances, and heuristics
each cover the subset where that class is the natural fit.

## File formats

For the full file format templates of each class, see **`CLASSES.md`**
at the repo root. In brief:

- **Lens**: `Overview / Analytical Procedure / Evaluation Criteria /
  Red Flags / Output Format`
- **Frame**: `Overview / Categories / Classification Procedure /
  Implications per Category / Common Misclassifications`
- **Model**: `Overview / Core Variables and Relationships / Key
  Predictions / Application Procedure / Boundary Conditions / Output
  Format`
- **Stance**: `Overview / Foundational Commitments / Guiding Questions
  / What It Reveals / What It Obscures / Output Format`
- **Heuristic**: `The Rule / When It Applies / When It Misleads /
  Common Misuse / How Agents Use It` (usually bundled as one file per
  domain)

## Extending the library

To add a single new framework file by hand:

1. **Identify the framework.** It must be a real method used by
   practitioners, not something you just thought of. Cite the source.
2. **Pick the correct class.** Read `CLASSES.md` § "Picking the right
   class." Does the source include a procedure? → lens. Is it just a
   taxonomy? → frame. A theoretical model? → model. An interpretive
   commitment? → stance. A single-rule aphorism? → heuristic.
3. **Write the file** at `library/{class}s/{domain}/{slug}.md` using
   the matching template in `CLASSES.md`. The load-bearing section
   differs by class (Analytical Procedure for a lens; Classification
   Procedure for a frame; Application Procedure for a model; Guiding
   Questions for a stance; When It Applies / Misleads for a heuristic).
4. **Register the item in `catalog.yml`** under the right class
   section with a `one_liner` specific enough that a triage step can
   decide whether to load it.
5. **Open a PR** with the new file, the catalog update, and (if
   appropriate) an updated example agent that uses it.

### Adding many frameworks at once

To add frameworks in bulk, see `scripts/PHASE3_RUN.md` for the batch
generation pipeline. The entry point is `scripts/lenslab_batch.py`,
which is a markdown-output adapter over a parallel runner vendored
from the iris-curriculum project. It drives Haiku 4.5 with
class-specific few-shot prompts from `scripts/prompts/` and writes
generated files straight into `library/{class}s/{domain}/`. The full
v0.3 batch of 639 files cost roughly a few dollars on Haiku 4.5.

Contributions of non-English-language and domain-local frameworks are
especially welcome — the existing `achievement-standard-alignment` lens
is Korea-specific by design, and similar local frameworks exist for
many other educational, legal, regulatory, and cultural contexts
worldwide.

## Why "lenslab"

A lens shapes what you see without telling you what to think. A lab is
where you craft and test new instruments. Together: a workshop for
building analytical instruments, and a collection of ones already made.
v0.2 introduced the 5-class taxonomy — lenses are still the headline
instrument, but they now sit alongside frames, models, stances, and
heuristics as first-class citizens. v0.3 grows the collection from
18 hand-written files to 657 via a batch generation pipeline.

## License

MIT. See `LICENSE` for details.
