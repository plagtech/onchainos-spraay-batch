# Reference: `GET /api/v1/balances`

Fetch the native ETH balance plus ERC-20 token balances for any wallet on Base,
with USD values where a price is available. Use this before estimating/executing
to confirm the sender can cover the distribution, gas, and x402 fees.

- **Method:** `GET`
- **URL:** `https://gateway.spraay.app/api/v1/balances`
- **x402 price:** $0.005 (USDC on Base)

## Query parameters

| Param | Required | Description |
|---|---|---|
| `address` | yes | Wallet address to check (`0x` + 40 hex chars). |
| `showAll` | no | `true` to include zero balances; defaults to omitting them. |
| `tokens` | no | Comma-separated list of custom ERC-20 **contract addresses** to include beyond the default set. |

### Example request

```
GET https://gateway.spraay.app/api/v1/balances?address=0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
```

With extras:

```
GET https://gateway.spraay.app/api/v1/balances?address=0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045&showAll=true&tokens=0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913
```

## Response

An array of per-token balance objects. Shape (illustrative — matches the
documented schema, not a live capture):

```json
{
  "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
  "chainId": 8453,
  "balances": [
    { "token": "ETH",  "symbol": "ETH",  "balance": "0.4213", "decimals": 18, "usdValue": "1043.21" },
    { "token": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913", "symbol": "USDC", "balance": "250.00", "decimals": 6, "usdValue": "250.00" }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `token` | string | `ETH` for native, otherwise the ERC-20 contract address. |
| `symbol` | string | Human-readable token symbol. |
| `balance` | string | Balance in human-readable units (already decimal-adjusted). |
| `decimals` | number | Token decimals. |
| `usdValue` | string | USD value where a price is available. |

## How to use it

- Verify the sender holds enough of the token being distributed.
- Verify there is enough native ETH for the batch transaction's gas.
- Verify there is enough USDC to cover x402 gateway fees.

## x402 payment

Paid endpoint ($0.005, USDC on Base). Handled automatically by `x402-client` on
the `402` challenge. See `x402-payment.md`.

## Errors

| Code | Meaning |
|---|---|
| `INVALID_ADDRESS` | `address` is missing or malformed. |
| `PAYMENT_REQUIRED` | x402 payment missing/invalid (HTTP 402). |
