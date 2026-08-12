# 🔑 Bitkey Address Generator

> Derive Bitcoin receive addresses from your **Bitkey multisig descriptors** — ready for tax software, portfolio trackers, and offline audits.

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
[![Bitcoin](https://img.shields.io/badge/Bitcoin-Multisig-f7931a?logo=bitcoin&logoColor=white)](https://bitcoin.org)
[![Status](https://img.shields.io/badge/status-utility-informational)](#)

---

## Why this exists

[Bitkey](https://bitkey.world) is a self-custody Bitcoin wallet that uses a **2-of-3 multisig** setup. Tax tools and portfolio trackers often need a plain list of addresses (or an xpub/descriptor export) to match on-chain activity — but working from raw multisig descriptors is awkward.

This project targets that gap: **turn Bitkey multisig descriptors into importable Bitcoin addresses** so you can reconcile activity in software like Koinly, CoinTracker, or custom spreadsheets without relying on a single hot-wallet export.

---

## What it is for

| Use case | How this helps |
|----------|----------------|
| **Crypto tax software** | Import address lists so buys, sells, and transfers can be matched on-chain |
| **Portfolio tracking** | Watch balances without keeping the full wallet software open |
| **Audits / bookkeeping** | Produce a deterministic address set from the wallet descriptors |
| **Privacy-conscious ops** | Work from descriptors offline; no need to re-enter seed material into tax apps |

---

## Conceptual flow

```text
Bitkey multisig descriptors
        │
        ▼
  Parse & validate descriptors
        │
        ▼
  Derive address range (gap limit / index range)
        │
        ▼
  Export addresses (CSV / text) for tax & tracking tools
```

Typical inputs are the **multisig output descriptors** Bitkey (or related recovery tooling) can surface. Outputs are standard Bitcoin addresses suitable for watch-only import.

---

## Design goals

- **Descriptor-first** — work from multisig descriptors, not seed phrases pasted into random tools
- **Tax-software friendly** — address lists that bulk-import cleanly
- **Offline-friendly** — derivation should not require a network or third-party custodian
- **Explicit ranges** — control how many addresses you generate (receive / change paths as applicable)

---

## Security notes

> **Treat descriptors and derived address data carefully.**

- Multisig descriptors can reveal wallet structure and watchable balances.
- Prefer generating addresses on an offline or trusted machine.
- Never commit real descriptors, seeds, or private keys to git.
- This repo should only ever hold **example/placeholder** material in docs.

---

## Repository status

This repository currently holds project scaffolding (license + docs). A practical MVP for the generator is:

1. Accept one or more descriptors via file or CLI flag  
2. Derive addresses for a configurable index range  
3. Write `addresses.csv` (`index`, `path`, `address`)

Common building blocks for that pipeline include Bitcoin Core `deriveaddresses`, BDK, or similar descriptor-aware libraries.

---

## Related

- [Bitkey](https://bitkey.world) — self-custody multisig wallet  
- [Bitcoin descriptors (BIPs)](https://github.com/bitcoin/bips) — output script descriptors  
- Tax importers that accept address lists or CSVs (Koinly, CoinTracker, etc.)

---

## License

Distributed under the terms of the **GNU General Public License v3.0**. See [LICENSE](LICENSE) for details.

---

<p align="center">
  <sub>Built for Bitkey users who need clean address exports for taxes and tracking — not another place to store seed phrases.</sub>
</p>
