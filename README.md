# lenslab

**Lenses over personas.** A Claude Code plugin for building domain-expert analysis agents that actually reason like experts — not by pretending to be one, but by loading the structured analytical frameworks experts actually use.

## The problem with persona prompts

Telling an LLM "You are a senior security engineer" changes its tone but not its reasoning depth. It will write more assertive sentences and sprinkle in domain vocabulary, but the underlying analysis remains ad-hoc. Ask it to review an API and you'll get a confident-sounding list of suggestions that misses the same things an untrained reviewer would miss.

Real security engineers don't run on vibes. They walk checklists: STRIDE for threat modeling, OWASP Top 10 for API review, CVSS for severity. These checklists are the reason their analysis is deep — they force coverage, they surface blind spots, and they produce findings that compare across reviews.

The same is true in every domain. Curriculum reviewers use Bloom's taxonomy and achievement-standard alignment. Strategists use Eisenhower matrices and pre-mortems. Researchers use Occam's Razor and first-principles decomposition.

**lenslab treats these frameworks as first-class, reusable assets.** Each one is a structured "lens" file with a concrete Analytical Procedure. You build agents by composing lenses, not by writing adjective-heavy persona prompts.

## Core ideas

- **Lenses > personas.** An agent's analytical depth lives in the lens file. The persona is a one-line framing; the lenses do the work.
- **Lenses are reusable.** Once written, the same `stride.md` lens powers a security-reviewer agent, a DevSecOps pipeline agent, and a threat-modeling workshop facilitator.
- **Catalog first, load on demand.** All lenses live in `lenses/catalog.yml` as a triage index. Agents read the catalog first, select the lenses that apply, and only then load the full lens files. This keeps context small.
- **Agents stay thin.** An agent config is ~30 lines: a one-sentence persona, a list of lens references with `always` / `when-relevant` / `on-request` usage tags, and the fixed triage → analyze → synthesize workflow.

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
2. Read `lenses/catalog.yml` and propose a candidate lens set.
3. Let you add or remove lenses.
4. Generate any missing lenses (following the required file format).
5. Write the agent config to `.claude/agents/<agent-name>.yml` in the current project.

### Invoke the agent

Once the config is saved, invoke the agent like any other Claude Code agent. It will:

1. **Triage** — read `catalog.yml`, decide which of its attached lenses apply to the specific input, and state the reasoning.
2. **Analyze** — apply each selected lens using its `Analytical Procedure`, emitting results in the lens's `Output Format`. Each finding is tagged with confidence.
3. **Synthesize** — cross-check findings across lenses, escalate issues confirmed by multiple lenses, and produce a prioritized recommendation list.

### Example agents

See `agents/examples/` for templates:

- **security-analyst.yml** — STRIDE + OWASP API Top 10 + CVSS, with rumsfeld-matrix as a when-relevant meta layer.
- **curriculum-reviewer.yml** — Achievement Standard Alignment + Bloom's Taxonomy + Cognitive Load Theory for Korean K-12 materials.

## Included lenses

### General / Meta

| Lens | Best for |
|---|---|
| `rumsfeld-matrix` | Mapping knowledge state and surfacing Unknown Unknowns |
| `eisenhower-matrix` | Triaging backlogs into Do / Schedule / Delegate / Eliminate |
| `occams-razor` | Selecting between hypotheses by comparing explicit assumption counts |
| `first-principles` | Breaking free of analogies by decomposing to irreducible requirements |
| `socratic-method` | Stress-testing a claim across six question categories |

### Security

| Lens | Best for |
|---|---|
| `stride` | Design-time threat enumeration across components × 6 threat categories |
| `owasp-top10` | Endpoint-by-endpoint review against OWASP API Security Top 10 (2023) |
| `cvss-scoring` | Objective severity scoring using CVSS v3.1 Base Metrics |

### Education

| Lens | Best for |
|---|---|
| `blooms-taxonomy` | Classifying objectives, content, and assessment by cognitive level |
| `achievement-standard-alignment` | Verifying alignment to Korean national curriculum (성취기준) |
| `cognitive-load-theory` | Diagnosing learning-material difficulty as intrinsic / extraneous / germane load |

## File format reference

### Lens file (`lenses/<domain>/<name>.md`)

```markdown
---
name: <framework name>
domain: <general | security | education | ...>
source: <author, institution, year>
best_for: <one line — when this lens is most useful>
---

# <framework name>

## Overview
<2-3 sentences>

## Analytical Procedure
<Numbered, directly-executable steps. The load-bearing section.>

## Evaluation Criteria
<Checklist or rubric>

## Red Flags
<Concrete patterns that indicate a problem>

## Output Format
<Template for structured output>
```

### Catalog entry (`lenses/catalog.yml`)

```yaml
- name: <lens-name>
  domain: <domain>
  path: <domain>/<lens-name>.md
  one_liner: "<specific enough for triage>"
```

### Agent config (`.claude/agents/<name>.yml`)

See `agents/examples/` for complete templates.

## Contributing a new lens

1. **Identify the framework.** It must be a real method used by practitioners, not something you just thought of. Cite the source.
2. **Write the lens file** following the format above. The `Analytical Procedure` is the make-or-break section — it must be concrete enough that someone unfamiliar with the framework can execute it step by step. Abstract instructions ("assess quality", "consider impact") are a failure mode.
3. **Register the lens in `catalog.yml`** with a `one_liner` specific enough that a triage step can decide whether to load it.
4. **Open a PR** with the new lens, the catalog update, and (if appropriate) an updated example agent that uses it.

Contributions of non-English-language frameworks are especially welcome — the existing `achievement-standard-alignment` lens is Korea-specific by design, and similar domain-local lenses exist for many other educational, legal, and regulatory contexts worldwide.

## Why "lenslab"

A lens shapes what you see without telling you what to think. A lab is where you craft and test new instruments. Together: a workshop for building analytical instruments, and a collection of ones already made.

## License

MIT. See `LICENSE` for details.
