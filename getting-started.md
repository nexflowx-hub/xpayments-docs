# Getting Started

XPayments separates **merchant-facing credentials** from **provider credentials**. A merchant integrates with a Store-scoped XPayments API key while provider credentials remain inside the XPayments GatewayVault.

## Choose an integration surface

| Surface | Best for | Merchant frontend | Server credential |
|---|---|---|---|
| Native S2S | PIX and provider-neutral payment flows | Merchant-owned | `xp_test_*` / `xp_live_*` |
| Checkout XPay | Hosted or embedded checkout | XPayments-hosted payment UI | `xp_test_*` / `xp_live_*` |
| Stripe-compatible Direct | Existing or new Stripe Payment Intents / Elements flows | Stripe.js / Payment Element | `xp_test_*` / `xp_live_*` |

## Environments

Use `xp_test_*` with TEST Stores and `xp_live_*` with LIVE Stores. XPayments rejects incompatible key/provider environment combinations.

A Stripe-compatible Store can also expose its browser-safe Stripe publishable key (`pk_test_*` / `pk_live_*`) for Stripe.js. Never expose the XPayments `xp_*` key in browser code.

## Money representation

Create APIs use **minor units** unless a method-specific guide says otherwise.

Examples:

- EUR `500` = €5.00
- BRL `500` = R$5.00

Merchant webhook payloads currently expose the transaction amount in **major units**. For example, `5` means €5.00 when the currency is EUR. See [Merchant Webhooks](guides/webhooks.md).

## Financial finality

Do not mark an order paid from a redirect, a browser callback or a newly-created payment object. The authoritative order transition should be driven by the XPayments transaction state and merchant webhook.

Typical lifecycle:

```text
Merchant backend creates payment
        ↓
XPayments binds Store + GatewayVault + Transaction
        ↓
Provider processes payment
        ↓
Provider webhook reaches XPayments
        ↓
XPayments Transaction becomes succeeded / failed / canceled
        ↓
Finance Core / Wallet ledger
        ↓
Merchant webhook
        ↓
Merchant order state
```

## Production base URLs

```text
API                    https://api.xpayments.digital
Native API             https://api.xpayments.digital/api/v1
Stripe-compatible      https://api.xpayments.digital/api/stripe/v1
Checkout               https://checkout.xpayments.digital
```

## Next steps

For Stripe.js / Payment Element integrations, continue with [Stripe Direct Overview](guides/stripe-direct.md).

For PIX, continue with [PIX S2S](guides/pix.md).
