---
schema: foundry-doc-v1
document_version: 0.1.0
deployment_name: media-knowledge-documentation
catalog_tier: pointsav-fleet-deployment
source_crate: app-mediakit-knowledge
last_updated: 2026-04-26
cites:
  - ni-51-102
  - osc-sn-51-721
---

# media-knowledge-documentation

Catalog entry for the PointSav knowledge wiki deployment shape. Each
instance of this name runs a single `app-mediakit-knowledge` binary,
serving a Markdown content tree as a Wikipedia-shaped read-and-edit
surface. The binary binds to the loopback interface by default; public
exposure requires a separate reverse-proxy step, which is an operator
decision outside this catalog entry.

This entry is the catalog definition — it describes what
`media-knowledge-documentation` instances are for and how to operate
them. Numbered runtime instances live under
`~/Foundry/deployments/media-knowledge-documentation-N/` and are
gitignored.

## What this deployment shape provides

- A single statically-linked Rust binary (`app-mediakit-knowledge`)
  serves a directory of CommonMark-with-wikilinks files
- Content is canonical on disk and in Git; every database and search
  index is derived state rebuildable from the file tree
- Loopback bind (`127.0.0.1:9090`) by default; no external network
  exposure without an explicit reverse-proxy configuration
- Single-tenant per instance: one binary, one content directory, one
  operator

## Source

Source crate: `app-mediakit-knowledge` in `pointsav-monorepo`.
Architecture: `app-mediakit-knowledge/ARCHITECTURE.md`.

## Operational guides

- [guide-provision-node.md](guide-provision-node.md) — provision the
  system user, directories, binary, and systemd unit on the host
- [guide-deployment.md](guide-deployment.md) — provision a numbered
  instance directory and connect it to a content tree

## Disclosure posture

All artifacts in this catalog entry are maintained under the
continuous-disclosure posture at
`conventions/bcsc-disclosure-posture.md`. Forward-looking statements
(planned phases, intended capabilities) carry "planned" or "intended"
language, a stated reasonable basis, and material assumptions.
Operational descriptions in this catalog entry reflect the current
shipped state of `app-mediakit-knowledge` Phase 1.
