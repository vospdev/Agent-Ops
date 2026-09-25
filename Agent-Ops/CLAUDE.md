# Agent Ops Platform — Project Memory

This file is read automatically at the start of every Claude Code session in
this repo. It exists so we don't have to re-explain the architecture from
scratch each time. Keep it accurate as decisions change; keep it short.

## What this project is

A portfolio-grade AI agent platform for enterprise architecture job search
(Eric Vosper, Senior Software Developer, Trinity Church Wall Street). Two
pieces:

1. **Agent Ops Console** — Next.js/React console for run history, a
   guardrail approve/reject queue, and a schema diff viewer. Deployed on
   Vercel.
2. **Integration Copilot** — a real agent that proposes field mappings and
   transformation logic between two data schemas (an actual enterprise
   systems problem). Python/FastAPI, deployed on a DigitalOcean droplet.

## Active build target: Cloud (DigitalOcean)

There are two architecture variants in the planning doc. **Cloud is the one
we are building.** Self-Hosted (Mac mini) is a reference/alternate design
only — do not build against it unless explicitly told the plan changed.

Why Cloud: Vercel serverless functions can't cleanly reach a Mac mini behind
NAT without Tailscale gymnastics. A DigitalOcean droplet with a real public
IP removes that problem entirely.

## Tech stack

| Layer | Choice |
|---|---|
| Console | Next.js / React, deployed on Vercel |
| Domain data | Neon Postgres (mapping history, guardrail decisions, audit records, agent registry) |
| Agent runtime | Python / FastAPI / Pydantic, on the droplet |
| Tool interface | MCP — Integration Copilot is the **MCP client**; a standalone MCP server exposes the tools (see below) |
| Model | Claude via Anthropic API — invoked only for ambiguity resolution and transformation drafting, never for deterministic parsing/validation |
| Observability | Langfuse Cloud (free tier, 50K obs/month) — no self-hosted ClickHouse/Postgres |
| Ingress / TLS | Caddy reverse proxy + Let's Encrypt on a real DNS record (e.g. `api.yourdomain.com`) |
| Infra | DigitalOcean droplet, 2GB RAM / 1 vCPU ($12/mo), Ubuntu 24.04 LTS, 2–4GB swapfile |
| CI | GitHub-hosted runners (`ubuntu-latest`) — unit tests, eval suite, lint. Blocks merge on eval regression. |
| CD | Self-hosted runner **on the droplet, deploy-only**, triggered only on merge to `main`: `git pull` + `docker compose up -d --build` |

**Why the CI/CD split matters**: PR evals must never run on the production
host (resource contention with the live app) and untrusted PR code must
never execute there either. Two separate workflow files, not one
conditional workflow — keep the boundary unambiguous.

**Disposable infrastructure framing**: the droplet is recreatable, not
backed up. Real state lives in GitHub (source/config), Neon (domain data),
Langfuse (traces), and an external secrets store. DigitalOcean is
disposable runtime only.

## MCP topology (resolved decision)

**Option A**: Integration Copilot is the MCP **client**. A standalone MCP
**server** exposes named tools with real input/output schemas:
`inspect_schema`, `propose_mapping`, `validate_mapping`, `detect_pii`,
`lookup_mapping_history`.

We deliberately did NOT choose Option B (Integration Copilot itself as an
MCP server reachable by external clients like Claude Desktop) — that adds
external-facing protocol surface we don't need in v1.

MCP is an explicit **learning objective** for this project (hands-on MCP
experience for job interviews), not purely a technical necessity. When in
doubt about scope, keep the MCP surface bounded to what's listed above.

## Deterministic vs. LLM pipeline (core principle)

Integration Copilot's pipeline: **deterministic parsing/normalization →
deterministic candidate matching (exact/type/name-similarity) → Claude
(ambiguity resolution + transformation drafting only) → deterministic
validation → Guardrail Policy Engine (ALLOW/REVIEW/REJECT)**.

The LLM never produces the final guardrail decision — that's pure logic
(thresholds + rules) consuming a structured `MappingDecision` object
(confidence, pii_detected, type_mismatch, transformation_risk, sensitivity,
review_required).

Async execution: `POST /runs` returns a `run_id` immediately; work happens
in the background; status via polling/SSE. This avoids Vercel's serverless
timeout (10–15s) on a 15–40s Claude mapping call.

## Repo layout

```
console/          # Next.js/React Agent Ops Console
agents/           # Integration Copilot (Python/FastAPI)
mcp-server/       # standalone MCP server (schema/mapping/validation tools)
infra/            # droplet provisioning, Docker Compose, Caddy config, CI/CD workflows
docs/backlog/     # one markdown file per backlog ticket
DEVLOG.md         # dev log — see "Dev Log discipline" below
```

## Backlog ticket format

Every ticket in `docs/backlog/` follows this structure:

```
## What
## Why
## Current behavior
## Expected behavior
## Technical context
## Acceptance criteria
```

Tickets are numbered in build order: `01-prerequisites.md` through
`07-guardrails-audit-trail.md`.

## Secrets convention

API keys and tokens go in `.env` files (gitignored) or a secrets manager —
**never hardcoded, never committed.**

## Dev Log discipline

Write an entry in `DEVLOG.md` right after a ticket's acceptance criteria
are fully checked (or at least weekly if a ticket is still in progress).
Template:

```
## [Date] — [Ticket/Component]

**Built:** what shipped, one or two sentences
**Skills/tools touched:** concrete tools, languages, patterns
**New vocabulary/concepts:** terms looked up or newly understood
**Decision made + why:** a real choice and its reasoning
**Lesson / gotcha:** what tripped you up, what you'd do differently
**Résumé-ready line:** one sentence, already phrased as a bullet point
```

## Claude Code session economy (how we work in this repo)

- **One session per backlog ticket.** Open `docs/backlog/0N-*.md`, work that
  ticket, close out. Don't carry the whole spec in context across tickets.
- This `CLAUDE.md` is the persistent memory — it should mean we never need
  to re-paste the full architecture spec into a session.
- Use **Plan Mode** before implementing anything nontrivial (design
  decisions, not boilerplate).
- Use `/compact` if a session's context fills up mid-ticket rather than
  pushing through with a bloated context window.
- Use subagents for noisy exploratory research (library docs, error
  investigation) — keep that out of the main working thread.
- Write the Dev Log entry immediately after a ticket's acceptance criteria
  are met, while context is still fresh — don't defer it to a separate
  reload session.
- `git commit` and PR descriptions in this repo carry attribution lines
  required by this Claude Code session — do not omit them.
