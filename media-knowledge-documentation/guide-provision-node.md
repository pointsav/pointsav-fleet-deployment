---
schema: foundry-runbook-v1
component: media-knowledge-documentation
audience: workspace-master
canonical_language: en
status: active
runbook_version: 0.1.0
last_updated: 2026-04-26
---

# guide-provision-node — media-knowledge-documentation

Node provisioning runbook for the `app-mediakit-knowledge` wiki
engine on the workspace VM. This guide installs the system user,
directories, binary, and systemd unit. It does not start a numbered
instance or connect a content tree — that is `guide-deployment.md`.

The on-disk layout and systemd hardening pattern mirror
`infrastructure/local-slm/` (bootstrap.sh + local-slm.service), with
the substitutions appropriate to the wiki engine.

---

## Prerequisites

- Operator has `sudo` access on the host
- `systemd` is available (`systemctl --version`)
- A compiled `app-mediakit-knowledge` binary is available (produced by
  `cargo build --release` inside the `app-mediakit-knowledge` crate,
  or supplied by a future release artefact)
- The binary path is known; this guide uses
  `~/Foundry/clones/project-knowledge/pointsav-monorepo/app-mediakit-knowledge/target/release/app-mediakit-knowledge`
  as the build output path

---

## On-disk layout

After this guide completes, the following paths exist on the host:

| Path | Owner | Contents |
|---|---|---|
| `/usr/local/bin/app-mediakit-knowledge` | root | installed binary |
| `/var/lib/local-knowledge/` | `local-knowledge` | system user home; data root |
| `/var/lib/local-knowledge/state/` | `local-knowledge` | derived state (search index, KV store); not Git-tracked |
| `/etc/systemd/system/local-knowledge.service` | root | systemd unit |

Content directories (`content-wiki-documentation` and equivalents)
live outside this layout — they are Git-tracked separately and
supplied to the binary at start time via `--content-dir`.

---

## Step 1 — Create the system user

The wiki engine runs as a dedicated `local-knowledge` system user with
no login shell:

```
sudo useradd \
    --system \
    --create-home \
    --home-dir /var/lib/local-knowledge \
    --shell /usr/sbin/nologin \
    local-knowledge
```

Verify:

```
id local-knowledge
```

Expected: a line showing the `local-knowledge` user and group. If the
user already exists, the `useradd` command exits with a non-zero code;
that is safe — skip to Step 2.

---

## Step 2 — Create the data directory

```
sudo mkdir -p /var/lib/local-knowledge
sudo chown -R local-knowledge:local-knowledge /var/lib/local-knowledge
```

---

## Step 3 — Install the binary

Build the release binary if not already built:

```
cd ~/Foundry/clones/project-knowledge/pointsav-monorepo/app-mediakit-knowledge
cargo build --release
```

Install to `/usr/local/bin/`:

```
sudo install -m 755 \
    ~/Foundry/clones/project-knowledge/pointsav-monorepo/app-mediakit-knowledge/target/release/app-mediakit-knowledge \
    /usr/local/bin/app-mediakit-knowledge
```

Verify:

```
/usr/local/bin/app-mediakit-knowledge --version
```

---

## Step 4 — Install the systemd unit

Create `/etc/systemd/system/local-knowledge.service` with the
following content. The `--content-dir` and `--state-dir` values are
placeholders; the actual paths are supplied via a systemd override in
`guide-deployment.md` when a numbered instance is provisioned.

```ini
[Unit]
Description=PointSav Knowledge Wiki (app-mediakit-knowledge)
Documentation=file:///srv/foundry/clones/project-knowledge/pointsav-fleet-deployment/media-knowledge-documentation/guide-deployment.md
After=network.target

[Service]
Type=simple
User=local-knowledge
Group=local-knowledge

WorkingDirectory=/var/lib/local-knowledge

Environment="HOST=127.0.0.1"
Environment="PORT=9090"

# Content and state directories are supplied via a systemd override
# (see guide-deployment.md Step 6). This ExecStart is a placeholder
# that fails fast if no override is present — intentional.
ExecStart=/usr/local/bin/app-mediakit-knowledge serve \
    --content-dir /var/lib/local-knowledge/content \
    --state-dir /var/lib/local-knowledge/state \
    --bind ${HOST}:${PORT}

Restart=on-failure
RestartSec=5s

# Hardening — mirrors local-slm.service pattern
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
ReadWritePaths=/var/lib/local-knowledge

# journald captures stdout/stderr
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Write the unit file:

```
sudo tee /etc/systemd/system/local-knowledge.service > /dev/null << 'EOF'
[Unit]
Description=PointSav Knowledge Wiki (app-mediakit-knowledge)
Documentation=file:///srv/foundry/clones/project-knowledge/pointsav-fleet-deployment/media-knowledge-documentation/guide-deployment.md
After=network.target

[Service]
Type=simple
User=local-knowledge
Group=local-knowledge

WorkingDirectory=/var/lib/local-knowledge

Environment="HOST=127.0.0.1"
Environment="PORT=9090"

ExecStart=/usr/local/bin/app-mediakit-knowledge serve \
    --content-dir /var/lib/local-knowledge/content \
    --state-dir /var/lib/local-knowledge/state \
    --bind ${HOST}:${PORT}

Restart=on-failure
RestartSec=5s

NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
ReadWritePaths=/var/lib/local-knowledge

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF
```

Reload systemd:

```
sudo systemctl daemon-reload
```

---

## Step 5 — Verify unit is registered

```
systemctl cat local-knowledge.service
```

The unit should print without error. Do not start it yet — the
content directory override in `guide-deployment.md` Step 6 must be
applied first.

---

## What this guide does NOT do

- Does not start or enable the service. That is `guide-deployment.md`
  Step 7.
- Does not open any firewall port. The service binds to `127.0.0.1`
  only; no external exposure.
- Does not configure a reverse proxy.
- Does not download or provision a content tree. Content trees are
  Git-tracked separately; the operator supplies the path at deploy
  time.

---

## Updating the binary

To replace the binary with a newer build:

```
sudo systemctl stop local-knowledge.service

sudo install -m 755 \
    ~/Foundry/clones/project-knowledge/pointsav-monorepo/app-mediakit-knowledge/target/release/app-mediakit-knowledge \
    /usr/local/bin/app-mediakit-knowledge

sudo systemctl start local-knowledge.service
systemctl status local-knowledge.service
```

The system user, data directory, and unit file remain unchanged.
Updating the binary is a binary swap only.

---

## Removing the node

To fully remove the node provisioning:

```
sudo systemctl stop local-knowledge.service
sudo systemctl disable local-knowledge.service
sudo rm /etc/systemd/system/local-knowledge.service
sudo systemctl daemon-reload
sudo rm /usr/local/bin/app-mediakit-knowledge
sudo userdel local-knowledge
sudo rm -rf /var/lib/local-knowledge
```

Content trees (e.g. `content-wiki-documentation`) are not touched by
this removal sequence — they are separate Git-tracked repos.

---

## Cross-references

- `guide-deployment.md` — next step: provision a numbered instance
- `app-mediakit-knowledge/ARCHITECTURE.md` §4 — process model and
  systemd unit shape
- `infrastructure/local-slm/bootstrap.sh` — precedent bootstrap
  pattern (local-slm system user, /var/lib layout, binary install,
  unit install)
- `infrastructure/local-slm/local-slm.service` — precedent unit file
- `conventions/zero-container-runtime.md` — no Docker; systemd binary
  deployments
