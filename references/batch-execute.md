# Reference: `POST /api/v1/batch/execute`

Build a single Base transaction that distributes one token to up to 200
recipients through the Spraay batch contract
(`0x1646452F98E36A3c9Cfc3eDD8868221E207B5eEC`). The gateway returns **unsigned**
transaction data — the sender signs and broadcasts it.

- **Method:** `POST`
- **URL:** `https://gateway.spraay.app/api/v1/batch/execute`
- **x402 price:** $0.02 (USDC on Base)
- **Content-Type:** `application/json`

## Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `token` | string | yes | Symbol (`USDC`, `ETH`, `WETH`, `DAI`, …) **or** an ERC-20 contract address on Base. `ETH` = native currency. |
| `recipients` | string[] | yes | 1–200 wallet addresses, each `0x` + 40 hex chars. |
| `amounts` | string[] | yes | Amounts in human-readable token units (e.g. `"100"` = 100 USDC), **not wei**. Must be the same length as `recipients`; `amounts[i]` is sent to `recipients[i]`. |
| `sender` | string | yes | The wallet address that will sign and pay for the batch transaction. |

`recipients` and `amounts` are **parallel arrays**, not an array of objects.

### Example request

```json
{
  "token": "USDC",
  "recipients": [
    "0xAbC0000000000000000000000000000000000001",
    "0xAbC0000000000000000000000000000000000002",
    "0xAbC0000000000000000000000000000000000003"
  ],
  "amounts": ["10.00", "25.50", "5.00"],
  "sender": "0xYourAgentWalletAddress00000000000000000"
}
```

## Response

The gateway returns the unsigned transaction plus a summary of the batch. Shape
(illustrative — matches the documented contract; sign the `transaction` object
and broadcast it to Base):

```json
{
  "transaction": {
    "to": "0x1646452F98E36A3c9Cfc3eDD8868221E207B5eEC",
    "from": "0xYourAgentWalletAddress00000000000000000",
    "data": "0x...",
    "value": "0",
    "chainId": 8453
  },
  "token": "USDC",
  "recipientCount": 3,
  "totalAmount": "40.50",
  "fee": "0.02"
}
```

| Field | Type | Description |
|---|---|---|
| `transaction` | object | Unsigned tx (`to`, `from`, `data`, `value`, `chainId`). Sign with `EVM_PRIVATE_KEY` and broadcast. |
| `token` | string | The token that will be distributed. |
| `recipientCount` | number | Number of recipients in the batch. |
| `totalAmount` | string | Sum of all `amounts`, in token units. |
| `fee` | string | The x402 gateway fee charged for this call (USD). |

For native `ETH`, `transaction.value` carries the total wei being distributed;
for ERC-20 tokens the sender must have already approved the batch contract, or
the returned `data` includes the transfer path defined by the contract.

## After broadcasting

Once the signed transaction confirms, you have a transaction hash. Report it and
link to Basescan:

```
https://basescan.org/tx/{txHash}
```

## x402 payment

This endpoint is paid. On the first unpaid request the gateway responds `402
Payment Required` with x402 requirements in the body. The `x402-client` package
pays $0.02 in USDC on Base and retries automatically. See `x402-payment.md`.

## Errors

```json
{ "error": "Insufficient balance for token USDC", "code": "INSUFFICIENT_BALANCE" }
```

| Code | Meaning |
|---|---|
| `INSUFFICIENT_BALANCE` | Sender cannot cover the total amount or gas. |
| `INVALID_ADDRESS` | A recipient (or `sender`) address is malformed. |
| `LENGTH_MISMATCH` | `recipients` and `amounts` differ in length. |
| `TOO_MANY_RECIPIENTS` | More than 200 recipients supplied. |
| `TOKEN_NOT_FOUND` | Token symbol/contract not resolvable on Base. |
| `GAS_ESTIMATION_FAILED` | The batch could not be simulated. |
