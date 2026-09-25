# Dev Log

This log tracks what's actually been built, what was learned, and why
decisions were made — so that when it's time to update the résumé, the
skills/vocabulary/decisions are already written down instead of needing to
be reconstructed from memory.

**Cadence:** write an entry after a backlog ticket's acceptance criteria
are fully checked, or at least weekly if a ticket is still in progress.
Every ticket in the Self-Hosted and Cloud backlogs should produce at least
one entry here before it's considered done — treat "Dev log entry written"
as a habit alongside acceptance criteria, not a separate ticket of its own.

**Entry template:**

```
## [Date] — [Ticket/Component]

**Built:** what shipped, one or two sentences
**Skills/tools touched:** concrete tools, languages, patterns
**New vocabulary/concepts:** terms looked up or newly understood
**Decision made + why:** a real choice and its reasoning
**Lesson / gotcha:** what tripped you up, what you'd do differently
**Résumé-ready line:** one sentence, already phrased as a bullet point
```

---

## Sept 22, 2026 — Mac Mini Homelab Concept

**Built:** Scoped the initial idea of repurposing an old Mac mini (Late
2014, dual-core i7, 16GB RAM) as a self-hosted home lab server for an AI
agent platform portfolio project, rather than defaulting straight to cloud
hosting.

**Skills/tools touched:** hardware inventory assessment, home networking
basics, self-hosting tradeoff analysis.

**New vocabulary/concepts:** home lab, self-hosted vs. managed
infrastructure, NAT traversal problem (network address translation
blocking inbound connections to a home device).

**Decision made + why:** Started with the self-hosted route because the
hardware was already owned (no monthly cost) and because running the full
stack locally would be a deeper infrastructure learning experience than
just deploying to a managed cloud platform.

**Lesson / gotcha:** Didn't yet know that a home network's NAT would
become a real architectural blocker for a cloud-hosted frontend (Vercel)
trying to reach this machine — that surfaced later.

**Résumé-ready line:** Evaluated self-hosted vs. managed cloud
infrastructure tradeoffs for a production-style AI agent deployment,
including hardware capacity planning.

---

## Sept 22, 2026 — Agent Ops Platform Architecture Spec (Self-Hosted)

**Built:** Wrote the first full requirements/architecture spec for the
"Agent Ops Console + Integration Copilot" project — executive summary,
system architecture, component specs (Console, Integration Copilot, MCP
Server, Mac mini infrastructure layer), agent template pattern, version
control/deployment plan, dev workflow, tech stack summary, cost and
timeline.

**Skills/tools touched:** technical requirements writing, system
architecture diagramming, enterprise AI agent design patterns (guardrails,
observability, evaluation).

**New vocabulary/concepts:** MCP (Model Context Protocol), Langfuse
(LLM observability platform), agent template pattern, human-in-the-loop
guardrails, eval harness.

**Decision made + why:** Structured the project as two complementary
pieces — a reusable Agent Ops platform and a concrete Integration Copilot
agent built on it — specifically to demonstrate architecture judgment
(not just "wrapped an LLM call in a UI") for AI-focused enterprise
architecture job applications.

**Lesson / gotcha:** Writing the spec before touching code forced a lot of
scope decisions (what's in v1, what's a stretch goal) that would have been
much messier to make mid-build.

**Résumé-ready line:** Authored a full requirements and architecture
specification for a production-style AI agent platform, defining
functional/non-functional requirements, success criteria, and scope
boundaries.

---

## Sept 22, 2026 — First Architecture Review Round

**Built:** Ran the self-hosted spec through architectural review and
incorporated feedback.

**Skills/tools touched:** critical review of my own architecture,
buy-vs-build tradeoff analysis.

**New vocabulary/concepts:** OOM (out-of-memory) risk, reverse proxy,
CI/CD runner isolation.

**Decision made + why:** Confirmed the "buy vs. build" call to integrate
Langfuse rather than build custom observability — validated as the right
call by outside review, reinforcing it as a piece of architectural
judgment worth calling out explicitly in the writeup.

**Lesson / gotcha:** Getting review feedback early (before implementation)
surfaced infrastructure risks — like resource contention between CI and a
live app — that would have been much more expensive to discover after
building.

**Résumé-ready line:** Incorporated structured architectural review
feedback into infrastructure and CI/CD design decisions before
implementation began.

---

## Sept 22, 2026 — Mac Mini Hardware Reality Check

**Built:** Assessed the actual Mac mini hardware specs (macOS Monterey
12.0.1, Late 2014, 3GHz dual-core i7, 16GB 1600MHz DDR3, spinning disk) and
decided what hardware changes would be needed to make it viable — an SSD
upgrade for boot/Docker performance, with the option deferred until an
actual load test showed it was necessary.

**Skills/tools touched:** hardware capacity planning, SSD vs. HDD
tradeoffs, boot-drive migration research, external SSD comparison shopping
(Samsung T7, Crucial X9, WD Elements, SanDisk).

**New vocabulary/concepts:** boot-from-external-SSD, NAND flash pricing
dynamics (2026 AI-datacenter-driven price spike affecting consumer SSD
prices).

**Decision made + why:** Made the hardware-optimization ticket explicitly
conditional — get it running first, evaluate for lag, only then decide
whether to invest in an SSD — instead of treating it as an unconditional
prerequisite. Avoids over-engineering hardware for a prototype before
knowing it's actually needed.

**Lesson / gotcha:** Live on-device retail pricing (checked directly on a
phone) was significantly higher than what web search returned for the
same SSDs — turned out to be a real, current NAND flash price spike, not
a bad listing. Lesson: trust live pricing over search-indexed pricing when
they disagree meaningfully.

**Résumé-ready line:** Performed hardware capacity assessment and
conditional upgrade planning for a self-hosted infrastructure prototype,
scoping optimization work to actual measured need rather than
speculation.

---

## Sept 22, 2026 — Pivot: Self-Hosted vs. Cloud

**Built:** Identified the core architectural flaw in the self-hosted
design — Vercel serverless functions can't cleanly participate in a
Tailscale mesh to reach a Mac mini behind home NAT — and designed a
parallel Cloud variant (DigitalOcean droplet with a real public IP) as the
new primary build target, while keeping the Self-Hosted design as a
reference/alternate architecture.

**Skills/tools touched:** infrastructure architecture pivoting,
NAT/networking troubleshooting reasoning, DigitalOcean platform research,
reverse-proxy and TLS design (Caddy + Let's Encrypt).

**New vocabulary/concepts:** NAT traversal, bearer token service auth vs.
human identity auth, disposable infrastructure pattern.

**Decision made + why:** Chose DigitalOcean over continuing to fight the
Mac mini's NAT problem because a real public IP eliminates an entire class
of networking complexity for a relatively small monthly cost ($12/mo,
largely covered by signup credit) — a clear case where "pay a little,
remove a hard problem" was the right call for a portfolio-scale prototype.

**Lesson / gotcha:** Recognizing a networking constraint as a fundamental
architecture blocker (not a tweak) early avoided sinking more time into
Tailscale workarounds that would have been fragile at best.

**Résumé-ready line:** Diagnosed a fundamental networking constraint in a
self-hosted architecture and redesigned the deployment target around
managed cloud infrastructure to resolve it.

---

## Sept 22–24, 2026 — Second Review Round: MCP Resolution, Guardrails, CI/CD Split

**Built:** Rewrote the Cloud architecture spec end-to-end based on a
second round of architectural review: resolved MCP's topology (Integration
Copilot as MCP client, standalone MCP server — "Option A" over "Option
B"), upsized the droplet from 1GB to 2GB RAM with a swapfile, split CI
(GitHub-hosted runners) from CD (self-hosted, deploy-only), scoped the
console UI away from rebuilding Langfuse's trace UI, promoted the
guardrail policy engine to a first-class structured component, and
explicitly dropped a floated "code-generation agent" idea as out of scope.

**Skills/tools touched:** MCP client/server architecture, structured
policy engine design, CI/CD security boundary design, scope discipline
under review feedback.

**New vocabulary/concepts:** MCP client vs. MCP server roles,
`MappingDecision` object pattern, categorized eval reporting (per-failure-
type pass/fail instead of one aggregate score), agent template pattern
(deferred until after a reference implementation exists).

**Decision made + why:** Chose MCP "Option A" (agent-as-client, standalone
tool server) over "Option B" (agent-as-external-facing-MCP-server)
specifically to avoid unnecessary external protocol surface in v1, while
still getting hands-on MCP experience — explicitly framed as a job-
interview-relevant learning objective, not just a technical necessity.
Also explicitly declined to fold a "generates code from a mockup" agent
idea into the platform, since it has no clean eval boundary and executes/
deploys LLM-generated code — a fundamentally larger risk class than field
mapping.

**Lesson / gotcha:** Multiple independent reviews flagging the same droplet
sizing risk (1GB → OOM) was a good signal to actually act on it rather than
treat it as a nitpick — when review feedback converges from different
angles, that's a stronger signal than any single critique.

**Résumé-ready line:** Resolved a Model Context Protocol (MCP) topology
ambiguity by adopting an agent-as-client architecture with a standalone
MCP tool server, and split CI/CD across isolated runner types to protect
a live production deployment from PR-time evaluation workloads.

---

## Sept 24, 2026 — Dev Log Established

**Built:** Designed and backfilled a lightweight, git-tracked `DEVLOG.md`
(mirrored as a doc tab) for capturing skills, vocabulary, decisions, and
résumé-ready lines as the project progresses — deliberately rejecting a
database-backed logging endpoint in favor of a plain markdown file that
gets version-controlled for free.

**Skills/tools touched:** technical writing discipline, self-documentation
process design, git-based knowledge tracking.

**New vocabulary/concepts:** none new — this ticket was about process, not
a new technical concept.

**Decision made + why:** Chose a git-tracked markdown file over a custom
logging endpoint/database to avoid building unnecessary infrastructure for
a documentation habit — and tied the discipline structurally to the
GitHub monorepo setup ticket's acceptance criteria so it isn't purely
aspirational.

**Lesson / gotcha:** The easiest way to make a documentation habit stick
is to attach it to an existing checkpoint (a ticket's acceptance criteria)
rather than trust it to happen as a standalone habit.

**Résumé-ready line:** Established a git-tracked development log process
tied to ticket acceptance criteria, capturing decisions and skills gained
throughout an infrastructure build for reuse in technical writeups.

---

## Sept 24, 2026 — Ticket 01: Prerequisite Accounts and Installs

**Built:** Confirmed all six external accounts needed before infrastructure
or code work starts: GitHub, Vercel, Neon, Anthropic Console (API key),
DigitalOcean, and Langfuse Cloud (API keys generated). Decided the secrets
storage approach (`.env`, gitignored) and committed a `.gitignore` to the
repo enforcing it.

**Skills/tools touched:** GitHub Desktop (repo clone/commit/push), DigitalOcean
billing/credits, Langfuse Cloud project and API key setup, GitHub fine-grained
personal access tokens.

**New vocabulary/concepts:** fine-grained GitHub PAT scoping (repository
and permission-level access control), egress/git proxy policy (a
sandboxed dev environment can restrict which repos it's authorized to
push to, independent of any token supplied).

**Decision made + why:** Pushed the initial repo scaffold via GitHub
Desktop instead of directly from the cloud dev session, after the
session's git proxy denied a direct push (the target repo wasn't yet in
its authorized-repository set) — a fine-grained PAT alone couldn't
override that policy layer, so the local-machine path was the pragmatic
unblock rather than chasing the session setting further that day.

**Lesson / gotcha:** DigitalOcean's signup credit changed as of July 15,
2026 — new accounts now get $5 for 90 days, not the $200/60-day credit
the original spec assumed. Caught and corrected in
`docs/backlog/01-prerequisites.md` before it caused confusion later; a
reminder that infra-vendor pricing/promo terms are worth re-verifying
right before acting on them, not just trusting an earlier spec.

**Résumé-ready line:** Stood up the full external-service account and
secrets foundation (GitHub, Vercel, Neon, Anthropic, DigitalOcean,
Langfuse Cloud) for a cloud-deployed AI agent platform, including
resolving a sandboxed dev environment's git egress policy restriction.
