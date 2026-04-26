# Cleanup Log — pointsav-fleet-deployment

Living record of in-flight cleanup work, open questions, and
decisions made during active development. Read at session start;
update at session end when meaningful cleanup occurs. Maintained
in-repo so the history travels with the code.

---

## How this file is maintained

- **Read at session start** — Claude Code reads this file at the
  start of every session per the workspace `CLAUDE.md`. The tables
  below reflect the current state of in-flight work.
- **Update at session end** — append a dated entry to the top of
  the **Session entries** section when a session includes
  meaningful cleanup. Trivial edits do not belong here.
- **Commit each update with the code changes it describes** so
  log and work travel together through git history.

---

## Open questions

*(none yet — first session entry below)*

---

## Session entries

Newest at top.

---

## 2026-04-26 — `media-knowledge-documentation` catalog activated (project-knowledge cluster session 3)

Catalog entry expanded from Reserved-folder (3 files) to Active
(8 files) for `app-mediakit-knowledge` Phase 1 (commit `722ae18`
in pointsav-monorepo). Files added or modified under
`media-knowledge-documentation/`:

- `README.md` — rewritten; "Sovereign Disclosure Standard"
  reference removed; replaced with BCSC-grounded posture per
  `conventions/bcsc-disclosure-posture.md`. Frontmatter `cites:`
  added.
- `README.es.md` — new; Spanish overview (strategic-adaptation
  style per workspace CLAUDE.md §6, not 1:1 translation).
- `MANIFEST.md` — new; catalog-tier (not instance-tier)
  declaration per workspace CLAUDE.md §10. Adapted from
  `~/Foundry/templates/deployment-MANIFEST.md.tmpl` with `tier:
  catalog` field added to distinguish from a numbered-instance
  MANIFEST.
- `guide-deployment.md` — expanded from placeholder to nine-step
  procedure (instance directory → MANIFEST fill → content-tree
  identification → state-dir → systemd override → enable →
  verify via curl on /healthz and /).
- `guide-provision-node.md` — rewritten to mirror
  `infrastructure/local-slm/` precedent: dedicated
  `local-knowledge` system user, `/var/lib/local-knowledge/` data
  root, `/usr/local/bin/app-mediakit-knowledge` binary,
  `/etc/systemd/system/local-knowledge.service` unit with
  identical hardening flags (`NoNewPrivileges`, `ProtectSystem=
  strict`, `ProtectHome`, `PrivateTmp`, `ReadWritePaths`).

Project-registry row updated: Reserved-folder → Active.

**Open questions surfaced (carried forward from Sonnet sub-agent's
report — operator review needed at next session):**

1. **Content-dir path for the PointSav instance.** The
   `guide-deployment.md` Step 6 systemd override hard-codes
   `/home/mathew/Foundry/vendor/content-wiki-documentation`. The
   workspace path on this VM may also be
   `/srv/foundry/vendor/content-wiki-documentation` depending on
   operator's mount layout. Operator confirmation needed for the
   canonical path; the guide is a template until ratified.
2. **`ProtectHome=true` vs home-directory content paths.** If
   the content-tree lives under `/home/mathew/Foundry/...`, the
   systemd unit's `ProtectHome=true` blocks read access. Either
   move the content-tree to `/var/lib/local-knowledge/content/`
   (or symlink from there), or relax `ProtectHome` to `read-only`.
   Operator decision.
3. **`bootstrap.sh` parallel to `infrastructure/local-slm/`.**
   Sonnet sub-agent suggested wrapping the provision steps in an
   idempotent shell script for one-command provisioning. Reasonable
   follow-up; deferred.
4. **Resource limits (`MemoryMax`, `CPUQuota`).** local-slm sets
   these for the 7B inference workload; the wiki engine's profile
   is unmeasured. Add after Phase 1 production memory measurement.

**Cluster context.** Commit produced under
`cluster/project-knowledge` branch from a Sonnet sub-agent in the
project-knowledge cluster session 3, reviewed and committed by
Opus Task Claude. L1 trajectory capture writes a corpus record to
`~/Foundry/data/training-corpus/engineering/project-knowledge/<sha>.jsonl`.

---

## 2026-04-26 — Cleanup log file initialised

This file (`.claude/rules/cleanup-log.md`) did not exist prior to
this commit. Initialised as part of the
`media-knowledge-documentation` catalog activation, following the
pattern used in pointsav-monorepo and content-wiki-documentation.
The repo's `.claude/rules/` directory previously held only
`project-registry.md`.
