# Prism Roadmap

Prism is a Claude Code plugin that provides a catalog of analytical instruments
across 5 classes — lens, frame, model, stance, heuristic — for use when building
domain-expert agents and skills. This roadmap describes where the catalog stands
today and what each upcoming minor version changes.

## v0.1.0 (Current)

- Renamed from lenslab. Same 657-instrument library across 50 domains.
- Plugin identity clarified: Prism is a catalog reference provider, not an agent
  or skill generator.
- Top-level class term unified as "instrument" (lens, frame, model, stance,
  heuristic are its five sub-classes).
- Breaking change from lenslab v0.3.0: name, plugin identifier, and package
  version all reset to 0.1.0 to mark a fresh start under the new identity.
- Existing `create-agent` slash command and `agent-creator` skill still ship in
  this release for continuity, but are deprecated and removed in v0.2.0.

## v0.2.0 (Next)

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

## v0.3.0 (Later)

Extensions on top of the v0.2 structure.

- `contribute-instrument` skill — when a user has created a local instrument
  they want to upstream, this skill walks through fork, branch, and PR against
  the Prism repo.
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
