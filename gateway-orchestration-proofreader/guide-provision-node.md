# Provision Node Guide — gateway-orchestration-proofreader

VM-level prerequisites for hosting the proofreader workflow gateway.

## VM specification (Tier 0 compatible)

- e2-small or larger (1 vCPU, 1-2 GB RAM minimum)
- 30 GB pd-balanced disk
- us-west1-a (current foundry-workspace zone) or operator-chosen region
- nginx + certbot installed at provision time
- Docker installed (LanguageTool runs as Docker container)

## System dependencies

- `docker` and `docker-compose` for LanguageTool container
- `python3` for any helper scripts

## Pre-flight checks

- DNS A record `proofreader.pointsav.com` resolves to the VM's public IP
- Firewall allows inbound 80/443 from operator-permitted CIDRs
- LanguageTool Docker image pulled and ready
- Doorman (`local-doorman.service`) available locally OR network path to a foundry-workspace Doorman if running on a separate publishing VM

## Bring-up smoke test

After bring-up per `guide-deployment.md`:

```bash
curl -fsS https://proofreader.pointsav.com/healthz
curl -fsS https://proofreader.pointsav.com/  # should return the proofreader UI
```

---

*Copyright © 2026 PointSav Digital Systems. All rights reserved.*

*Woodfine Capital Projects™, Woodfine Management Corp™, PointSav Digital Systems™, Totebox Orchestration™, and Totebox Archive™ are trademarks of Woodfine Capital Projects Inc., used in Canada, the United States, Latin America, and Europe. All other trademarks are the property of their respective owners.*
