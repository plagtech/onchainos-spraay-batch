# Reference: x402 Payments

The Spraay gateway is monetized with **x402** — an open HTTP micropayment
protocol. There are no API keys, accounts, or registration. Each paid endpoint
returns `402 Payment Required` with machine-readable payment requirements; the
client pays in USDC on Base and retries. Your wallet is your account.

## How it works

1. The agent sends a normal request to a gateway endpoint (e.g.
   `POST /api/v1/batch/execute`).
2. If no valid payment is attached, the gateway responds **HTTP 402** with a JSON
   body describing the payment requirements (amount, asset, network, pay-to
   address, scheme).
3. The client builds a signed **payment proof** authorizing the required USDC
   transfer on Base, and re-sends the request with an `X-PAYMENT` header.
4. The gateway verifies the payment with a **facilitator**, settles it, and
   returns the real response plus an `X-PAYMENT-RESPONSE` header confirming
   settlement.

The `x402-client` npm package performs steps 2–4 automatically — you just make
the request as if it were free.

## Payment parameters

| Property | Value |
|---|---|
| Scheme | `exact` (`@x402/evm`) |
| Network | Base (EIP-155:8453) |
| Asset | USDC on Base (`0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`) |
| Facilitator | Coinbase — `https://api.cdp.coinbase.com/platform/v2/x402` |

## Per-call prices

| Endpoint | Price (USDC) |
|---|---|
| `POST /api/v1/batch/estimate` | $0.001 |
| `POST /api/v1/batch/execute` | $0.02 |
| `GET /api/v1/balances` | $0.005 |
| `GET /api/v1/prices` | $0.005 |

These x402 fees pay for the gateway call only. They are **separate** from:
- the on-chain **gas** for the batch transaction (paid in ETH by `sender`), and
- the **tokens** being distributed to recipients.

## Using `x402-client`

```js
import { createX402Client } from "x402-client";

const client = createX402Client({
  privateKey: process.env.EVM_PRIVATE_KEY, // wallet with USDC on Base
  network: "base",
});

// Fees are paid automatically on the 402 challenge.
const res = await client.post(
  "https://gateway.spraay.app/api/v1/batch/execute",
  {
    token: "USDC",
    recipients: ["0x...", "0x..."],
    amounts: ["10.00", "25.00"],
    sender: "0xYourAgentWalletAddress00000000000000000",
  }
);
```

## Raw HTTP (manual x402)

If you are not using `x402-client`, implement the handshake yourself:

1. Send the request; on `402`, read the payment requirements from the response
   body.
2. Construct and sign the payment payload per the `exact` scheme for Base.
3. Base64-encode it into the `X-PAYMENT` request header and re-send.
4. On success, read `X-PAYMENT-RESPONSE` for the settlement receipt.

## What you need

- An **EVM wallet on Base** (`EVM_PRIVATE_KEY`).
- Enough **USDC on Base** to cover the per-call fees above.
- Enough **ETH on Base** for the batch transaction's gas (only for
  `/batch/execute`).

More about the protocol: <https://x402.org>.
