# 💧 Spraay Batch Payments for OKX OnchainOS

**Batch send tokens to 200 wallets in one transaction. No API keys — pay per call
with USDC on Base.**

An [OKX OnchainOS](https://github.com/okx/onchainos-skills) skill that gives any
MCP-connected AI agent (Claude Code, Cursor, Codex, OpenClaw, Hermes) the ability
to batch-send ERC-20 tokens or native ETH to up to **200 recipients in a single
Base transaction**, powered by the **Spraay x402 Gateway**. Payment is
per-request over [x402](https://x402.org) (USDC on Base) — no accounts, no API
keys, no OKX credentials. The agent's own wallet settles each micropayment.

## Install

```bash
npx skills add plagtech/onchainos-spraay-batch
```

Or manually:

```bash
git clone https://github.com/plagtech/onchainos-spraay-batch
# symlink or copy skills/spraay-batch into your agent's skills directory
ln -s "$(pwd)/onchainos-spraay-batch/skills/spraay-batch" ~/.claude/skills/spraay-batch
```

Set the one required env var (a Base wallet holding USDC for fees + the tokens to
send) and install the payment client:

```bash
export EVM_PRIVATE_KEY=0x...
npm install x402-client
```

## Quick start

```js
import { createX402Client } from "x402-client";
const client = createX402Client({ privateKey: process.env.EVM_PRIVATE_KEY, network: "base" });

// 1. Estimate gas for a 2-recipient USDC batch ($0.001)
await client.post("https://gateway.spraay.app/api/v1/batch/estimate",
  { token: "USDC", recipientCount: 2 });

// 2. Execute — returns an unsigned tx to sign & broadcast ($0.02)
await client.post("https://gateway.spraay.app/api/v1/batch/execute", {
  token: "USDC",
  recipients: ["0xAbc...1", "0xAbc...2"],
  amounts: ["10.00", "25.00"],
  sender: process.env.SENDER_ADDRESS,
});
```

`recipients` and `amounts` are **parallel arrays** — `amounts[i]` goes to
`recipients[i]`. Amounts are human-readable units (not wei). The gateway returns
**unsigned** transaction data; your wallet signs and broadcasts it to Base.

## Pricing

All fees are settled in USDC on Base via x402. They cover the gateway call only —
on-chain gas and the distributed tokens are separate.

| Endpoint | Method | Price | Description |
|---|---|---|---|
| `/api/v1/batch/estimate` | POST | **$0.001** | Estimate gas before executing |
| `/api/v1/batch/execute` | POST | **$0.02** | Batch send to up to 200 recipients |
| `/api/v1/balances` | GET | **$0.005** | Check ERC-20 + native balances |
| `/api/v1/prices` | GET | **$0.005** | Live token prices (Uniswap V3 on Base) |

## How x402 works

x402 is an open HTTP micropayment protocol. When you call a paid endpoint without
payment, the gateway replies `402 Payment Required` with the payment terms. The
`x402-client` package signs a USDC payment proof on Base, attaches it as an
`X-PAYMENT` header, and retries — automatically. Your wallet is your account:
no keys, no signups. Settlement is verified via the Coinbase facilitator. Learn
more at [x402.org](https://x402.org). Details:
[`references/x402-payment.md`](references/x402-payment.md).

## Documentation

- **Skill:** [`skills/spraay-batch/SKILL.md`](skills/spraay-batch/SKILL.md)
- **Endpoint refs:** [`references/`](references/) — execute, estimate, balances,
  prices, x402
- **Worked examples:** [`examples/`](examples/) —
  [payroll](examples/payroll.md), [airdrop](examples/airdrop.md),
  [treasury](examples/treasury.md)

## Links

- **Gateway:** https://gateway.spraay.app
- **Batch contract (Basescan):**
  https://basescan.org/address/0x1646452F98E36A3c9Cfc3eDD8868221E207B5eEC
- **Network:** Base mainnet (Chain ID 8453)
- **BPA spec:** https://github.com/plagtech/bpa
- **Spraay app:** https://spraay.app
- **GitHub:** https://github.com/plagtech/onchainos-spraay-batch
- **Twitter/X:** [@Spraay_app](https://x.com/Spraay_app)

## License

[MIT](LICENSE) © 2026 plagtech
