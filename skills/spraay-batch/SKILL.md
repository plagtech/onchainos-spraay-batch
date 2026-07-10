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

# 💧 Spraay Batch Payments

Batch-send ERC-20 tokens or native ETH to **up to 200 recipients in a single
Base transaction**, using Spraay's live batch-payment gateway. This skill is for
any MCP-connected agent (Claude Code, Cursor, Codex, OpenClaw, Hermes) that
needs to run payroll, airdrops, prize payouts, or treasury disbursements without
sending one transfer at a time.

Every gateway call is paid **per-request over x402** (USDC on Base). There are no
API keys, no accounts, and no OKX credentials — the agent's own EVM wallet settles
each micropayment automatically via the `x402-client` package. The token transfers
themselves are signed by the agent's wallet; the gateway only builds the batch and
returns unsigned transaction data.

## Prerequisites

- **`EVM_PRIVATE_KEY`** — environment variable holding the private key of a Base
  wallet that has:
  1. A small USDC balance to pay x402 gateway fees (see pricing below), and
  2. The tokens (USDC, ETH, WETH, or any ERC-20) it intends to distribute, plus
     ETH for the batch transaction's own gas.
- **`x402-client`** — `npm install x402-client`. Handles the x402 payment
  handshake (signs a payment proof, attaches the `X-PAYMENT` header, retries on
  `402`). Raw `fetch` also works if you implement x402 manually — see
  `{baseDir}/references/x402-payment.md`.

The gateway runs on **Base mainnet (Chain ID 8453)**. The batch contract is
`0x1646452F98E36A3c9Cfc3eDD8868221E207B5eEC`.

## Pricing

| Operation | Endpoint | Method | x402 price |
|---|---|---|---|
| Estimate a batch | `/api/v1/batch/estimate` | POST | **$0.001** |
| Execute a batch | `/api/v1/batch/execute` | POST | **$0.02** |
| Check balances | `/api/v1/balances` | GET | **$0.005** |
| Get prices | `/api/v1/prices` | GET | **$0.005** |

Prices are settled in USDC on Base via x402. They are the gateway's per-call
fees and are separate from the on-chain gas for the batch transaction itself.

## Commands

### 1. Estimate a batch payment — `POST /api/v1/batch/estimate`

Estimate gas before committing. Estimation depends on the number of recipients
and the token type, so it takes a count rather than the full recipient list.

```
POST https://gateway.spraay.app/api/v1/batch/estimate
Content-Type: application/json

{
  "token": "USDC",
  "recipientCount": 10
}
```

Response includes the estimated gas (in wei) and its USD equivalent. See
`{baseDir}/references/batch-estimate.md` for the full schema.

### 2. Execute a batch payment — `POST /api/v1/batch/execute`

Recipients and amounts are **two parallel arrays** — `amounts[i]` is sent to
`recipients[i]`. Both arrays must be the same length. Amounts are strings in
human-readable token units (e.g. `"100"` = 100 USDC), never wei. `sender` is the
wallet that will sign the resulting transaction.

```
POST https://gateway.spraay.app/api/v1/batch/execute
Content-Type: application/json

{
  "token": "USDC",
  "recipients": [
    "0xAbC0000000000000000000000000000000000001",
    "0xAbC0000000000000000000000000000000000002"
  ],
  "amounts": ["10.00", "25.00"],
  "sender": "0xYourAgentWalletAddress00000000000000000"
}
```

The gateway returns **unsigned transaction data** targeting the Spraay batch
contract. The agent signs it with `EVM_PRIVATE_KEY` and broadcasts it to Base.
After the transaction confirms, report the tx hash and a Basescan link. See
`{baseDir}/references/batch-execute.md`.

### 3. Check balances — `GET /api/v1/balances`

```
GET https://gateway.spraay.app/api/v1/balances?address=0xYourAgentWalletAddress00000000000000000
```

Optional query params: `showAll=true` (include zero balances), and
`tokens=0x...,0x...` (comma-separated ERC-20 contract addresses to include).
Returns native ETH plus ERC-20 balances with USD values where available. See
`{baseDir}/references/balances.md`.

### 4. Get prices — `GET /api/v1/prices`

```
GET https://gateway.spraay.app/api/v1/prices
```

Returns live on-chain USD prices sourced from Uniswap V3 pools on Base, each with
a timestamp. Pass `?token=WETH` (or any symbol) to fetch a single token; omit for
all supported tokens. See `{baseDir}/references/prices.md`.

## Recommended workflow

Always run operations in this order:

1. **Check balances** (`/balances`) — confirm the sender holds enough of the
   token to distribute, plus ETH for gas and USDC for x402 fees.
2. **Estimate** (`/batch/estimate`) — show the user the projected gas cost before
   spending anything on execution.
3. **Confirm with the user** — present the recipient count, total amount, token,
   and estimated cost, and wait for explicit approval.
4. **Execute** (`/batch/execute`) — receive the unsigned tx, sign with
   `EVM_PRIVATE_KEY`, broadcast to Base.
5. **Report** — return the transaction hash and a Basescan link
   (`https://basescan.org/tx/{txHash}`), and summarize what was sent.

Optionally call `/prices` first when the user specifies amounts in USD and you
need to convert them into token units.

## Supported tokens

`USDC`, `ETH` (native), `WETH`, and **any ERC-20 on Base**. Pass a recognized
symbol (`USDC`, `ETH`, `WETH`, `DAI`, …) in the `token` field. For any token
without a built-in symbol, pass its **contract address** in the `token` field
instead of a symbol.

## Limits

- **Max 200 recipients** per batch.
- **Minimum amount:** `0.000001` per recipient.
- Amounts are **human-readable units** (e.g. `"10.5"`), never wei.
- `recipients` and `amounts` arrays must be equal length.

## Error handling

Errors are returned as JSON with an `error` field and a machine-readable `code`:

```json
{ "error": "Insufficient balance for token USDC", "code": "INSUFFICIENT_BALANCE" }
```

Common cases and recovery:

| Code | Meaning | Agent recovery |
|---|---|---|
| `INSUFFICIENT_BALANCE` | Sender lacks enough token/ETH | Re-run `/balances`, reduce amounts, or top up |
| `INVALID_ADDRESS` | A recipient address is malformed | Validate each address is `0x` + 40 hex chars |
| `TOKEN_NOT_FOUND` | Symbol/contract not resolvable on Base | Pass the ERC-20 contract address instead |
| `GAS_ESTIMATION_FAILED` | Chain could not simulate the batch | Retry; check recipient count ≤ 200 |
| `PAYMENT_REQUIRED` (HTTP 402) | x402 payment missing/invalid | Ensure USDC balance; let `x402-client` retry |

On HTTP `402`, the response body carries the x402 payment requirements — the
`x402-client` package pays and retries automatically. See
`{baseDir}/references/x402-payment.md`.

## References

- `{baseDir}/references/batch-execute.md` — full `/batch/execute` schema
- `{baseDir}/references/batch-estimate.md` — full `/batch/estimate` schema
- `{baseDir}/references/balances.md` — full `/balances` schema
- `{baseDir}/references/prices.md` — full `/prices` schema
- `{baseDir}/references/x402-payment.md` — how x402 micropayments work
- `{baseDir}/examples/payroll.md` — pay 10 contributors
- `{baseDir}/examples/airdrop.md` — airdrop to 100 wallets
- `{baseDir}/examples/treasury.md` — DAO treasury disbursement
