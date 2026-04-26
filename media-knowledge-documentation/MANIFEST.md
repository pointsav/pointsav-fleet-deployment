---
schema: foundry-manifest-v1
tier: catalog
deployment_name: media-knowledge-documentation
guide: pointsav-fleet-deployment/media-knowledge-documentation
source_crate: app-mediakit-knowledge
source_repo: pointsav-monorepo
tenant_variations:
  - pointsav (PointSav documentation instance — content-wiki-documentation)
created: 2026-04-26
created_by: task-cluster-project-knowledge
state: active
doctrine_version: 0.0.1
source_version: 0.1.0
---

# media-knowledge-documentation — catalog declaration

This is the catalog-tier manifest for the `media-knowledge-documentation`
deployment shape. It declares what this deployment name means, what
source crate produces it, and what tenant variations are planned. It
is not a numbered instance record — numbered instances declare
themselves in their own MANIFEST.md under
`~/Foundry/deployments/media-knowledge-documentation-N/`.

## Purpose

Each `media-knowledge-documentation` instance serves a single Markdown
content tree (a `content-wiki-*` Git checkout or equivalent) as a
Wikipedia-shaped wiki via the `app-mediakit-knowledge` binary. The
engine is content-tree-agnostic; the content directory is supplied at
start time via `--content-dir`.

Default bind: `127.0.0.1:9090`. Public exposure requires a reverse
proxy; that is an operator decision and is not part of this catalog
entry.

## Source crate

`app-mediakit-knowledge` in `pointsav-monorepo`. Phase 1 shipped
2026-04-26 (commit `722ae18`). Architecture documented at
`pointsav-monorepo/app-mediakit-knowledge/ARCHITECTURE.md`.

## Subordinate components

Each instance runs:

- `app-mediakit-knowledge` binary — HTTP server (axum), Markdown
  renderer (comrak), embedded static assets (rust-embed)
- systemd unit `local-knowledge.service` — runs as dedicated
  `local-knowledge` system user
- Content directory — a Git-tracked Markdown tree supplied at deploy
  time (separate from the binary; not owned by this catalog entry)

## Tenant variations

| Tenant | Content tree | Instance naming | State |
|---|---|---|---|
| pointsav | `content-wiki-documentation` checkout | `media-knowledge-documentation-1` | planned |

Additional tenant instances (if any) follow the same pattern:
one binary per tenant, one content directory per instance, numbered
sequentially.

## Lifecycle

| Date | Event |
|---|---|
| 2026-04-26 | Catalog entry created; source crate Phase 1 shipped |
