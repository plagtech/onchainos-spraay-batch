# Example: Treasury — DAO grant disbursement with USD→token conversion

**Scenario:** A DAO treasury approved grants denominated in **USD**, but wants to
pay them in **WETH** on Base. The agent looks up the live WETH price, converts
each USD grant into a WETH amount, then sends them all in one batch.

This shows the **prices → convert → estimate → execute** flow.

## 0. Approved grants (in USD)

```json
[
  { "address": "0xGrantee0000000000000000000000000000001", "usd": "5000" },
  { "address": "0xGrantee0000000000000000000000000000002", "usd": "2500" },
  { "address": "0xGrantee0000000000000000000000000000003", "usd": "10000" }
]
```

## 1. Look up the live WETH price

```
GET https://gateway.spraay.app/api/v1/prices?token=WETH
```

Read `prices.WETH.usd` from the response (on-chain price from Uniswap V3 pools on
Base, with a timestamp). Suppose it returns `2475.90`.

## 2. Convert USD → WETH amounts

```js
const wethUsd = 2475.90; // from /prices
const grants = [
  { address: "0xGrantee0000000000000000000000000000001", usd: 5000 },
  { address: "0xGrantee0000000000000000000000000000002", usd: 2500 },
  { address: "0xGrantee0000000000000000000000000000003", usd: 10000 },
];

const recipients = grants.map(g => g.address);
const amounts = grants.map(g => (g.usd / wethUsd).toFixed(6)); // WETH, 6 dp precision
// => ["2.019468", "1.009734", "4.038937"]
```

Always show the user the derived token amounts and the price/timestamp you used,
so the conversion is auditable.

## 3. Check balances

```
GET https://gateway.spraay.app/api/v1/balances?address=0xTreasuryWalletAddress0000000000000000
```

Confirm the treasury holds enough **WETH** (~7.07 WETH here), plus **ETH** for
gas and **USDC** for x402 fees.

## 4. Estimate

```
POST https://gateway.spraay.app/api/v1/batch/estimate
Content-Type: application/json

{ "token": "WETH", "recipientCount": 3 }
```

## 5. Confirm and execute

After the user approves the converted amounts:

```
POST https://gateway.spraay.app/api/v1/batch/execute
Content-Type: application/json

{
  "token": "WETH",
  "recipients": [
    "0xGrantee0000000000000000000000000000001",
    "0xGrantee0000000000000000000000000000002",
    "0xGrantee0000000000000000000000000000003"
  ],
  "amounts": ["2.019468", "1.009734", "4.038937"],
  "sender": "0xTreasuryWalletAddress0000000000000000"
}
```

Sign the returned `transaction` with `EVM_PRIVATE_KEY` and broadcast to Base.

## 6. Report

> ✅ Disbursed **3 grants** (~$17,500) as **7.068139 WETH** in one tx,
> at a WETH price of $2,475.90.
> Tx: https://basescan.org/tx/0x&lt;txHash&gt;

## Notes

- Re-fetch `/prices` immediately before executing if the price may have moved;
  include the `timestamp` in your report for auditability.
- To pay grants in a token without a built-in symbol, pass its **contract
  address** in the `token` field instead of a symbol.
- For mixed tokens (some grants in USDC, some in WETH), run **one batch per
  token** — a batch distributes a single token.
