# 💧 onchainos-spraay-batch — SPEC.md

## Overview

An OKX OnchainOS skill that gives any AI agent on the OKX.AI marketplace the ability to batch-send ERC-20 tokens and native currency to up to 200 recipients in a single transaction on Base — powered by the Spraay x402 Gateway.

**Repo:** `plagtech/onchainos-spraay-batch`
**Target platform:** OKX OnchainOS (AI Skills format)
**License:** MIT

---

## What This Skill Does

Exposes Spraay's live batch payment infrastructure as an OKX OnchainOS skill so any MCP-connected agent (Claude Code, Cursor, Codex, OpenClaw, Hermes) can:

1. **Estimate** the cost of a batch payment before committing
2. **Execute** a multi-recipient token transfer in one transaction
3. **Check balances** to verify sufficient funds before sending
4. **Look up prices** to convert between USD and token amounts

All endpoints are live on Base mainnet. Payment is x402 (USDC on Base) — no API keys, no accounts, no OKX credentials needed. The agent's EVM wallet pays per call automatically.

---

## Endpoints Wrapped

| Endpoint | Method | Price | Description |
|---|---|---|---|
| `gateway.spraay.app/api/v1/batch/execute` | POST | $0.02 | Batch send tokens to up to 200 recipients |
| `gateway.spraay.app/api/v1/batch/estimate` | POST | $0.001 | Estimate gas + fees before executing |
| `gateway.spraay.app/api/v1/balances` | GET | $0.005 | Check wallet balances (ERC-20 + native) |
| `gateway.spraay.app/api/v1/prices` | GET | $0.005 | Token prices (USDC, ETH, WETH, etc.) |

**Payment protocol:** x402 (`@x402/evm` exact scheme, USDC on Base)
**Batch contract:** `0x1646452F98E36A3c9Cfc3eDD8868221E207B5eEC` (Base mainnet)
**Network:** Base (Chain ID 8453)

---

## Skill File Structure

Follow the OKX OnchainOS skill conventions from `okx/onchainos-skills` and `okx/agent-skills`:

```
onchainos-spraay-batch/
├── SPEC.md                          # This file
├── README.md                        # Overview, install, usage examples
├── LICENSE                          # MIT
├── skills/
│   └── spraay-batch/
│       └── SKILL.md                 # Main skill file (YAML frontmatter + Markdown body)
├── references/
│   ├── batch-execute.md             # Detailed endpoint doc: /batch/execute
│   ├── batch-estimate.md            # Detailed endpoint doc: /batch/estimate
│   ├── balances.md                  # Detailed endpoint doc: /balances
│   ├── prices.md                    # Detailed endpoint doc: /prices
│   └── x402-payment.md             # How x402 payment works (for agent context)
└── examples/
    ├── payroll.md                   # Example: pay 10 contributors
    ├── airdrop.md                   # Example: distribute tokens to 100 wallets
    └── treasury.md                  # Example: DAO treasury disbursement
```

---

## SKILL.md Specification

The main `skills/spraay-batch/SKILL.md` must follow this format:

### YAML Frontmatter

```yaml
---
name: spraay-batch
description: >
  Batch send ERC-20 tokens or native ETH to up to 200 recipients in one
  transaction on Base. Use for payroll, airdrops, prize distribution,
  treasury disbursements, contributor payments, bounty payouts, token
  distributions, and any multi-recipient transfer. Trigger on: 'send
  tokens to multiple wallets', 'batch payment', 'pay N recipients',
  'airdrop tokens', 'distribute to wallets', 'payroll run', 'mass
  transfer', 'multi-send', 'batch USDC', 'send to list of addresses',
  'treasury payout', 'split payment', 'bulk transfer', 'prize payout'.
  Supports USDC, ETH, WETH, and all ERC-20 tokens on Base. Powered by
  Spraay x402 Gateway — no API keys needed, agent pays per call in USDC
  via x402 protocol.
license: MIT
metadata:
  author: plagtech
  version: "1.0.0"
  homepage: "https://spraay.app"
  repository: "https://github.com/plagtech/onchainos-spraay-batch"
  gateway: "https://gateway.spraay.app"
  contract: "0x1646452F98E36A3c9Cfc3eDD8868221E207B5eEC"
  network: "Base (EIP-155:8453)"
  payment: "x402 (USDC on Base)"
  bpa_spec: "https://github.com/plagtech/bpa"
agent:
  requires:
    env:
      - name: EVM_PRIVATE_KEY
        description: "Private key for a wallet with USDC on Base (for x402 payments) and tokens to send"
    packages:
      - name: "x402-client"
        install: "npm install x402-client"
        description: "x402 payment client for automatic micropayments"
---
```

### Markdown Body

The body should include these sections in order:

1. **Overview** — One paragraph: what the skill does, who it's for, how payment works.

2. **Prerequisites** — `EVM_PRIVATE_KEY` env var pointing to a wallet with USDC on Base (for x402 gateway fees) and whatever tokens the agent wants to send. Mention `x402-client` npm package.

3. **Commands** — Document each operation the agent can perform:

   **Estimate a batch payment:**
   ```
   POST https://gateway.spraay.app/api/v1/batch/estimate
   Content-Type: application/json

   {
     "token": "USDC",
     "recipients": [
       { "address": "0x...", "amount": "10.00" },
       { "address": "0x...", "amount": "25.00" }
     ]
   }
   ```
   Response includes estimated gas, total amount, fee breakdown.

   **Execute a batch payment:**
   ```
   POST https://gateway.spraay.app/api/v1/batch/execute
   Content-Type: application/json

   {
     "token": "USDC",
     "recipients": [
       { "address": "0x...", "amount": "10.00" },
       { "address": "0x...", "amount": "25.00" }
     ]
   }
   ```
   Response includes transaction hash, block number, gas used. The agent's wallet signs the batch via the Spraay contract.

   **Check balances:**
   ```
   GET https://gateway.spraay.app/api/v1/balances?address=0x...
   ```

   **Get prices:**
   ```
   GET https://gateway.spraay.app/api/v1/prices
   ```

4. **Recommended workflow** — The agent should always:
   - Check balances first (verify sufficient funds)
   - Estimate before executing (show the user projected cost)
   - Execute only after user confirmation
   - Report the tx hash and link to Basescan after success

5. **Supported tokens** — USDC, ETH (native), WETH, and any ERC-20 on Base. For non-standard tokens, pass the contract address in the `token` field instead of the symbol.

6. **Limits** — Max 200 recipients per batch. Min amount: 0.000001. Amounts are in human-readable units (not wei).

7. **Error handling** — Common errors: insufficient balance, invalid address, token not found, gas estimation failure. Include the error shape and suggested agent recovery actions.

8. **References** — Point to `{baseDir}/references/` files for detailed endpoint docs using the `{baseDir}` runtime path convention from OKX's skill format.

---

## Description Field Guidelines

The `description` in the YAML frontmatter drives agent routing — it's how LLMs decide when to activate this skill. Per OKX conventions:

- Keep under 900 characters (Codex CLI enforces 1024 max)
- Enumerate natural-language trigger phrases that cover all use cases
- Include common synonyms: "batch payment", "multi-send", "airdrop", "payroll", "mass transfer"
- Mention supported tokens explicitly (USDC, ETH)
- Mention the chain (Base)
- Mention that no API keys are needed (differentiator)

---

## Reference Files

Each file in `references/` provides deep context an agent can load when it needs details beyond what's in SKILL.md:

### `batch-execute.md`
- Full request/response schema with all fields
- Token field accepts symbols (USDC, ETH, WETH) or contract addresses
- Recipients array format: `{ address: string, amount: string }`
- Response: `{ txHash, blockNumber, gasUsed, recipients, totalAmount, token, fee }`
- x402 payment header format
- Basescan link format: `https://basescan.org/tx/{txHash}`

### `batch-estimate.md`
- Same request format as execute
- Response: `{ estimatedGas, estimatedFeeUSD, totalAmount, token, recipientCount }`
- Use this before execute to show the user what they'll pay

### `balances.md`
- Query params: `address` (required), `tokens` (optional comma-separated)
- Response: array of `{ token, symbol, balance, decimals }`

### `prices.md`
- No params required, returns all supported token prices
- Response: `{ ETH: number, USDC: number, WETH: number, ... }`

### `x402-payment.md`
- How x402 works: agent wallet signs a payment proof, attaches to request header
- The `x402-client` npm package handles this automatically
- Facilitator: Coinbase (`https://api.cdp.coinbase.com/platform/v2/x402`)
- No API keys, no accounts, no registration — just a wallet with USDC

---

## Example Files

Each file in `examples/` is a complete worked example an agent can reference:

### `payroll.md`
Scenario: Pay 10 contributors their monthly amounts in USDC.
Shows: balance check → estimate → user confirmation → execute → report tx hash.

### `airdrop.md`
Scenario: Distribute 1 USDC each to 100 community members.
Shows: building recipient list from CSV/array → estimate → execute.

### `treasury.md`
Scenario: DAO treasury sends mixed amounts to grant recipients.
Shows: price lookup (convert USD to token amounts) → batch execute.

---

## README.md Content

The README should include:

1. **Header** — 💧 Spraay Batch Payments for OKX OnchainOS
2. **One-liner** — "Batch send tokens to 200 wallets in one transaction. No API keys — pay per call with USDC on Base."
3. **Install** — `npx skills add plagtech/onchainos-spraay-batch` (or manual clone + symlink)
4. **Quick start** — 3-line example of an agent estimating + executing a batch
5. **Pricing table** — All 4 endpoints with costs
6. **How x402 works** — Brief explainer with link to x402.org
7. **Links** — Gateway, contract on Basescan, BPA spec, Spraay app, GitHub, Twitter (@Spraay_app)

---

## Implementation Notes for Claude Code

- Study the skill format at `https://github.com/okx/onchainos-skills` (especially existing skills like `okx-wallet-portfolio` and `okx-dex-swap`) before writing any files
- Also study `https://github.com/okx/agent-skills` for the YAML frontmatter conventions and description patterns
- The skill is documentation-only — no CLI binary, no runtime code. The agent makes HTTP calls directly to the Spraay gateway using `x402-client` or raw fetch with x402 payment headers
- All endpoint URLs, contract addresses, and pricing must match the live gateway at `https://gateway.spraay.app`
- Use `{baseDir}` for reference file paths per OKX convention
- Test that the description field is under 900 characters
- Include a `.github/` directory with a basic issue template if desired
- Do NOT fabricate contract addresses or endpoint responses — use only real data from the live gateway
