# 02 — DigitalOcean Droplet Setup

## What

Create the DigitalOcean droplet, harden SSH access, configure the firewall
and swapfile, install Docker and Caddy, and register a self-hosted GitHub
Actions runner scoped strictly to deployment.

## Why

This resolves the two biggest risks flagged in architecture review:
undersized RAM causing OOM crashes, and running PR evals on the same host
that serves the live app.

## Current behavior

No droplet exists.

## Expected behavior

A droplet is provisioned, secured, and running Docker + Caddy, with a
deploy-only self-hosted GitHub Actions runner registered and surviving
reboot.

## Technical context

- Basic droplet: **2GB RAM / 1 vCPU ($12/mo)**, Ubuntu 24.04 LTS
- **2–4GB swapfile** configured at provisioning time (backstop for memory
  spikes, not a substitute for the 2GB tier)
- SSH: key-based only, password auth disabled
- DigitalOcean Cloud Firewall: allow only ports **80, 443, 22**
- Docker Engine + Docker Compose installed via apt
- Caddy running as a reverse-proxy container; DNS record (e.g.
  `api.yourdomain.com`) pointed at the droplet's public IP for automatic
  Let's Encrypt SSL
- GitHub Actions self-hosted runner installed as a **systemd service**,
  labeled/scoped so PR CI workflows can never target it — deploy jobs only
- Bearer token generated for service-to-service auth (Vercel → droplet),
  stored in **both** GitHub Actions secrets and Vercel environment
  variables
- Tailscale optional, admin-only SSH access if used at all — not part of
  the Vercel→droplet path

## Acceptance criteria

- [ ] Droplet reachable via key-based SSH only (password auth disabled)
- [ ] Swapfile active (2–4GB)
- [ ] Firewall restricted to ports 80, 443, 22 only
- [ ] Docker Engine + Compose installed and verified (`docker compose version`)
- [ ] Caddy serving a valid Let's Encrypt certificate on the DNS record
- [ ] Self-hosted GitHub Actions runner registered, labeled deploy-only, survives reboot
- [ ] Bearer token generated and stored in both GitHub Actions secrets and Vercel env vars
