# 🔑 Bitkey Address Generator

> Derive **Native SegWit multisig addresses** from your Bitkey-style `sortedmulti` descriptors, scan balances on your own Bitcoin node, and export a tax-friendly Excel workbook.

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Bitcoin](https://img.shields.io/badge/Bitcoin-P2WSH%20Multisig-f7931a?logo=bitcoin&logoColor=white)](https://bitcoin.org)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
[![RPC](https://img.shields.io/badge/Bitcoin%20Core-JSON--RPC-informational)](https://developer.bitcoin.org/reference/rpc/)

---

## Why this exists

[Bitkey](https://bitkey.world) is a **2-of-3 multisig** self-custody wallet. Tax software and portfolio trackers often want a list of **receive (and change) addresses** — but working from raw multisig descriptors is painful.

This tool:

1. **Parses** external + internal `sortedmulti` descriptors  
2. **Derives** Bech32 (`bc1…`) P2WSH addresses offline  
3. Optionally **scans balances** with Bitcoin Core `scantxoutset`  
4. Optionally **imports watch-only descriptors** and summarizes recent txs  
5. **Exports Excel** ready for bookkeeping / tax imports  

No seed phrases. No private keys. Descriptors + a node you control.

---

## Features

| Feature | Details |
|---------|---------|
| 🧩 **Descriptor parsing** | Bitkey-style `wsh(sortedmulti(...))` external & change |
| 📬 **Address derivation** | BIP32 child keys → sorted-pubkey P2WSH → Bech32 |
| 🔍 **Balance scan** | `scantxoutset` over derived addresses (no wallet required for balances) |
| 📒 **Watch-only wallet** | Optional descriptor wallet + `listtransactions` summary |
| 📊 **Excel export** | Sheets: External, Internal, Transaction Summary |
| 🧪 **Dry run** | Derive & print addresses with zero RPC / zero files |
| 🔐 **Env-based secrets** | RPC credentials stay in `.env` (gitignored) |

---

## How it works

```text
wallet_descriptors.txt          .env (RPC creds)
         │                            │
         ▼                            ▼
  parse External / Internal     Bitcoin Core node
  sortedmulti descriptors              │
         │                             │
         ▼                             │
  BIP32 derive m/0/i & m/1/i           │
  → P2WSH → bc1… addresses             │
         │                             │
         ├──── scantxoutset ───────────┤
         ├──── importdescriptors ──────┤  (optional)
         └──── listtransactions ───────┘
                        │
                        ▼
                 wallet_data.xlsx
          (External / Internal / Tx Summary)
```

---

## Project layout

```text
bitkey_address_generator/
├── generate_wallet_data.py   # Main CLI
├── requirements.txt
├── .env.example              # Template only — copy to .env
├── .gitignore                # Blocks .env, *.txt, *.xlsx, venv
├── LICENSE
└── README.md
```

> **Not in git (by design):** `.env`, `wallet_descriptors.txt`, `wallet_data.xlsx`, and any other local secrets or outputs.

---

## Prerequisites

- **Python 3.10+**
- Your **Bitkey multisig descriptors** (external + internal)
- Optional but recommended: a **Bitcoin Core** (or Umbrel / similar) node with RPC enabled for balances & history

---

## Setup

```bash
git clone https://github.com/vinxp97/bitkey_address_generator.git
cd bitkey_address_generator

python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS / Linux:
source .venv/bin/activate

pip install -r requirements.txt
cp .env.example .env
```

Edit `.env` with your RPC settings (never commit this file).

---

## Descriptor file format

Create a local file (default name `wallet_descriptors.txt` — **gitignored**):

```text
External: wsh(sortedmulti(2,[FINGERPRINT/path]xpub.../0/*,[...],[...]))#checksum
Internal: wsh(sortedmulti(2,[FINGERPRINT/path]xpub.../1/*,[...],[...]))#checksum
```

- **External** = receive chain  
- **Internal** = change chain  

Lines are matched case-insensitively on the `External:` / `Internal:` prefixes.

---

## How to use

### 1. Dry run (offline address check)

No node, no Excel — just validate derivation:

```bash
python generate_wallet_data.py --dry-run -i wallet_descriptors.txt -n 10
```

Prints indexed external and change addresses to the terminal.

### 2. Full run (balances + Excel)

```bash
python generate_wallet_data.py \
  -i wallet_descriptors.txt \
  -o wallet_data.xlsx \
  --node-url 127.0.0.1:8332 \
  --rpc-user myuser \
  --rpc-password mypass \
  -n 20
```

Or rely entirely on `.env` defaults:

```bash
python generate_wallet_data.py
```

### 3. Balances only (skip tx sheet)

```bash
python generate_wallet_data.py --skip-transactions
```

### 4. Verbose debug

```bash
python generate_wallet_data.py --dry-run -v
```

---

## CLI reference

| Flag | Env var | Default | Meaning |
|------|---------|---------|---------|
| `-i` / `--input` | `INPUT_FILE` | `wallet_descriptors.txt` | Descriptor file |
| `-o` / `--output` | `OUTPUT_FILE` | `wallet_data.xlsx` | Excel path |
| `--node-url` | `NODE_URL` | `127.0.0.1:8332` | RPC host:port |
| `--rpc-user` | `RPC_USER` | _(empty)_ | RPC username |
| `--rpc-password` | `RPC_PASSWORD` | _(empty)_ | RPC password |
| `-n` / `--count` | `ADDRESS_COUNT` | `20` | Addresses per chain |
| `--wallet-name` | `WALLET_NAME` | `bitkey_watchonly` | Watch-only wallet name |
| `--dry-run` | — | off | Derive only |
| `--skip-transactions` | — | off | Skip wallet import / tx sheet |
| `-v` / `--verbose` | — | off | Debug logs |

---

## Excel output

| Sheet | Contents |
|-------|----------|
| **External Addresses** | Index, address, balance (BTC) |
| **Internal Addresses** | Index, change address, balance (BTC) |
| **Transaction Summary** | TXID, amount, inputs/outputs (tagged), change flag, confirmations, time |

Use the address columns for tax software import or portfolio watch lists.

---

## Security notes

> Treat descriptors as **sensitive watch-only material**.

- Descriptors reveal wallet structure and allow balance watching  
- Prefer running on a machine you trust; keep `.env` private  
- This repo’s `.gitignore` blocks `.env`, `*.txt` (except `requirements.txt`), and `*.xlsx`  
- **Never** commit real descriptors, RPC passwords, or generated workbooks  
- The tool is **watch-only** — it does not handle private keys or signing  

---

## Troubleshooting

| Symptom | What to try |
|---------|-------------|
| `Descriptor format is incorrect` | Confirm `sortedmulti(...)` and `External:` / `Internal:` labels |
| RPC auth errors | Check `RPC_USER` / `RPC_PASSWORD` / `NODE_URL` (Umbrel often uses `umbrel.local:8332`) |
| `scantxoutset` busy / fails | Script aborts prior scans; wait and retry; use `-v` |
| Empty transaction sheet | Node wallet may need a full rescan; try `--skip-transactions` for balances only |
| Bech32 / derivation mismatch | Verify you used the **current** Bitkey export and external vs internal lines aren’t swapped |

---

## Dependencies

| Package | Role |
|---------|------|
| `bip32utils` | HD key child derivation from xpubs |
| `bech32` | Bech32 address encoding |
| `bitcoinlib` | Bitcoin Core JSON-RPC client (`AuthServiceProxy`) |
| `openpyxl` | Excel export |
| `python-dotenv` | Load `.env` |

---

## Disclaimer

This is a personal utility for address export and node-assisted reporting. It is **not** tax advice, not an official Bitkey product, and not a substitute for verifying addresses inside your hardware wallet UI before sending funds.

---

## License

Distributed under the **GNU General Public License v3.0**. See [LICENSE](LICENSE) for details.

---

<p align="center">
  <sub>Bitkey multisig descriptors → bc1 addresses → balances on your node → Excel for taxes.</sub>
</p>
