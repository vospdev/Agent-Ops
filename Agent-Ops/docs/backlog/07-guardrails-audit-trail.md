# 07 — Guardrail Policy Engine and Audit Trail

## What

Build the Guardrail Policy Engine as a first-class structured component
(not agent-embedded logic), and wire the human approval flow to produce a
full audit record on every decision.

## Why

A guardrail decision should be a deterministic policy outcome derived from
structured signals, not an LLM judgment call. A human-approval system
needs to know who approved what and why.

## Current behavior

No guardrail policy engine or audit trail exists.

## Expected behavior

Every run produces a `MappingDecision`, consumed once by a pure-logic
policy engine that resolves ALLOW/REVIEW/REJECT deterministically. Every
human-resolved REVIEW produces a complete audit record. CI blocks merges
on guardrail regressions, not just mapping-accuracy regressions.

## Technical context

- `MappingDecision` object (confidence, pii_detected, type_mismatch,
  transformation_risk, sensitivity, review_required) produced once by
  Integration Copilot's deterministic validation stage, consumed by the
  policy engine — never re-derived ad hoc
- Policy engine is pure logic (thresholds + rules), no LLM call —
  testable and deterministic
- Audit records stored in Neon, linked to the run's `trace_id` in
  Langfuse Cloud
- "Resolved" status requires either ALLOW from the policy engine or an
  explicit authenticated human approval — **never** auto-resolves on
  REVIEW
- CI eval suite includes cases specifically testing the guardrail path
  (PII correctly flagged, low-confidence correctly flagged, adversarial
  input correctly rejected) — not just mapping accuracy

## Acceptance criteria

- [ ] `MappingDecision` produced for every run with all fields populated
- [ ] Policy engine resolves ALLOW/REVIEW/REJECT deterministically with no LLM call in the decision path
- [ ] Every human-resolved REVIEW produces a complete audit record (who/what/when/decision/reason/trace_id)
- [ ] A REVIEW cannot be marked resolved without an explicit authenticated approval
- [ ] Eval suite includes guardrail-specific test cases with category-level pass/fail
- [ ] A deliberately regressed guardrail case blocks the PR from merging on the GitHub-hosted CI runner
