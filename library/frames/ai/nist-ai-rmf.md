---
name: nist-ai-rmf
display_name: NIST AI Risk Management Framework
class: frame
underlying_class: native
domain: ai
source: National Institute of Standards and Technology (NIST), U.S. Department of Commerce, 2023
best_for:
  - Sorting an AI system or initiative into a governance and accountability stage
  - Determining which risk-management activities and oversight mechanisms apply
  - Allocating accountability and decision-rights across teams
one_liner: "Classify governance, mapping, measurement, and management activities across the AI system lifecycle."
---

# NIST AI Risk Management Framework

## Overview

The NIST AI RMF sorts an AI system or initiative into one of four interdependent stages — **Govern, Map, Measure, Manage** — based on where it sits in its risk-management lifecycle. The core claim is that different stages call for **different organizational structures, documentation practices, and decision gates**, and that skipping or conflating stages is a systematic source of uncontrolled risk escalation. The frame does not prescribe *what* risks to mitigate, but *when* and *how* to surface, structure, and monitor them. Proper classification determines which governance bodies, measurement practices, and escalation paths are legitimate.

## Categories

1. **Govern**
   - The stage of **establishing organizational policy, roles, and accountability** for AI risk management across the enterprise.
   - Inputs: organizational strategy, risk appetite, regulatory landscape, stakeholder expectations.
   - Outputs: AI governance structure, roles and responsibilities, risk policy, measurement criteria, resource allocation.
   - Discriminating criterion: the system or initiative exists primarily in organizational documents and decision-making forums, not yet in technical implementation. The question "who decides?" is more urgent than "does it work?"
   - Example: a financial services firm establishing an AI risk committee, defining escalation paths for model review, and publishing an AI risk policy.

2. **Map**
   - The stage of **identifying and documenting the AI system's purpose, context, and potential impact sources** before measurement or deployment.
   - Inputs: system design, intended use cases, known constraints and dependencies, stakeholder landscape, regulatory context.
   - Outputs: impact assessment, context documentation, risk register (preliminary), resource and capability mapping.
   - Discriminating criterion: the system is designed or in early development; stakeholders can describe what the AI is *supposed* to do, but measurement and real-world behavior are not yet the primary source of truth. The question "what could go wrong?" is being asked abstractly.
   - Example: documenting a hiring AI system's intended scope (screening resumes, not final hiring decisions), identifying potential bias sources, listing stakeholder interests (HR, legal, candidates).

3. **Measure**
   - The stage of **continuously monitoring and evaluating** the AI system's real-world behavior, performance, and harms against predefined metrics and thresholds.
   - Inputs: operational system, performance data, ground truth labels (where available), user feedback, audit logs.
   - Outputs: performance scorecards, incident reports, measurement dashboards, feedback loops.
   - Discriminating criterion: the system is deployed or in active testing; measurement results are flowing in and informing near-term operational decisions. The question "is it actually behaving as expected?" is being answered with data.
   - Example: tracking a recommendation engine's click-through rate, false-positive rate by demographic cohort, latency, and escalating to the Manage stage when metrics breach thresholds.

4. **Manage**
   - The stage of **responding to measurement findings** — adjusting systems, escalating issues, retraining models, modifying guardrails, or decommissioning when risks become untenable.
   - Inputs: measurement alerts, incident reports, feedback, regulatory changes, stakeholder escalations.
   - Outputs: action plans, system updates, incident resolutions, decommissioning decisions, feedback to Govern and Map stages.
   - Discriminating criterion: a measurement-driven decision or anomaly requires action; the system is in active governance response mode. The question "what do we do about this?" has moved from planning into execution.
   - Example: a model's accuracy drops below acceptable thresholds for a protected class; the team retrains, applies a fairness constraint, or holds the system from production until remediation is complete.

## Classification Procedure

1. **Clarify the scope.** Name the specific AI system, initiative, or policy question you are sorting (not "AI in our company" but "our resume-screening model" or "our AI governance charter").

2. **Ask: Does an organizational AI governance structure exist that covers this system?**
   - If **no, or only nascent**, the system may be in **Govern** stage. Proceed to step 3.
   - If **yes**, proceed to step 4.

3. **If in Govern: Is the system primarily being defined through policy, roles, and decision-rights documentation?**
   - If **yes** → Classify as **Govern**. The next move is to advance governance maturity (clarity of roles, accountability, escalation paths).
   - If **no**, return to step 2 — governance exists, so move to step 4.

4. **Ask: Is the system actively deployed or in active measurement/testing?**
   - If **no** → proceed to step 5 (likely **Map**).
   - If **yes** → proceed to step 6.

5. **If not yet deployed: Is a detailed impact assessment, stakeholder context map, and preliminary risk register documented?**
   - If **yes** → Classify as **Map**. The next move is to finalize measurement criteria before deployment.
   - If **no**, return to step 2 — governance clarity is still missing.

6. **If deployed or in active testing: Are measurement results (performance metrics, incident reports, feedback) flowing regularly and informing decisions?**
   - If **yes** → proceed to step 7.
   - If **no** → classify as **Map** (system is live but measurement discipline is missing; this is a structural gap).

7. **Ask: Is a measurement threshold breached, an incident active, or a stakeholder escalation requiring system modification or governance change in progress?**
   - If **yes** → Classify as **Manage**. The next move is to execute the response and document it.
   - If **no** → Classify as **Measure**. The next move is to maintain measurement discipline and tune thresholds.

## Implications per Category

| Category | Primary question | Accountability | Key activity | Next escalation |
|---|---|---|---|---|
| **Govern** | *Who decides, and by what criteria?* | C-level, governance committee, policy owner | Define roles, policy, escalation paths, risk appetite | Establish measurement readiness (→ Map) |
| **Map** | *What is this system, and what could go wrong?* | Product owner, data team, risk analyst | Impact assessment, stakeholder mapping, preliminary risk register, measurement criteria design | Instrument and deploy (→ Measure) |
| **Measure** | *Is it behaving as expected?* | Operations, monitoring team, model owner | Real-time dashboards, incident alerting, feedback loops, performance scoring | Respond to threshold breach or incident (→ Manage) |
| **Manage** | *What do we do about this finding?* | Incident commander, model retraining team, governance committee | Root cause analysis, system updates, escalation, decommissioning decisions, feedback to policy | Close incident and return to Measure; update Govern if policy change needed |

Each stage also feeds **backward**: a Manage decision may reveal a governance gap (feedback to Govern), a measurement finding may invalidate a Map assumption (feedback to Map), or a policy change may require re-measurement (Govern → Measure loop).

## Common Misclassifications

- **Govern mistaken for Map.** Building an AI system without governance in place, then trying to retrofit policy after deployment. The tell is that escalation paths and role clarity are unclear when an incident surfaces. Consequence: slow response, accountability diffusion.

- **Map mistaken for Measure.** Assuming that design-time impact assessment is sufficient without continuous measurement. The tell is that the system is deployed but measurement discipline is weak or nonexistent. Consequence: drift from intended behavior is undetected.

- **Measure mistaken for Manage.** Collecting metrics without action thresholds or escalation procedures. The tell is that dashboards exist but there is no agreed-upon decision rule for when to intervene. Consequence: metrics become organizational theater.

- **Manage mistaken for Measure.** Treating every incident as a one-off fire-to-extinguish rather than feeding it back into measurement discipline. The tell is that the same class of incident recurs. Consequence: reactive posture, no systemic improvement.

- **Skipping Map.** Deploying a system directly from Govern to Measure without documented impact assessment and stakeholder alignment. The tell is that measurement criteria are undefined or contested. Consequence: disputes over whether the system is performing acceptably.

- **Classifying the entire "AI effort" instead of specific systems.** NIST RMF applies per-system or per-capability, not enterprise-wide; a firm may simultaneously Govern a new policy, Map a new model, Measure an existing system, and Manage an incident in a third. Consequence: confusion about which governance structure applies to which decision.
