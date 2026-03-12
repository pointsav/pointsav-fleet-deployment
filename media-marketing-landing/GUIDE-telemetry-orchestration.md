# 🧭 GUIDE: OS-MEDIAKIT TELEMETRY ORCHESTRATION
**Operational Tier:** 3 (Fleet Deployment)
**Standard:** DS-ADR-06 (Zero-Cookie Architecture)

---

## I. EXECUTIVE SUMMARY
This guide outlines the operational deployment of the Sovereign Telemetry Engine (`os-mediakit`) within the Customer's isolated cloud infrastructure. 

This deployment satisfies the strict Institutional Mandate to capture highly accurate geographic intelligence from digital marketing assets without deploying third-party tracking cookies, violating user privacy, or relinquishing corporate data to centralized analytics monopolies.

## II. TOTEBOX ORCHESTRATION
The telemetry architecture is deployed using a strict "Totebox" methodology. The underlying engineering (written in Rust by the Vendor, PointSav Digital Systems™) is completely abstracted away from the Customer's daily operations.

The operational Totebox consists of three distinct phases:
1. **The Edge Beacon:** A mathematically precise, 15-line vanilla JavaScript payload injected into the `index.html` of the public-facing websites.
2. **The Cloud Relay:** An isolated background daemon running on the Tier-2 Cloud Node that strictly validates incoming JSON payloads and maps IP addresses to Metro Regions using a localized, offline `.mmdb` database.
3. **The Air-Gapped Extraction:** A cron-scheduled "Pull Diode" that securely downloads the immutable `.csv` ledgers to a localized Tier-3 terminal, instantly synthesizing the data into human-readable Markdown reports.

## III. INTELLIGENCE METRICS CAPTURED
By enforcing strict data hygiene, the `os-mediakit` captures only the data required for macro-level institutional auditing, completely bypassing the need for Personally Identifiable Information (PII).

* **Time Matrix:** Captures the exact ISO 8601 timestamp of every unique page load, allowing for accurate mapping of marketing velocity.
* **Metro Region Matrix:** Translates network routing data into broad physical geographies (e.g., Vancouver, New York, London) to verify international market penetration without tracking individual user coordinates.

## IV. DISASTER RECOVERY
The Tier-2 Cloud Node does not utilize fragile database clusters. All data is appended to flat `.csv` ledgers. The automated Pull Diode strictly enforces a **10-Day Rolling Retention Policy**. Every 24 hours, the local extraction script securely downloads the current ledger and mathematically purges any backup files older than 10 days, ensuring perfect data hygiene and rapid recovery capabilities.

## V. LICENSING
Refer to the `LICENSE` file within this directory. This deployment architecture remains under an **Incubation Phase**, with all rights strictly reserved by Woodfine Capital Projects Inc.
