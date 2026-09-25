# 03 — GitHub Monorepo Setup

## What

Create the GitHub repo with the `console/`, `agents/`, `mcp-server/`,
`infra/` layout, and set up strictly separated CI (tests + eval suite on
GitHub-hosted runners) and CD (deploy on the droplet's self-hosted runner).

## Why

Avoids PR evals starving the live app of CPU/RAM, and avoids untrusted PR
code ever executing on the production host.

## Current behavior

Local directory skeleton exists (`console/`, `agents/`, `mcp-server/`,
`infra/`, `docs/backlog/`) with git initialized but no commits and no
remote.

## Expected behavior

Repo exists on GitHub with the four-folder layout, two separate CI/CD
workflow files enforcing the runner boundary, Vercel connected for
console auto-deploy, a Projects board, and `DEVLOG.md` committed at the
root.

## Technical context

- PR workflow: `runs-on: ubuntu-latest` — unit tests, eval harness, lint /
  security checks. Fails the PR on eval regression.
- Deploy workflow: `runs-on: self-hosted` — triggered **only** on merge to
  `main`. Runs `git pull` + `docker compose up -d --build` only.
- **Two separate workflow files**, not one conditional workflow — keeps the
  runner boundary unambiguous.
- Vercel connected directly to the repo for console auto-deploy on merge
  to `main`.
- GitHub Issues + one Projects board, columns by build phase.
- `DEVLOG.md` committed at repo root with the entry template (see
  `07-guardrails-audit-trail.md`'s sibling ticket note below and
  `CLAUDE.md`'s Dev Log discipline section).

## Acceptance criteria

- [ ] Repo created with `console/`, `agents/`, `mcp-server/`, `infra/`, `docs/backlog/` layout
- [ ] PR workflow runs on a GitHub-hosted runner and blocks merge on eval regression
- [ ] Deploy workflow runs only on the self-hosted runner, only post-merge to `main`
- [ ] Confirmed a PR's eval run never touches the droplet
- [ ] Vercel auto-deploys the console on merge to `main`
- [ ] Projects board created, columns by build phase
- [ ] `DEVLOG.md` exists in the repo root with the entry template committed — an entry gets added when this ticket's acceptance criteria are met
