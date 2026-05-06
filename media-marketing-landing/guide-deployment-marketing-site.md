# Deployment Guide — Marketing Landing Site (PointSav tenant)

Covers the operation of the PointSav marketing landing site at `home.pointsav.com`. The active deployment instance is `media-marketing-landing-2` — PointSav vendor-tier tenant of `app-mediakit-marketing`.

This is PointSav's open public-reference deployment of its own marketing software. SMB customers may run their own private deployments on their own service-SLM nodes; this is PointSav running it as an open resource.

## Stack composition

- `app-mediakit-marketing` binary (compiled from `pointsav-monorepo/app-mediakit-marketing/`)
- Flat-file content directory (Markdown + YAML frontmatter; no database)
- service-content DataGraph integration (entity references on landing pages, optional)
- nginx vhost terminating TLS via Let's Encrypt
- systemd unit serving on a dedicated 909x port (see deployment instance MANIFEST)

## WordPress muscle-memory at the UX layer

The landing site presents WordPress-familiar navigation: `/wp-admin`-style Dashboard, Pages, Media, Themes, Settings. Content authors who know WordPress should not need to relearn. The Rust + flat-file architecture sits underneath — invisible to authors, but eliminates the PHP/MySQL operational burden + plugin sprawl.

## Bring-up sequence

1. Install binary at `/usr/local/bin/app-mediakit-marketing` (Master scope; operator-presence sudo).
2. Create `local-marketing-pointsav.service` systemd unit pointing at `~/Foundry/deployments/media-marketing-landing-2/`.
3. Configure environment:
   - `SERVICE_MARKETING_MODULE_ID=pointsav`
   - `SERVICE_MARKETING_CONTENT_DIR=/srv/marketing-pointsav/content/`
   - `SERVICE_MARKETING_BIND=127.0.0.1:910N` (different port from Woodfine instance)
4. Configure nginx vhost for `home.pointsav.com` reverse-proxying to local port.
5. Issue Let's Encrypt cert via certbot.
6. Smoke test: `curl https://home.pointsav.com/healthz`.

## Tier 0 compatibility

Marketing landing runs on a $7/mo Tier 0 node — same shape as the Woodfine tenant. Rust binary self-contained; no GPU; no AI required for the substrate; AI tier is additive when Tier C key is present.

## Content workflow

- Pages live as Markdown files in `content/pages/`
- Media uploads in `content/media/`
- Theme tokens in `themes/<theme-name>/tokens.css` (sourced from `pointsav-design-system`)
- Per-page entity references resolve via service-content DataGraph (`module_id=pointsav`)
- Page edits captured for training corpus via Doorman audit-log

## SMB deployment pattern

A customer wanting to run their own marketing landing site:

1. Provision their own Tier 0 VM
2. Install `app-mediakit-marketing` binary
3. Set `SERVICE_MARKETING_MODULE_ID=<their-tenant>` (e.g., `acme-holdings`)
4. Point their domain at the VM
5. Configure their own nginx + cert
6. Done — sovereign, no vendor lock-in, $7/mo

The PointSav public-reference deployment at `home.pointsav.com` demonstrates the pattern for prospective customers.

## When this site graduates to its own VM

Per `conventions/publishing-tier-architecture.md` per-site VM graduation: provision a dedicated VM, rsync the deployment instance folder, swap DNS. Path on the new VM stays `~/deployments/media-marketing-landing-2/` for clean lift.

---

*Copyright © 2026 PointSav Digital Systems. All rights reserved.*

*Woodfine Capital Projects™, Woodfine Management Corp™, PointSav Digital Systems™, Totebox Orchestration™, and Totebox Archive™ are trademarks of Woodfine Capital Projects Inc., used in Canada, the United States, Latin America, and Europe. All other trademarks are the property of their respective owners.*
