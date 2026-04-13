---
name: debate
description: Orchestration skill where multiple agent personas independently analyze a document, proposal, or problem and converge through structured rounds. Three modes — review (multi-perspective analysis with judgment), ideate (divergent idea exploration), solve (problem analysis into solution convergence). Infers mode from context when no flag is given. Prism instruments (stances, lenses) can serve as expertise sources for participants. Triggers on "analyze from multiple perspectives", "have agents debate", "debate", "consensus", "multi-angle review", "find solutions", "brainstorm ideas", "ideation session"
---

# debate

An orchestration engine where multiple persona agents independently
analyze a target, with the orchestrator controlling rounds and driving
consensus, convergence, or divergence.

---

## Modes

| Mode | Purpose | Exit condition | Typical scenarios |
|---|---|---|---|
| `review` | Multi-perspective analysis with judgment | Consensus reached or MAX_ROUNDS exhausted | Code review, design review, PRD review |
| `ideate` | Maximize idea divergence and expansion | MAX_ROUNDS exhausted (no convergence judgment) | Brainstorming, ideation, exploratory analysis |
| `solve` | Propose solutions and converge on the best | Convergence reached or MAX_ROUNDS exhausted | Bug resolution, incident response, strategy planning |

### Mode selection rules

1. **Explicit flag** — if the user specifies `--review`, `--ideate`, or `--solve`, enter that mode immediately
2. **Context inference** — without a flag, infer from signals:
   - Analysis / review / judgment / assessment → `review`
   - Ideas / divergence / brainstorming / exploration → `ideate`
   - Fix / resolve / alternatives / strategy → `solve`
3. **Default to `review`** when uncertain — the most general-purpose mode
4. **Mode transition** — after `review` completes, if unresolved issues remain and the user asks "find solutions too," transition to `solve` is natural. **Never auto-transition** — always confirm with the user first

---

## Prerequisites

- **Target**: document, text, code, proposal, problem statement, etc.
- **Participating agents**: composed from one or more of these sources:
  - Agent files in `.claude/agents/` (project or user root)
  - Prism instruments (especially `stance`, `lens`) — loaded via `/prism fetch` as expertise sources for personas
- **If no agents are specified, confirm with the user before proceeding** (no automatic selection)

### Prism integration

When the user says "debate with prism instruments" or names specific instruments:
1. Load instrument paths and summaries via `/prism fetch`
2. Insert instrument read directives into each subagent prompt
3. Each subagent uses the instrument's procedure/perspective as its analysis frame

Without integration, the skill operates on pure agent personas. Prism is an optional enhancement, not a hard dependency.

---

## Execution flow

### STEP 0 — Session initialization

The orchestrator (main session) performs:

1. Prepare the target as a `[TARGET]` block (document, problem statement, etc.)
2. Determine mode (apply rules above)
3. Confirm participating agent list and assign role labels
4. Set configuration:
   - `MAX_ROUNDS`: default 4 (user-configurable)
   - `CONSENSUS_THRESHOLD`: default 0.75 (used in review/solve modes)
5. Initialize state: `round = 1`, `issues = []`, `solution_pool = []`

---

### STEP 1 — Round execution (iterative)

Each round: **spawn all participating agents in parallel via the Agent tool.**
Use mode-specific prompt templates.

#### Common prompt header

```
You are [AGENT_NAME].
Role description: [AGENT_DESCRIPTION]
[PRISM_INSTRUMENT_DIRECTIVE — only if applicable]

## Target
[TARGET]

## Current round
Round [N] / [MAX_ROUNDS]

## Previous round summary (omit for Round 1)
[PREVIOUS_ROUND_SUMMARY]
```

#### Mode-specific tasks and output formats

---

### MODE: review

**Task block:**
```
## Your task
1. Analyze the target from your persona's perspective.
2. If previous rounds exist, review other agents' arguments and state agreement/disagreement explicitly.
3. Follow the JSON output format below exactly. No text output outside JSON.
```

**Output JSON:**
```json
{
  "agent": "[AGENT_NAME]",
  "round": "[N]",
  "analysis": {
    "key_findings": ["Key finding (one sentence each)"],
    "concerns": ["Concern or issue"],
    "strengths": ["Strength or positive aspect"]
  },
  "stance_on_pending_issues": [
    {
      "issue": "Pending issue text",
      "position": "agree | disagree | neutral",
      "rationale": "Rationale for position (2-3 sentences)"
    }
  ],
  "new_issues": [
    {
      "issue": "Newly raised issue",
      "severity": "critical | major | minor",
      "rationale": "Why this matters"
    }
  ],
  "overall_recommendation": "approve | approve_with_conditions | reject",
  "recommendation_rationale": "Rationale for final recommendation (3-5 sentences)"
}
```

---

### MODE: ideate

**Task block:**
```
## Your task
1. Analyze the target from your persona's perspective and propose as many diverse ideas as possible.
2. Draw inspiration from other agents' ideas to extend, remix, or combine. Prioritize expansion over criticism.
3. Seek angles that don't overlap with existing ideas.
4. Follow the JSON output format below exactly.
```

**Output JSON:**
```json
{
  "agent": "[AGENT_NAME]",
  "round": "[N]",
  "perspective": "This agent's unique lens on the target (1-2 sentences)",
  "ideas": [
    {
      "title": "Idea title",
      "description": "Detailed description (2-3 sentences)",
      "novelty": "high | medium | low",
      "inspired_by": "Other agent's idea that inspired this (null if original)",
      "tags": ["classification tags"]
    }
  ],
  "combinations": [
    {
      "source_ideas": ["Titles of ideas being combined"],
      "combined_idea": "New idea born from combination",
      "synergy": "Why this combination creates value"
    }
  ],
  "unexplored_angles": ["Suggested directions not yet explored"]
}
```

---

### MODE: solve

**Task block (Round 1):**
```
## Your task
1. Analyze the problem from your persona's perspective and propose a solution.
2. Follow the JSON output format below exactly.
```

**Task block (Round 2+):**
```
## Your task
1. Review other agents' solutions.
2. Specify which elements you accept, which need improvement, and which are unacceptable.
3. Present a new or improved version that integrates prior proposals.
4. Explicitly note any withdrawals or revisions to your previous proposal.
```

**Output JSON:**
```json
{
  "agent": "[AGENT_NAME]",
  "round": "[N]",
  "problem_framing": "Core perspective on the problem (1-2 sentences, Round 1 only)",
  "proposed_solution": {
    "title": "Solution title",
    "description": "Detailed description (3-5 sentences)",
    "key_steps": ["Execution step"],
    "feasibility": "high | medium | low",
    "feasibility_rationale": "Feasibility assessment rationale"
  },
  "review_of_others": [
    {
      "agent": "Agent under review",
      "elements_accepted": ["Acceptable elements"],
      "elements_rejected": [
        { "element": "Unacceptable element", "reason": "Rejection reason" }
      ],
      "suggested_improvement": "Improvement suggestion (null if none)"
    }
  ],
  "tradeoffs": [
    {
      "tradeoff": "Tradeoff description",
      "severity": "critical | major | minor",
      "mitigation": "Mitigation approach (null if none)"
    }
  ],
  "convergence_signal": {
    "willing_to_adopt": ["Elements willing to integrate"],
    "dealbreakers": ["Absolutely unacceptable elements"]
  },
  "revised_from_previous": "Changes from previous round (null for Round 1)"
}
```

---

### STEP 2 — Judgment (orchestrator)

After each round, the orchestrator analyzes collected JSON results.
**Judgment logic varies by mode.**

#### review mode — consensus judgment

**1. Recommendation agreement rate (quantitative)**

| overall_recommendation distribution | Judgment |
|---|---|
| Same value >= 75% | Direction consensus |
| Same value 50–74% | Partial consensus (conditional proceed) |
| Same value < 50% | No consensus |

**2. Issue resolution rate (quantitative)**

```
resolution_rate = (pending_issues where all agents agree) / (total pending_issues)
```
- >= 0.8 → sufficient
- < 0.8 → next round needed

**3. Critical issue presence (takes priority)**
- Any unresolved `severity: critical` `new_issues` → no consensus (blocking condition)

**Exit condition (AND):** agreement >= 75% + resolution >= 80% + no critical issues remaining

#### ideate mode — no judgment

- No convergence/consensus judgment is performed
- Instead, track the count of unique new ideas per round
- Include "cumulative idea pool status" in round summaries
- On MAX_ROUNDS exhaustion, compile the full idea pool into a report

#### solve mode — convergence judgment

**1. Solution convergence rate (quantitative)** — core approach similarity

| Agreement rate | Judgment |
|---|---|
| >= 75% | Direction convergence |
| 50–74% | Partial convergence |
| < 50% | No convergence |

**2. Dealbreaker presence (takes priority)**
- Any remaining `dealbreakers` → no convergence (blocking condition)

**3. Critical tradeoff mitigation rate (quantitative)**
```
mitigation_rate = (critical tradeoffs with mitigation) / (total critical tradeoffs)
```
- >= 0.8 → sufficient
- < 0.8 → next round needed

**Exit condition (AND):** no dealbreakers + convergence >= 75% + mitigation >= 80%

---

### STEP 3 — Round summary (orchestrator)

When exit conditions are not met, the orchestrator generates a summary
for the next round.

#### review mode summary
```
PREVIOUS_ROUND_SUMMARY:
  Agreed items: [items where all agents agree]
  Pending issues: [items with disagree/neutral + new critical/major issues]
  Agent stances:
    [AGENT]: recommendation + rationale summary (1-2 sentences)
```

#### ideate mode summary
```
PREVIOUS_ROUND_SUMMARY:
  Idea pool (cumulative): [all unique ideas — title + proposer]
  New this round: [ideas first appearing this round]
  Combined ideas: [ideas from combinations]
  Unexplored directions: [aggregated unexplored_angles]
```

#### solve mode summary
```
PREVIOUS_ROUND_SUMMARY:
  Solution pool:
    [Agent A]: [solution.title] — [description summary, 1 sentence]
    [Agent B]: ...
  Agreed elements: [intersection of all willing_to_adopt]
  Unresolved tradeoffs: [critical items with null mitigation]
  Dealbreakers: [remaining dealbreaker list]
```

---

### STEP 4 — Final report

Generated when exit conditions are met or MAX_ROUNDS is exhausted.

#### review mode report

```markdown
# Debate Final Report — Review

**Target:** [target summary]
**Participants:** [agent list]
**Rounds completed:** [N] / [MAX_ROUNDS]
**Consensus status:** Consensus reached | Partial consensus | No consensus

---

## 1. Final recommendation
**Agreed recommendation:** [approve | approve_with_conditions | reject | split]
[Core rationale summary, 2-4 sentences]

## 2. Agreed items
- [Item 1]
- [Item 2]

## 3. Remaining disagreements
| Issue | Severity | Agent positions |
|---|---|---|
| [content] | critical/major/minor | [A: agree, B: disagree, ...] |

## 4. Per-agent final positions
### [Agent A]
- **Recommendation:** approve/reject/...
- **Key findings:** ...
- **Main concerns:** ...

## 5. Conditional approval actions (if applicable)
1. [Required action]

---
*Generated: [timestamp] | Skill: debate (review)*
```

#### ideate mode report

```markdown
# Debate Final Report — Ideate

**Topic:** [target summary]
**Participants:** [agent list]
**Rounds completed:** [N] / [MAX_ROUNDS]
**Unique ideas generated:** [N]

---

## 1. Idea map

### By category
#### [Category/tag A]
- **[Idea title]** (by [Agent]) — [1-sentence description] | novelty: high/medium/low
- ...

#### [Category/tag B]
- ...

## 2. Notable combinations
| Source ideas | Combined result | Synergy |
|---|---|---|
| [A's X + B's Y] | [Combined idea] | [Synergy description] |

## 3. Unexplored directions
> Angles suggested by agents but not yet deeply explored
- [Direction 1]
- [Direction 2]

## 4. Per-agent perspective summary
### [Agent A]
- **Unique perspective:** ...
- **Ideas proposed:** N
- **Highest novelty idea:** ...

---
*Generated: [timestamp] | Skill: debate (ideate)*
```

#### solve mode report

```markdown
# Debate Final Report — Solve

**Problem:** [problem summary]
**Participants:** [agent list]
**Rounds completed:** [N] / [MAX_ROUNDS]
**Convergence status:** Converged | Partial convergence | No convergence

---

## 1. Agreed solution
**Title:** [integrated solution title]
[Core approach, 3-5 sentences]

### Execution steps
1. [Step 1]
2. [Step 2]

### Feasibility
**Overall assessment:** high | medium | low
[Rationale, 2-3 sentences]

## 2. Agreed core elements
- [Element 1]
- [Element 2]

## 3. Tradeoffs and mitigations
| Tradeoff | Severity | Mitigation |
|---|---|---|
| [content] | critical/major/minor | [approach or "unresolved"] |

## 4. Unresolved divergences (if applicable)
| Issue | Agent positions | Orchestrator judgment |
|---|---|---|
| [divergence] | [A: option X, B: option Y] | [judgment] |

## 5. Per-agent final proposals
### [Agent A]
- **Proposal:** [title]
- **Core approach:** ...
- **Feasibility:** high/medium/low

## 6. Recommended next steps
1. [Immediate action 1]
2. [Immediate action 2]

---
*Generated: [timestamp] | Skill: debate (solve)*
```

---

## Orchestrator checklist

### Before each round
- [ ] Target is fully included in subagent prompts
- [ ] Previous round summary is accurately composed
- [ ] Each subagent is **independently** spawned (no shared context)
- [ ] Prism instrument directives are correctly inserted for applicable subagents

### After each round
- [ ] JSON parse failures: exclude affected agent from round results, notify user
- [ ] Mode-specific judgment logic correctly applied (ideate has none)
- [ ] review/solve: check blocking conditions (critical issues / dealbreakers) first
- [ ] review/solve: compute quantitative metrics, evaluate exit conditions

### When writing final report
- [ ] Use the correct report format for the active mode
- [ ] If terminated by MAX_ROUNDS exhaustion, explicitly list unresolved items
- [ ] If mode transition is natural, suggest next step to user (never force)

---

## NOT this skill

- **Catalog browsing** — route to `/prism search`
- **Instrument creation** — route to `/prism`
- **Instrument loading** — route to `/prism fetch` (debate can invoke this internally)
- **Single-analyst sequential multi-tool analysis** — use Prism's standard 7-step workflow directly. Debate is for **multi-perspective parallel analysis + consensus/divergence** only

---

## Operational rules

- Subagents must be spawned **in parallel** via the Agent tool — no sequential execution
- Each subagent has independent context — no real-time sharing of other subagents' current-round responses
- Inter-round information transfer happens only through orchestrator-generated summaries
- MAX_ROUNDS default is 4 — most debates converge in 2-3 rounds; 4 is the safety ceiling
- On JSON parse failure, exclude that agent from the round and notify the user in one line
