# 01 — Prerequisite Accounts and Installs

## What

Confirm or create every external account and API key needed before any
infrastructure or code work starts, and confirm where secrets will live.

## Why

Nothing downstream (droplet setup, monorepo CI/CD, MCP server, Integration
Copilot) can be verified end-to-end without these in place. Doing this
first avoids getting blocked mid-ticket on an account signup or key
generation.

## Current behavior

No accounts or keys have been confirmed as set up for this project.

## Expected behavior

All required accounts exist, all required API keys are generated, and a
secrets storage approach is decided (`.env` files, gitignored, or a secrets
manager) before any ticket that needs them is started.

## Technical context

Accounts needed:

- **GitHub** — source control, Actions (CI/CD)
- **Vercel** — console hosting/deploy
- **Neon** — Postgres (domain data)
- **Anthropic Console** — API key for Claude
- **DigitalOcean** — droplet hosting (note: as of July 15, 2026, new
  accounts get a $5 credit for 90 days — not the earlier $200/60-day
  credit. Confirm whichever is actually available on the account and
  apply it; it no longer meaningfully offsets the $12/mo droplet cost)
- **Langfuse Cloud** — observability (free tier, 50K observations/month) —
  generate API keys

Secrets convention: all keys go in `.env` files (gitignored) or a secrets
manager — never hardcoded, never committed.

## Acceptance criteria

- [x] GitHub account confirmed
- [x] Vercel account confirmed
- [x] Neon account confirmed, project created
- [x] Anthropic Console account confirmed, API key generated
- [x] DigitalOcean account confirmed, $5/90-day signup credit applied
- [x] Langfuse Cloud account confirmed, API keys generated
- [x] Secrets storage approach decided and documented (`.env` gitignored, or secrets manager)
