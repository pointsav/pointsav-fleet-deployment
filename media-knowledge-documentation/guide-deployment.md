---
schema: foundry-runbook-v1
component: media-knowledge-documentation
audience: workspace-master
canonical_language: en
status: active
runbook_version: 0.1.0
last_updated: 2026-04-26
---

# guide-deployment — media-knowledge-documentation

Operational runbook for provisioning a numbered
`media-knowledge-documentation` instance. Read
`guide-provision-node.md` first; node provisioning (system user,
binary, systemd unit) is a prerequisite for this guide.

This guide covers the instance layer: creating a numbered instance
directory under `~/Foundry/deployments/`, connecting it to a content
tree, starting the service, and verifying it is live.

---

## Prerequisites

- Node provisioning complete per `guide-provision-node.md`:
  - `local-knowledge` system user exists
  - `/var/lib/local-knowledge/` directory exists and is owned by
    `local-knowledge:local-knowledge`
  - `/usr/local/bin/app-mediakit-knowledge` binary is installed
  - `/etc/systemd/system/local-knowledge.service` unit is installed
  - `systemctl daemon-reload` has been run
- A content tree is available at a known path on this host (for the
  PointSav instance, this is the `content-wiki-documentation` checkout
  under `~/Foundry/vendor/content-wiki-documentation/`)
- Operator has `sudo` access on the host

---

## Step 1 — Determine instance number

List existing instances to find the next available number:

```
ls ~/Foundry/deployments/ | grep media-knowledge-documentation
```

If no instances exist, start at `1`. If `media-knowledge-documentation-1`
exists, use `2`, and so on.

---

## Step 2 — Create the instance directory

```
INSTANCE=media-knowledge-documentation-1
mkdir -p ~/Foundry/deployments/$INSTANCE
```

---

## Step 3 — Create the instance MANIFEST.md

Copy the deployment template and fill in the instance-specific fields:

```
cp ~/Foundry/templates/deployment-MANIFEST.md.tmpl \
   ~/Foundry/deployments/$INSTANCE/MANIFEST.md
```

Edit the MANIFEST.md:

- `deployment`: `media-knowledge-documentation-1`
- `guide`: `pointsav-fleet-deployment/media-knowledge-documentation`
- `tenant`: `pointsav`
- `purpose`: `local-dev` (or `production` when the instance is the
  canonical live deployment)
- `host`: `foundry-workspace`
- `source_version`: the version of `app-mediakit-knowledge` installed
  (check `app-mediakit-knowledge --version`)
- `state`: `active`

---

## Step 4 — Identify the content directory

For the PointSav instance, the content directory is the
`content-wiki-documentation` checkout:

```
CONTENT_DIR=~/Foundry/vendor/content-wiki-documentation
```

Verify the directory exists and contains Markdown files:

```
ls $CONTENT_DIR/*.md | head -5
```

The engine serves whatever directory `--content-dir` points at. The
content directory is not modified by the engine.

---

## Step 5 — Create the state directory

The engine writes derived state (search index, KV store) to a
separate state directory that is not tracked in Git:

```
sudo mkdir -p /var/lib/local-knowledge/state
sudo chown local-knowledge:local-knowledge /var/lib/local-knowledge/state
```

---

## Step 6 — Configure the systemd unit for this instance

The unit installed by `guide-provision-node.md` uses a drop-in file
to specify the content and state directories. Create an override for
the content path:

```
sudo systemctl edit local-knowledge.service
```

Add the following to the override file (this opens in your default
editor; save and close when done):

```ini
[Service]
ExecStart=
ExecStart=/usr/local/bin/app-mediakit-knowledge serve \
    --content-dir /home/mathew/Foundry/vendor/content-wiki-documentation \
    --state-dir /var/lib/local-knowledge/state \
    --bind 127.0.0.1:9090
```

The empty `ExecStart=` line clears the base unit's ExecStart before
setting the new value. Replace the `--content-dir` path if this is not
the PointSav instance.

Reload the daemon:

```
sudo systemctl daemon-reload
```

---

## Step 7 — Start the service

```
sudo systemctl enable --now local-knowledge.service
```

Check status:

```
systemctl status local-knowledge.service
journalctl -u local-knowledge.service -f
```

The binary logs its bind address and content directory on startup.
Wait for a line similar to:

```
listening on 127.0.0.1:9090
```

---

## Step 8 — Verify the service is responding

Health check:

```
curl -s http://127.0.0.1:9090/healthz
```

Expected output: `ok`

List pages:

```
curl -s http://127.0.0.1:9090/
```

Expected: an index of `.md` files found in the content directory.

Render a specific page (substitute a slug that exists in the content
tree):

```
curl -s http://127.0.0.1:9090/wiki/topic-hello | head -40
```

---

## Step 9 — Record the instance in MANIFEST.md

Update the instance MANIFEST.md Lifecycle table with the
provisioning date and the `app-mediakit-knowledge` version:

```
| 2026-04-26 | Provisioned by task-cluster-project-knowledge |
```

---

## Stopping and disabling the service

```
sudo systemctl stop local-knowledge.service
sudo systemctl disable local-knowledge.service
```

Update the instance MANIFEST.md `state` field to `inactive` and add
a Lifecycle entry.

---

## Logs and observability

- Live log stream: `journalctl -u local-knowledge.service -f`
- Recent errors: `journalctl -u local-knowledge.service -p err -n 50`
- Service status: `systemctl status local-knowledge.service`

---

## Port

Default bind: `127.0.0.1:9090`. Override via `--bind` flag in the
systemd unit ExecStart line. The loopback-only default is intentional;
public exposure requires a reverse proxy that the operator configures
separately.

---

## Cross-references

- `guide-provision-node.md` — prerequisite: system user, binary, unit
- `app-mediakit-knowledge/ARCHITECTURE.md` §4 — systemd unit shape
- `app-mediakit-knowledge/ARCHITECTURE.md` §5 — data layout on disk
- `conventions/zero-container-runtime.md` — no Docker; systemd binary
  deployments
