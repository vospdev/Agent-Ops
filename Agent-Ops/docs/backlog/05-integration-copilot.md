# 05 — Integration Copilot Python Service

## What

Build the FastAPI service implementing the deterministic-parsing /
candidate-generation → LLM-reasoning (ambiguous cases + transformation
drafting only) → deterministic-validation → guardrail-policy-engine
pipeline, running asynchronously.

## Why

The LLM should only handle what deterministic code can't do reliably.
Running the pipeline asynchronously avoids hitting Vercel's serverless
timeout on a 15–40s mapping call.

## Current behavior

No Integration Copilot service exists.

## Expected behavior

A submitted mapping run returns a `run_id` immediately and completes in
the background. Claude is invoked only for ambiguity resolution and
transformation drafting. The Guardrail Policy Engine — not the LLM —
produces the final decision. Every run is traced.

## Technical context

- FastAPI + Pydantic
- Fuzzy-matching library for name-similarity candidate generation
- Claude via Anthropic API invoked **only** for ambiguity resolution and
  transformation drafting — never for parsing or validation
- Async execution: `POST /runs` returns a `run_id` immediately; work
  happens in the background; status delivered via polling or SSE
- Guardrail Policy Engine (not the LLM) produces the final
  ALLOW / REVIEW / REJECT decision
- Every run traced to Langfuse Cloud
- Registered as an MCP client per `04-mcp-server.md`
- Eval harness runs against a stored set of known schema pairs,
  categorized by failure type: exact match, semantic match, type
  mismatch, transformation, PII, hallucination, no valid mapping

## Acceptance criteria

- [ ] Deterministic parsing/candidate-generation runs with zero LLM calls
- [ ] Claude invoked only for ambiguity/transformation, confirmed via trace inspection
- [ ] A submitted run returns a `run_id` in under 1 second, with mapping completing asynchronously
- [ ] Guardrail Policy Engine (not the LLM) produces the final decision on every run
- [ ] ≥80% field-mapping accuracy on the eval set, with 0 PII false negatives
- [ ] Eval results broken out by category, not one aggregate score
- [ ] Every run traced in Langfuse Cloud
