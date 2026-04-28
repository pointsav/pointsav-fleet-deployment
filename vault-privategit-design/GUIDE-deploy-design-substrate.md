# GUIDE — deploy the design-system substrate

Operator runbook for provisioning a `vault-privategit-design-N`
instance. Used to launch the canonical PointSav showcase at
`design.pointsav.com`; reusable for any SMB customer fork.

This guide is English-only per `~/Foundry/CLAUDE.md` §14
(operational, not for public bilingual distribution).

---

## Prerequisites

| Prerequisite | How to verify |
|---|---|
| Substrate engine source built | `ls /srv/foundry/clones/project-design/pointsav-monorepo/target/release/app-privategit-design` |
| Vault directory exists | `ls /srv/foundry/deployments/vault-privategit-design-1/` (or your tenant path) |
| systemd available | `which systemctl` |
| nginx available | `which nginx` |
| DNS resolves to this VM | `dig +short <your-domain>` returns the VM's external IP |
| certbot installed | `which certbot` (for TLS) |
| `local-fs.service` running (optional) | `systemctl is-active local-fs` (substrate degrades gracefully if absent) |

For the canonical PointSav showcase, the workspace VM
`foundry-workspace` (external IP `34.53.65.203`, zone `us-west1-a`)
satisfies all of the above as of 2026-04-28.

---

## Stage 1 — Build the substrate engine

```
cd /srv/foundry/clones/project-design/pointsav-monorepo
cargo build --release -p app-privategit-design
```

Verify:

```
ls -lh target/release/app-privategit-design
file target/release/app-privategit-design
```

The release binary is statically linked against the system C library
and the Rust standard library. Strip is optional;
`/usr/local/bin/app-privategit-design` is its install destination.

---

## Stage 2 — Populate the vault

The vault directory is the substrate's content. Layout:

```
<vault-root>/
├── tokens/        DTCG primitive + semantic + component layers (*.json)
├── components/    HTML+CSS+ARIA recipes (*.json)
├── themes/        per-brand semantic-layer overrides (*.json)
├── research/      AI-readable design-decision rationale (*.md)
└── exports/       derived caches (populated at build time)
```

For the canonical PointSav showcase, seed from the canonical
template at `/srv/foundry/clones/project-design/pointsav-design-system/dtcg-vault/`:

```
sudo install -d -o $(id -u) -g foundry /srv/foundry/deployments/vault-privategit-design-1/{tokens,components,themes,research,exports}
sudo cp /srv/foundry/clones/project-design/pointsav-design-system/dtcg-vault/tokens/*.json   /srv/foundry/deployments/vault-privategit-design-1/tokens/
sudo cp /srv/foundry/clones/project-design/pointsav-design-system/dtcg-vault/components/*.json /srv/foundry/deployments/vault-privategit-design-1/components/
sudo cp /srv/foundry/clones/project-design/pointsav-design-system/dtcg-vault/themes/*.json   /srv/foundry/deployments/vault-privategit-design-1/themes/
sudo cp /srv/foundry/clones/project-design/pointsav-design-system/dtcg-vault/research/*.md   /srv/foundry/deployments/vault-privategit-design-1/research/
```

For an SMB customer fork, replace the source directory with your
tenant's vault template. The canonical layout is the same; the
content differs by tenant.

---

## Stage 3 — Run the bootstrap installer

The substrate's IaC at `/srv/foundry/infrastructure/local-design/`
carries an idempotent installer:

```
sudo /srv/foundry/infrastructure/local-design/bootstrap.sh
```

This installs:

| Artefact | Path |
|---|---|
| Service user | `local-design` (system user; nologin shell) |
| Binary | `/usr/local/bin/app-privategit-design` |
| systemd unit | `/etc/systemd/system/local-design.service` |
| nginx vhost (HTTP-only baseline) | `/etc/nginx/sites-available/design.pointsav.com` |
| nginx symlink | `/etc/nginx/sites-enabled/design.pointsav.com` |

The installer is idempotent — re-running it after a binary rebuild
re-installs only the changed files. The smoke test at the end
verifies `/healthz` responds before exiting.

---

## Stage 4 — Verify HTTP

```
curl -sS http://127.0.0.1:9094/healthz | jq .
curl -sS http://127.0.0.1:9094/readyz  | jq .
curl -sS http://127.0.0.1:9094/tokens.json | jq '.primitive.color | keys'
curl -sS -X POST http://127.0.0.1:9094/mcp \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","method":"describe","id":1}' | jq .
```

Expected:

- `/healthz` returns 200 with `status: ok`
- `/readyz` returns 200 with `tokens_loaded: true` once vault has `tokens/*.json`
- `/tokens.json` returns the DTCG bundle
- `/mcp describe` returns the substrate's surface map

If `/readyz` returns 503, the vault directory is missing tokens.
Re-run Stage 2.

---

## Stage 5 — Issue TLS certificate

Once HTTP is responding via the public domain (DNS resolved), issue
a Let's Encrypt certificate via certbot:

```
sudo certbot --nginx -d design.pointsav.com \
    --non-interactive \
    --agree-tos \
    -m open.source@pointsav.com \
    --redirect
```

`--redirect` adds the HTTP-to-HTTPS 301 to the nginx config
automatically. Verify:

```
curl -I https://design.pointsav.com/healthz
```

Expected: `HTTP/2 200`. The certificate auto-renews via certbot's
systemd timer.

---

## Stage 6 — Verify the public showcase

```
curl -sSI https://design.pointsav.com/ | head
curl -sS https://design.pointsav.com/research | jq .
curl -sS https://design.pointsav.com/research/design-philosophy | head -40
```

The showcase HTML loads; the research index lists at least
`design-philosophy` and `carbon-baseline-rationale`; the philosophy
markdown renders.

---

## Stage 7 — Wire MCP into AI agents

The MCP endpoint is at `https://<your-domain>/mcp`. JSON-RPC 2.0
over HTTP POST. Methods exposed in v0.0.1:

| Method | Purpose |
|---|---|
| `describe` | Surface map (methods + tenant + version) |
| `list_tokens` | DTCG token bundle for the tenant |
| `list_components` | Component recipes (name, description, research link) |
| `list_research` | Research entries (topic, title, decision_type) |

To wire the substrate into an AI agent's context-loading pipeline,
register the endpoint as an MCP server. Claude Agent SDK pattern:

```python
# Pseudocode — adapt to your agent runtime
mcp_servers = [
    {"url": "https://design.pointsav.com/mcp", "name": "design-substrate"},
]
```

For Claude Desktop, add to the user's MCP config. For Vercel v0,
GitHub Copilot, and other AI-codegen tools, follow the tool's MCP
registration flow (these surface arrives in 2026 H1-H2).

---

## SMB customer fork procedure

A customer who wants their own design-system substrate forks at the
source level:

```
git clone github.com/pointsav/app-privategit-design <smb-name>-design
cd <smb-name>-design

# 1. Build the substrate engine
cd pointsav-monorepo
cargo build --release -p app-privategit-design

# 2. Initialise the vault from the template
mkdir -p /srv/<smb-name>/vault-privategit-design-1/{tokens,components,themes,research,exports}
cp -r ../pointsav-design-system/dtcg-vault/* /srv/<smb-name>/vault-privategit-design-1/

# 3. Edit themes/<smb-brand>.json — re-point semantic references at
#    the customer's brand colors / type / motion choices.

# 4. Run bootstrap (parameterised — see infrastructure/local-design/README.md
#    for the per-tenant adapter pattern)
sudo bootstrap.sh \
    --tenant <smb-name> \
    --vault-dir /srv/<smb-name>/vault-privategit-design-1 \
    --domain design.<smb-name>.com

# 5. Issue TLS
sudo certbot --nginx -d design.<smb-name>.com ...
```

The customer ends up with a fully self-hosted, customer-owned
design-system substrate. No SaaS dependency. No hyperscaler runtime.
Their `themes/<smb-brand>.json` and any research files they author
are their work product, in their Git repo, signed by their key.

---

## Decommissioning

To stop and remove an instance:

```
sudo systemctl disable --now local-design.service
sudo rm /etc/systemd/system/local-design.service
sudo systemctl daemon-reload
sudo rm /etc/nginx/sites-enabled/design.pointsav.com
sudo rm /etc/nginx/sites-available/design.pointsav.com
sudo nginx -t && sudo systemctl reload nginx
sudo userdel local-design
sudo rm -rf /var/lib/local-design /usr/local/bin/app-privategit-design
# vault directory is preserved unless explicitly removed
```

Per `~/Foundry/CLAUDE.md` §11 action matrix, decommissioning is
Master + Task work — Master audits the graceful tear-down; Task
owns the procedure for its own instance.

---

## Troubleshooting

### `/readyz` returns 503

Vault tokens not loaded. Inspect:

```
ls /srv/foundry/deployments/vault-privategit-design-1/tokens/*.json
sudo journalctl -u local-design.service -n 50
```

Likely causes: vault directory missing, no `*.json` files in
`tokens/`, JSON parse failure. The journal carries the specific
parse error.

### nginx returns 502 on `/`

Substrate engine not running or not bound:

```
sudo systemctl status local-design.service
sudo ss -tlnp | grep 9094
```

If the service is up but not bound, check `DESIGN_BIND` in the
systemd unit.

### certbot challenge fails

DNS not resolving to this VM. Verify:

```
dig +short design.pointsav.com
```

Returns the VM's external IP. If DNS is correct but the challenge
still fails, check that the ACME location block in the nginx vhost
serves `/var/www/letsencrypt/.well-known/acme-challenge/`:

```
sudo install -d -o root -g root /var/www/letsencrypt
```

### MCP endpoint returns method-not-found

The substrate's v0.0.1 surface ships four methods (`describe`,
`list_tokens`, `list_components`, `list_research`). Other methods
return JSON-RPC error -32601. Subsequent milestones expand the
surface; check the substrate engine's `src/mcp.rs` for the
authoritative list.

---

## References

- Substrate engine source — `pointsav-monorepo/app-privategit-design/`
- Substrate doctrine — `~/Foundry/DOCTRINE.md` §III row 38
- Convention — `~/Foundry/conventions/design-system-substrate.md`
- IaC — `~/Foundry/infrastructure/local-design/`
- Cluster manifest — `~/Foundry/clones/project-design/.claude/manifest.md`
- DTCG format — https://design-tokens.github.io/community-group/format/
- MCP spec — https://modelcontextprotocol.io/
- WCAG 2.2 — https://www.w3.org/TR/WCAG22/
