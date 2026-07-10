# Example: Airdrop — 1 USDC each to 100 wallets

**Scenario:** Distribute exactly **1 USDC** to **100** community members from a
CSV or address array, in a single Base transaction.

Because 100 ≤ 200, this fits in one batch.

## 1. Build the recipient list

Start from a CSV (`wallets.csv`) or an in-memory array of addresses. For a fixed
per-wallet amount, generate a matching `amounts` array of the same length:

```js
// addresses: string[] of 100 Base wallet addresses
const recipients = addresses;              // length 100
const amounts = addresses.map(() => "1");  // "1" USDC each, length 100

if (recipients.length !== amounts.length) throw new Error("length mismatch");
if (recipients.length > 200) throw new Error("split into batches of 200");
```

Total to distribute: **100 USDC**.

## 2. Check balances

```
GET https://gateway.spraay.app/api/v1/balances?address=0xYourAgentWalletAddress00000000000000000
```

Confirm ≥ **100 USDC** to send, **ETH** for gas, and **USDC** for x402 fees.

## 3. Estimate

```
POST https://gateway.spraay.app/api/v1/batch/estimate
Content-Type: application/json

{ "token": "USDC", "recipientCount": 100 }
```

Show the user `estimatedGasUSD` — a 100-recipient batch costs more gas than a
small one, but is still one transaction rather than 100.

## 4. Execute

```
POST https://gateway.spraay.app/api/v1/batch/execute
Content-Type: application/json

{
  "token": "USDC",
  "recipients": ["0x...", "0x...", "... 100 addresses ..."],
  "amounts":    ["1", "1", "... 100 ones ..."],
  "sender": "0xYourAgentWalletAddress00000000000000000"
}
```

Sign the returned `transaction` and broadcast to Base.

## 5. Report

> ✅ Airdropped **1 USDC** to **100 wallets** (100 USDC total) in one tx.
> Tx: https://basescan.org/tx/0x&lt;txHash&gt;

## Scaling past 200

The batch limit is **200 recipients**. For a larger airdrop, chunk the list and
run one `/batch/execute` per chunk:

```js
function chunk(arr, size) {
  const out = [];
  for (let i = 0; i < arr.length; i += size) out.push(arr.slice(i, i + size));
  return out;
}

for (const group of chunk(recipients, 200)) {
  // estimate → execute → record txHash for each group
}
```

Each chunk is its own transaction and its own $0.02 gateway fee. Report every tx
hash back to the user.
