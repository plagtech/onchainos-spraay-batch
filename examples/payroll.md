# Example: Payroll — pay 10 contributors in USDC

**Scenario:** An agent runs monthly payroll, paying 10 contributors their agreed
USDC amounts on Base in one transaction.

This walks through the full recommended flow: **balances → estimate → confirm →
execute → report**.

## 0. Inputs

```json
[
  { "address": "0xAbc0000000000000000000000000000000000001", "usdc": "1200.00" },
  { "address": "0xAbc0000000000000000000000000000000000002", "usdc": "950.00"  },
  { "address": "0xAbc0000000000000000000000000000000000003", "usdc": "1800.00" },
  { "address": "0xAbc0000000000000000000000000000000000004", "usdc": "600.00"  },
  { "address": "0xAbc0000000000000000000000000000000000005", "usdc": "2200.00" },
  { "address": "0xAbc0000000000000000000000000000000000006", "usdc": "750.00"  },
  { "address": "0xAbc0000000000000000000000000000000000007", "usdc": "1100.00" },
  { "address": "0xAbc0000000000000000000000000000000000008", "usdc": "1350.00" },
  { "address": "0xAbc0000000000000000000000000000000000009", "usdc": "500.00"  },
  { "address": "0xAbc000000000000000000000000000000000000A", "usdc": "1600.00" }
]
```

Total to distribute: **12,050.00 USDC**. Sender wallet:
`0xYourAgentWalletAddress00000000000000000`.

## 1. Check balances

```
GET https://gateway.spraay.app/api/v1/balances?address=0xYourAgentWalletAddress00000000000000000
```

Confirm the sender holds **≥ 12,050 USDC**, some **ETH** for gas, and **USDC**
to cover x402 fees (~$0.026 total across this run). If USDC is short, stop and
tell the user.

## 2. Estimate gas

```
POST https://gateway.spraay.app/api/v1/batch/estimate
Content-Type: application/json

{ "token": "USDC", "recipientCount": 10 }
```

Read `estimatedGasUSD` from the response and keep it for the confirmation step.

## 3. Confirm with the user

> Ready to send **12,050.00 USDC** to **10 recipients** in one transaction on
> Base. Estimated network gas ≈ `$<estimatedGasUSD>`, plus a $0.02 gateway fee.
> Proceed?

Only continue on explicit approval.

## 4. Execute

Recipients and amounts are parallel arrays in the same order:

```
POST https://gateway.spraay.app/api/v1/batch/execute
Content-Type: application/json

{
  "token": "USDC",
  "recipients": [
    "0xAbc0000000000000000000000000000000000001",
    "0xAbc0000000000000000000000000000000000002",
    "0xAbc0000000000000000000000000000000000003",
    "0xAbc0000000000000000000000000000000000004",
    "0xAbc0000000000000000000000000000000000005",
    "0xAbc0000000000000000000000000000000000006",
    "0xAbc0000000000000000000000000000000000007",
    "0xAbc0000000000000000000000000000000000008",
    "0xAbc0000000000000000000000000000000000009",
    "0xAbc000000000000000000000000000000000000A"
  ],
  "amounts": ["1200.00","950.00","1800.00","600.00","2200.00","750.00","1100.00","1350.00","500.00","1600.00"],
  "sender": "0xYourAgentWalletAddress00000000000000000"
}
```

Sign the returned `transaction` with `EVM_PRIVATE_KEY` and broadcast it to Base.

## 5. Report

> ✅ Paid 10 contributors **12,050.00 USDC**.
> Tx: https://basescan.org/tx/0x&lt;txHash&gt;

## Notes

- The `x402-client` package pays the $0.02 execute fee automatically.
- If any address is malformed you'll get `INVALID_ADDRESS` — validate each is
  `0x` + 40 hex chars before executing.
- Keep the recipient/amount ordering identical between the two arrays.
