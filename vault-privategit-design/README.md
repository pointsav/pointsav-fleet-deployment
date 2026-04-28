# vault-privategit-design

[ 🇪🇸 Leer este documento en Español ](./README.es.md)

Catalog entry for the design-system substrate deployment shape under
Doctrine claim #38. Each instance serves a single tenant from a Git-
tracked vault containing DTCG-format design tokens, HTML+CSS+ARIA
component recipes, per-brand themes, AI-readable design-decision
research, and derived exports.

The canonical PointSav instance runs at `design.pointsav.com`. Each
SMB customer who forks the substrate runs their own instance at
their own domain. Single codebase, single deployment shape, two
contexts (vendor showcase, customer instance).

## Contents

| File | Purpose |
|---|---|
| `MANIFEST.md` | Catalog-tier definition (instances, infrastructure, vault layout) |
| `GUIDE-deploy-design-substrate.md` | Operator runbook for the launch and fork procedure |
| `README.md` / `README.es.md` | This README pair |

## Live instance

| Field | Value |
|---|---|
| Tenant | pointsav |
| Path on workspace VM | `/srv/foundry/deployments/vault-privategit-design-1/` |
| Public URL (target) | https://design.pointsav.com |
| State | First-iteration build pending bootstrap |
| Source | `pointsav-monorepo/app-privategit-design` (workspace-canonical) |

## Why this catalog entry exists

Most design-system platforms host the customer's design system on
the vendor's infrastructure — Specify, Backlight, Knapsack, Tokens
Studio Pro, FIGMA's design-system MCP. The substrate inverts: the
customer's design system lives in the customer's Git repository,
runs on the customer's infrastructure, and exposes its content to
AI agents and design tools through standards (W3C DTCG, MCP) rather
than vendor-specific APIs.

The vendor showcase at `design.pointsav.com` IS the product SMB
customers self-host. The same binary serves both — there is no
separate "enterprise edition", no SaaS variant, no API-only flavour.
Customers fork, deploy, and own.

## How to use

For the runbook covering install, configure, certificate issuance,
and customer fork: see `GUIDE-deploy-design-substrate.md`.

For the substrate engine source and architecture details: see
`pointsav-monorepo/app-privategit-design/CLAUDE.md`.

For the doctrine claim and cross-cluster context: see
`~/Foundry/DOCTRINE.md` §III row 38 and
`~/Foundry/conventions/design-system-substrate.md`.
