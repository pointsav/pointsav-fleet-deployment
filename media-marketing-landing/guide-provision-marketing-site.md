# Provision Node Guide — Marketing Landing Site (PointSav tenant)

VM-level prerequisites for hosting the PointSav marketing landing site.

## VM specification (Tier 0 compatible)

- e2-small or larger (1 vCPU, 1-2 GB RAM minimum)
- 30 GB pd-balanced disk (binary + content + media uploads)
- us-west1-a (current foundry-workspace zone) or operator-chosen region
- nginx + certbot installed at provision time

## System dependencies

- `nginx` for TLS termination + reverse proxy
- `certbot` for Let's Encrypt automation
- No database server — flat-file content
- Rust binary is self-contained (no language runtime needed)

## Pre-flight checks

- DNS A record `home.pointsav.com` resolves to the VM's public IP
- Firewall allows inbound 80/443 from operator-permitted CIDRs
- service-content endpoint reachable

## Bring-up smoke test

```bash
curl -fsS https://home.pointsav.com/healthz
curl -fsS https://home.pointsav.com/
curl -fsS https://home.pointsav.com/wp-admin
```

## Multi-tenant cohabitation

When the Woodfine instance (`media-marketing-landing-1` per customer-fleet catalog) and the PointSav instance (`media-marketing-landing-2` — this catalog) run on the same VM, they share the binary at `/usr/local/bin/app-mediakit-marketing` but maintain isolated content directories, ports, and systemd units. Two nginx vhosts route `home.woodfinegroup.com` and `home.pointsav.com` separately.

---

*Copyright © 2026 PointSav Digital Systems. All rights reserved.*

*Woodfine Capital Projects™, Woodfine Management Corp™, PointSav Digital Systems™, Totebox Orchestration™, and Totebox Archive™ are trademarks of Woodfine Capital Projects Inc., used in Canada, the United States, Latin America, and Europe. All other trademarks are the property of their respective owners.*
