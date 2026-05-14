# CLAUDE.md — pointsav-fleet-deployment

> This is the repo-level guide. It complements `~/Foundry/CLAUDE.md` (the
> workspace guide, which takes precedence on all shared rules).
>
> Cap: 150 lines per workspace `CLAUDE.md` §2.

Last updated: 2026-05-14.

---

## 1. What this repo is

`pointsav-fleet-deployment` is the **catalog tier** for PointSav fleet
deployments. It holds operational GUIDEs, MANIFEST declarations, and
bilingual READMEs for each deployment shape — not feature code. Every
directory here represents a deployment *shape* (the how-to catalog), not
a running *instance*.

Instances (numbered deployments like `media-knowledge-documentation-1`)
live under `~/Foundry/deployments/` — local-only, gitignored.

## 2. Who writes here

Totebox Sessions via the staging-tier commit flow. This repo is part of
the `project-knowledge` cluster
(`~/Foundry/clones/project-knowledge/pointsav-fleet-deployment/`). All
commits alternate between `jwoodfine` and `pwoodfine` identities using
`~/Foundry/bin/commit-as-next.sh`.

Admin-only changes (LICENSE propagation, factory-release-engineering
updates) arrive via `mcorp-administrator` from workspace level — not from
Totebox sessions.

## 3. Repo structure

```
pointsav-fleet-deployment/
├── CLAUDE.md                         this file
├── NEXT.md                           repo-scope open items
├── README.md / README.es.md          bilingual repo overview
├── INVENTORY.yaml                    machine-readable catalog index
├── media-knowledge-distribution/     Reserved-folder
├── media-knowledge-documentation/    Active — documentation.pointsav.com
├── media-marketing-landing/          Scaffold-coded
├── vault-privategit-design-system/   Reserved-folder
└── vault-privategit-source/          Reserved-folder
```

## 4. What belongs in each deployment directory

Per workspace `CLAUDE.md` §10, a deployment catalog directory must contain:

| File | Required | Notes |
|---|---|---|
| `README.md` | Yes | English description of deployment shape |
| `README.es.md` | Yes | Spanish pair (bilingual per workspace §6) |
| `MANIFEST.md` | Yes (Active) | Catalog-tier declaration (`tier: catalog`) |
| `guide-deployment.md` | Yes (Active) | Step-by-step provision/deploy runbook |
| `guide-provision-node.md` | Recommended | Node bootstrap and systemd unit setup |

No feature code, no binaries, no compiled artefacts. Those belong in
`pointsav-monorepo`.

## 5. Commit flow

- `git add <specific files>` — never `git add .` or `-A`
- `~/Foundry/bin/commit-as-next.sh "<message>"` — handles J/P alternation + SSH signing
- Push deferred to Stage 6 / `bin/promote.sh`

**Not permitted** without operator approval: `--amend` on pushed commits,
`--hard` reset, force-push.

## 6. Local rules

| File | Purpose |
|---|---|
| `.agent/rules/project-registry.md` | Inventory of all 5 deployment directories and their states |
| `.agent/rules/cleanup-log.md` | In-flight decisions and open questions |

## 7. Where to look next

- `NEXT.md` — open items for this repo
- `~/Foundry/CLAUDE.md` — workspace guide (inherited; takes precedence)
- `~/Foundry/IT_SUPPORT_Nomenclature_Matrix_V8.md` §4 — deployment prefix taxonomy
- `conventions/root-files-discipline.md` — allowed files at repo root
