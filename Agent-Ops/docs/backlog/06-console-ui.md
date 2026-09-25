# 06 — Agent Ops Console UI

## What

Build the Next.js/React console scoped to run history, the guardrail
approve/reject queue, and a schema diff viewer, with Langfuse Cloud
deep-links for full trace detail rather than a rebuilt drill-down.

## Why

Reimplementing Langfuse's trace/cost/eval-trend UI is weeks of
low-differentiation work. The console's real job is the domain-specific
approval workflow.

## Current behavior

No console UI exists.

## Expected behavior

A deployed console shows run history, a working guardrail queue tied to
real authenticated approvals, and a schema diff viewer — with the full
schema-upload-to-decision flow working end-to-end in the live deployment.

## Technical context

- Next.js on Vercel — server components for the run list, client
  components with real hooks for the guardrail queue/polling/SSE
- Run history pulls minimal summary data (status/cost/duration) from
  Langfuse Cloud's API and links out for full trace detail — no custom
  drill-down built
- Neon Postgres for domain state (guardrail decisions, approval audit
  records, mapping history)
- Lightweight auth (NextAuth/Auth.js with a single OAuth allowlist, or a
  scoped admin session) so an approval ties to a real authenticated
  identity
- Console calls Integration Copilot's async API (`POST /runs`,
  polling/SSE) — **never** calls the MCP server directly

## Acceptance criteria

- [ ] Run list shows status/timestamp/duration with links to the full Langfuse trace
- [ ] Schema diff viewer clearly shows source/target fields and the proposed mapping
- [ ] Guardrail queue shows flagged runs with working approve/reject actions
- [ ] An approval/rejection is tied to an authenticated human identity, not anonymous
- [ ] Full run (schema upload → mapping proposal → guardrail decision) works end-to-end in the live deployment
