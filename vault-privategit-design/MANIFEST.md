---
schema: foundry-fleet-catalog-v1
catalog_entry: vault-privategit-design
created: 2026-04-28
visibility: public-when-live
canonical_instance_path: ~/Foundry/deployments/vault-privategit-design-1/
canonical_instance_url: https://design.pointsav.com
doctrine_version: 0.0.11
doctrine_claim: 38
owning_cluster: project-design
substrate_engine_repo: pointsav-monorepo
substrate_engine_crate: app-privategit-design
substrate_engine_version_floor: 0.1.0
infrastructure_iac: /srv/foundry/infrastructure/local-design/
---

# Catalog entry — vault-privategit-design

This catalog entry describes the design-system substrate
deployment shape under Doctrine claim #38 (The Design System
Substrate). One catalog entry; multiple instances per tenant.

## What this deployment is

A self-hosted, customer-owned design-system substrate. Each instance
serves a single tenant from a vault directory containing DTCG-format
design tokens, HTML+CSS+ARIA component recipes, per-brand themes,
AI-readable design-decision research, and derived exports.

The substrate engine binary (`app-privategit-design`) reads the vault
from disk and exposes:

- A public HTML showcase at `/`
- A DTCG token bundle at `/tokens.json`
- An AI-readable research surface at `/research/<topic>` and `/research`
- A Model Context Protocol (MCP) JSON-RPC 2.0 server at `/mcp`
- Health and readiness probes at `/healthz`, `/readyz`

The engine is stateless above the vault; restarting the binary
re-reads the vault. Per-tenant isolation is achieved by running one
process per tenant, each pointed at its own vault directory.

## Per-tenant ownership

Per claim #38, each instance carries its own Git-tracked vault. The
customer always has the source. Migration cost falls toward zero —
the customer can lift the vault into any other infrastructure that
runs the substrate engine.

## Tenants

| Instance | Tenant | Path | URL | Status |
|---|---|---|---|---|
| vault-privategit-design-1 | pointsav | `/srv/foundry/deployments/vault-privategit-design-1/` | https://design.pointsav.com | First-iteration build pending bootstrap |
| vault-privategit-design-2 | woodfine | (when provisioned) | (TBD) | Declared for second-tenant fan-out; not yet provisioned |
| vault-privategit-design-N | (any SMB customer) | (their site) | (their domain) | Forks of this catalog |

Per the Tetrad Discipline (claim #37), the customer-leg fan-out is
declared structurally even when only the first instance is live —
visible structure replaces hidden assumption.

## Infrastructure

| Artefact | Location |
|---|---|
| systemd unit | `/srv/foundry/infrastructure/local-design/local-design.service` |
| nginx vhost | `/srv/foundry/infrastructure/local-design/nginx-design.conf` |
| Bootstrap installer | `/srv/foundry/infrastructure/local-design/bootstrap.sh` |
| Operator runbook | `GUIDE-deploy-design-substrate.md` (this folder) |

Bootstrap installs the binary at `/usr/local/bin/app-privategit-design`,
the systemd unit at `/etc/systemd/system/local-design.service`, and
the nginx vhost at `/etc/nginx/sites-enabled/design.pointsav.com`.

## Vault layout (canonical)

```
<deployment-instance>/
├── tokens/        — DTCG primitive + semantic + component layers
├── components/    — HTML+CSS+ARIA recipes
├── themes/        — per-brand semantic-layer overrides
├── research/      — AI-readable design-decision rationale (markdown)
└── exports/       — derived caches (Figma, Tailwind, Style Dictionary)
```

The substrate engine reads `tokens/*.json`, `components/*.json`,
`themes/*.json`, `research/*.md`. It does NOT recurse into
subdirectories — flat layout per layer.

## SMB customer fork procedure

The catalog entry IS the productized substrate. Any SMB customer can
fork:

1. `git clone github.com/pointsav/app-privategit-design <smb-name>-design`
2. Provision their vault from the canonical template at
   `pointsav-design-system/dtcg-vault/` (Carbon-baselined primitives,
   `pointsav-brand` as starting theme — customers replace with their
   brand).
3. Run `bootstrap.sh` (with their own systemd / nginx adapters; the
   installer is parameterised per environment).
4. Provision DNS + TLS (certbot or equivalent).
5. Live at `design.<smb-name>.com`.

Every step is documented in `GUIDE-deploy-design-substrate.md`. No
SaaS dependency; no hyperscaler runtime; full Git-tracked ownership.

## Trustworthy System Attestation (TSA)

Inherits claim #36's pattern. Quarterly report cites:
- Sigstore Rekor anchoring root for vault checkpoints
- Per-tenant SSH-signed `allowed_signers` chain
- WORM append-only ledger property (via service-fs)
- Customer-rooted Merkle log apex per claim #33

For the canonical PointSav showcase at `design.pointsav.com`, Master
signs. For SMB customer instances, the customer's substrate operator
signs.

## References

- Doctrine claim #38 — `~/Foundry/DOCTRINE.md` §III row 38
- Convention — `~/Foundry/conventions/design-system-substrate.md`
- Cluster manifest — `~/Foundry/clones/project-design/.claude/manifest.md`
- Substrate engine source — `pointsav-monorepo/app-privategit-design/`
- Canonical vault template — `pointsav-design-system/dtcg-vault/`
- Infrastructure IaC — `~/Foundry/infrastructure/local-design/`
