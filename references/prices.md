# Reference: `GET /api/v1/prices`

Live on-chain USD prices for tokens on Base, sourced from Uniswap V3 pools. Use
this to convert USD-denominated amounts into token units before building a batch.

- **Method:** `GET`
- **URL:** `https://gateway.spraay.app/api/v1/prices`
- **x402 price:** $0.005 (USDC on Base)

## Query parameters

| Param | Required | Description |
|---|---|---|
| `token` | no | A single token symbol (e.g. `WETH`, `cbBTC`). Omit to return all supported token prices. |

### Example request

```
GET https://gateway.spraay.app/api/v1/prices
```

Single token:

```
GET https://gateway.spraay.app/api/v1/prices?token=WETH
```

## Response

USD prices, each with a timestamp. Shape (illustrative — matches the documented
schema, values are placeholders not a live capture):

```json
{
  "chainId": 8453,
  "source": "uniswap-v3",
  "prices": {
    "ETH":  { "usd": 2476.30, "timestamp": 1752192000 },
    "WETH": { "usd": 2475.90, "timestamp": 1752192000 },
    "USDC": { "usd": 1.0000,  "timestamp": 1752192000 }
  }
}
```

| Field | Type | Description |
|---|---|---|
| `prices` | object | Map of token symbol → `{ usd, timestamp }`. |
| `usd` | number | Current USD price from Uniswap V3 pools on Base. |
| `timestamp` | number | Unix seconds when the price was read. |
| `source` | string | Price source (`uniswap-v3`). |

## Converting USD to token amounts

To send "$50 of WETH" to each recipient:

1. `GET /api/v1/prices?token=WETH` → read `prices.WETH.usd`.
2. `amount = 50 / prices.WETH.usd` → format to the token's precision.
3. Use that string in the `amounts` array of `/batch/execute`.

## Free alternative

For non-critical planning you can use the free, cached spot-price feed
(USDC/ETH/SOL, cached ~60s) exposed by the Spraay MCP server
(`spraay_free_prices`) instead of the paid on-chain endpoint.

## x402 payment

Paid endpoint ($0.005, USDC on Base). Handled by `x402-client` on the `402`
challenge. See `x402-payment.md`.
