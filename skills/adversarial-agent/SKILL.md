---
name: adversarial-review
description: Adversarial code review protocol based on structured disagreement and grounded evidence (arXiv:2608.18167). Trigger when changes (code, diff, or plan) are proposed before committing or finalizing a task.
---

## Core Constraint: Freeze the Artifact
During the review loop, **the code, diff, or plan is strictly frozen**. Do not apply hotfixes, rewrite files, or make speculative edits while the Reviewer and Critic deliberate. Edits are permitted only after findings are stabilized and verified.

---

## Phase 1: The Reviewer ($R$) — Adversarial Fault Finding

Act as an adversarial, senior code reviewer. Your objective is to actively break the implementation and uncover critical defects.

Evaluate:
- **Logical correctness:** Off-by-one errors, race conditions, unhandled nil/null/undefined states, state desynchronization.
- **Contract violations:** Undocumented side effects, broken invariants, incompatible downstream API changes.
- **Resource safety:** Leaks, unclosed descriptors/streams, unbounded memory allocation, unindexed queries.
- **Failure pathways:** Behavior under I/O errors, network drops, or invalid upstream payloads.

For each defect, you must emit a structured report:
- **Exact File and Line Range:** (e.g., `src/services/auth.ts:42-55`).
- **Failure Mechanism:** Concrete explanation of why and how the logic fails.
- **Minimal Breaking Scenario / Reproduction:** The specific input, payload, or execution sequence triggering the failure.

*Generic observations, purely stylistic nitpicks, and speculative warnings without runtime impact are strictly forbidden.*

---

## Phase 2: The Critic ($C$) — Audit of the Reviewer

Act as an independent auditor evaluating **only the claims made by the Reviewer ($R$)**. The Critic does not review the raw implementation directly; the Critic audits the validity and grounding of the Reviewer's objections.

For every finding submitted by $R$, the Critic must assign exactly one of three verdicts:

1. `AGREE`: The finding is substantiated, severe, and verified by the cited code.
2. `DISAGREE_EVIDENCE`: The Critic refutes the finding by citing explicit repo code showing the bug cannot manifest (e.g., prior sanitization, framework invariants, or impossible preconditions).
3. `DISAGREE_CONCERN`: The Critic determines the finding is speculative, nitpicky, cosmetic, or lacks verifiable grounding in the diff.

---

## Phase 3: Disagreement Resolution ($R$ Response)

The Reviewer processes the Critic's verdicts under strict evidentiary rules:

- On `AGREE`:
  - Retain the finding without dilution or compromise.
- On `DISAGREE_EVIDENCE`:
  - Inspect the evidence cited by the Critic. If verified, **retract the finding immediately**. If the evidence is incomplete, restate the defect strictly bounded by the proven facts.
- On `DISAGREE_CONCERN`:
  - Provide a one-turn defense: **quote exact code lines or write a minimal failing test case** proving reproducibility. If concrete proof cannot be produced, **discard the finding immediately**.

---

## Phase 4: Handoff & Execution

Deliberation concludes. Forward only survived, verified findings to the Primary Agent:

1. **If zero findings survive:**
   - Emit: `STATUS: VERIFIED`.
   - Proceed to commit or deliver the change.
2. **If confirmed findings survive:**
   - Present a concise list containing only validated bugs, file/line locations, and root causes.
   - Patch the code at the root cause level (do not apply superficial workarounds or disable linter rules).
   - Validate fixes against existing and new tests before completing the cycle.