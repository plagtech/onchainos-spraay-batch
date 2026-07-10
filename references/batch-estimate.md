# Reference: `POST /api/v1/batch/estimate`

Estimate the on-chain gas cost of a batch **before** executing it. Gas for a
Spraay batch is driven by the recipient count and the token type, so this
endpoint takes a count rather than the full recipient list — making it cheap to
call while planning.

- **Method:** `POST`
- **URL:** `https://gateway.spraay.app/api/v1/batch/estimate`
- **x402 price:** $0.001 (USDC on Base)
- **Content-Type:** `application/json`

## Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `recipientCount` | number | yes | Number of recipients to estimate for (1–200). |
| `token` | string | no | Token symbol or ERC-20 contract address. Defaults to `USDC`. ERC-20 transfers cost more gas than native `ETH`. |

### Example request

```json
{
  "token": "USDC",
  "recipientCount": 10
}
```

## Response

Returns the estimated gas in wei and its USD equivalent. Shape (illustrative —
matches the documented contract):

```json
{
  "token": "USDC",
  "recipientCount": 10,
  "estimatedGas": "312450",
  "estimatedGasUSD": "0.041",
  "chainId": 8453
}
```

| Field | Type | Description |
|---|---|---|
| `token` | string | Token the estimate was computed for. |
| `recipientCount` | number | Recipient count used for the estimate. |
| `estimatedGas` | string | Estimated gas units / cost in wei. |
| `estimatedGasUSD` | string | Estimated gas cost converted to USD. |
| `chainId` | number | `8453` (Base). |

The values above are illustrative placeholders, not a live capture — actual gas
depends on current Base network conditions.

## How to use it

Call `/batch/estimate` in the planning phase and show the user the projected gas
cost. This is the cheapest gateway call ($0.001), so estimating before executing
is always worthwhile. The gas figure here is the **on-chain** cost of the batch
and is separate from the $0.02 x402 fee for `/batch/execute`.

## x402 payment

Paid endpoint. On the first unpaid request the gateway responds `402 Payment
Required`; `x402-client` settles $0.001 in USDC on Base and retries. See
`x402-payment.md`.

## Errors

| Code | Meaning |
|---|---|
| `INVALID_RECIPIENT_COUNT` | `recipientCount` missing or outside 1–200. |
| `TOKEN_NOT_FOUND` | Token symbol/contract not resolvable on Base. |
| `GAS_ESTIMATION_FAILED` | The batch could not be simulated. |
