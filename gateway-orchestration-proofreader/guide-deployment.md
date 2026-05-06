# Deployment Guide — gateway-orchestration-proofreader

Covers the operation of the proofreader workflow gateway serving `proofreader.pointsav.com`. The active deployment instance is `gateway-orchestration-proofreader-1` on the workspace VM (today) or the publishing VM (per `conventions/publishing-tier-architecture.md` once provisioned).

## Stack composition

- `app-console-proofreader` binary (compiled from `pointsav-monorepo/`)
- LanguageTool Docker container (CPU-only grammar engine)
- Doorman (`local-doorman.service`) for AI-mediated calls (Tier 0 fallback: LanguageTool only; with Tier C key: AI rewrites)
- nginx vhost terminating TLS via Let's Encrypt
- systemd unit serving on `127.0.0.1:9091/9092` (or current allocated ports — see deployment instance MANIFEST)

## Bring-up sequence

1. Install binary at `/usr/local/bin/app-console-proofreader` (Master scope; operator-presence sudo).
2. Create `local-proofreader.service` systemd unit pointing at `~/Foundry/deployments/gateway-orchestration-proofreader-1/`.
3. Configure nginx vhost for `proofreader.pointsav.com` reverse-proxying to local ports.
4. Issue Let's Encrypt cert via certbot.
5. Smoke test: `curl https://proofreader.pointsav.com/healthz`.
6. Set `PROOFREADER_PASSWORD_HASH` env var when ready to lock down (currently dev-mode passthrough).

## Tier 0 compatibility

Proofreader runs on a $7/mo Tier 0 node:
- LanguageTool grammar checking is CPU-only (no GPU required)
- Doorman with no AI tier configured returns 503 for chat-completions; LanguageTool handles grammar on its own
- With Tier C key in Doorman config: AI-mediated rewrites become available
- Corpus capture for service-slm training works regardless of tier

## Per-call routing

- All inference calls transit Doorman per Doctrine claim #43 (Single-Boundary Compute Discipline)
- Audit ledger captures every prose-edit event as `event_type: prose-edit`
- Verdict-signing produces DPO pairs in `~/Foundry/data/training-corpus/apprenticeship/`

## Operational notes

- Authentication: dev-mode passthrough until operator seeds `PROOFREADER_PASSWORD_HASH`
- Monitoring: nginx access log + service-proofreader stdout journal
- Disk usage: minimal (no persistent state besides corpus tuples landing in workspace data dir)

## When this cluster scales beyond the shared host

Per `conventions/publishing-tier-architecture.md` per-site VM graduation: provision a dedicated VM, rsync the deployment instance folder, swap DNS. Path on the new VM stays `~/deployments/gateway-orchestration-proofreader-1/` for clean lift.

---

*Copyright © 2026 PointSav Digital Systems. All rights reserved.*

*Woodfine Capital Projects™, Woodfine Management Corp™, PointSav Digital Systems™, Totebox Orchestration™, and Totebox Archive™ are trademarks of Woodfine Capital Projects Inc., used in Canada, the United States, Latin America, and Europe. All other trademarks are the property of their respective owners.*
