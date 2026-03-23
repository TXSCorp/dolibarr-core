# TXS Corp — Dolibarr Core Fork

**Dolibarr Version:** 22.0.2  
**Maintained by:** TXS Corp Infrastructure  
**Repository:** https://github.com/TXSCorp/dolibarr-core

This repository tracks the TXS Corp fork of Dolibarr 22.0.2 core.
All modifications to Dolibarr core files are documented below.
Custom TXS modules live in a separate repo: [TXSCorp/dolibarr-custom](https://github.com/TXSCorp/dolibarr-custom).

---

## Branching Strategy

| Branch | Server | Purpose |
|--------|--------|---------|
| `txs-dev` | Dev EC2 (10.1.10.74) | Active development |
| `txs-staging` | txs-dev-staging (10.1.10.85) | Pre-production testing |
| `txs-production` | txs-prod-dolibarr (10.0.10.137) | Live production |
| `develop` | *(upstream only)* | Dolibarr upstream — never deploy from this |
| `main` | *(upstream only)* | Dolibarr upstream releases |

**Change flow:** `txs-dev` → PR → `txs-staging` → PR → `txs-production`

---

## Modified Core Files

All TXS Corp modifications to Dolibarr 22.0.2 core are documented in:
👉 [TXS-CORE-CHANGES.md](TXS-CORE-CHANGES.md)

---

## Upgrade Procedure

When upgrading Dolibarr to a new version:

1. Review every entry in [TXS-CORE-CHANGES.md](TXS-CORE-CHANGES.md) and diff each modified file against the new version
2. Re-apply TXS Corp changes manually where needed
3. Test on `txs-dev` first
4. Promote through `txs-staging` before merging to `txs-production`

---

## Related Repositories

| Repo | Purpose |
|------|---------|
| [TXSCorp/dolibarr-core](https://github.com/TXSCorp/dolibarr-core) | This repo — core fork |
| [TXSCorp/dolibarr-custom](https://github.com/TXSCorp/dolibarr-custom) | 24 custom TXS modules |
| [TXSCorp/sylius-custom](https://github.com/TXSCorp/sylius-custom) | Sylius e-commerce customizations |
