# PR Title:
feat: add spraay-batch — multi-recipient token transfers on Base

# PR Body:

## Summary

Adds `spraay-batch`, a third-party skill that gives OnchainOS agents the ability to batch-send ERC-20 tokens and native ETH to up to 200 recipients in a single transaction on Base.

## What it does

Any agent can estimate, then execute a multi-recipient token transfer through the Spraay x402 Gateway — no API keys, no accounts. The agent's wallet pays per call in USDC via x402, same protocol OnchainOS already supports natively.

**Endpoints exposed:**

| Endpoint | Price | Description |
|---|---|---|
| `POST /api/v1/batch/execute` | $0.02 | Batch send to up to 200 recipients |
| `POST /api/v1/batch/estimate` | $0.001 | Estimate gas + fees before executing |
| `GET /api/v1/balances` | $0.005 | Check wallet balances |
| `GET /api/v1/prices` | $0.005 | Token prices on Base |

## Use cases

- **Payroll** — pay contributors monthly in USDC
- **Airdrops** — distribute tokens to community members
- **Treasury** — DAO disbursements to grant recipients
- **Bounties / prizes** — pay multiple winners in one tx

## Why this belongs in OnchainOS

- **No overlap** — no existing skill handles multi-recipient transfers. The closest is `okx-dex-swap` (1:1 swaps) and `okx-wallet-portfolio` (read-only). This fills a gap.
- **x402 native** — payment protocol is already in the OnchainOS dispatcher, so integration is zero-friction.
- **Live on mainnet** — Spraay batch contract (`0x1646452F98E36A3c9Cfc3eDD8868221E207B5eEC`) has real transaction history on Base. Not a testnet prototype.
- **BPA 1.0 compliant** — implements the open Batch Payments for Agents spec.

## Technical details

- **Network:** Base (EIP-155:8453)
- **Gateway:** `gateway.spraay.app` (190 total endpoints, x402 + MPP)
- **Contract:** `0x1646452F98E36A3c9Cfc3eDD8868221E207B5eEC`
- **Tokens:** USDC, ETH, WETH, and any ERC-20 on Base
- **Skill format:** Standard YAML frontmatter + Markdown body, `{baseDir}` references, under 900-char description

## Files added

```
skills/spraay-batch/SKILL.md                         — Main skill file
skills/spraay-batch/references/batch-execute.md      — Execute endpoint docs
skills/spraay-batch/references/batch-estimate.md     — Estimate endpoint docs
skills/spraay-batch/references/balances.md           — Balances endpoint docs
skills/spraay-batch/references/prices.md             — Prices endpoint docs
skills/spraay-batch/references/x402-payment.md       — x402 protocol explainer
skills/spraay-batch/references/example-payroll.md    — Worked example: contributor payroll
skills/spraay-batch/references/example-airdrop.md    — Worked example: token airdrop
skills/spraay-batch/references/example-treasury.md   — Worked example: DAO treasury
```

All files live under `skills/spraay-batch/` to match the repo's skill convention (`skills/<name>/SKILL.md` + `references/`). Worked examples are folded into `references/` as `example-*.md`, since the repo has no separate `examples/` directory. No existing files are modified.

## Links

- Standalone repo: [plagtech/onchainos-spraay-batch](https://github.com/plagtech/onchainos-spraay-batch)
- Gateway: [gateway.spraay.app](https://gateway.spraay.app)
- Contract: [Basescan](https://basescan.org/address/0x1646452F98E36A3c9Cfc3eDD8868221E207B5eEC)
- BPA Spec: [plagtech/bpa](https://github.com/plagtech/bpa)
- x402 Protocol: [x402.org](https://x402.org)
- Twitter: [@Spraay_app](https://twitter.com/Spraay_app)
