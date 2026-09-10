# Getting Started

XPayments separates Merchant management, Store-scoped payment credentials and provider credentials. Start by preparing the Store, then choose one payment integration surface.

## 1. Authenticate to the Merchant Platform

Merchant management uses a JWT obtained from:

```text
POST https://api.xpayments.digital/api/v1/auth/login
```

Use the JWT for Store/API Key/Webhook/Transaction management. Do not use the Merchant JWT as a payment API key.

## 2. Create or select a Store

Canonical Merchant routes:

```text
GET  /api/v1/merchant/stores
POST /api/v1/merchant/stores
GET  /api/v1/merchant/stores/{storeId}
```

New Stores are created as `draft`. Before accepting payments, configure/activate the Store processing route, GatewayVault/provider connection and Merchant webhook.

See [Stores & Onboarding](concepts/stores-and-onboarding.md).

## 3. Configure credentials

Payment APIs use a Store-scoped key:

```text
xp_test_*  → TEST
xp_live_*  → LIVE
```

For Stripe.js/Elements, the Store may also expose a provider publishable key:

```text
pk_test_* / pk_live_*
```

Only `pk_*` is browser-safe. See [Environments & Credentials](concepts/environments-and-credentials.md).

## 4. Configure Merchant webhook

Create one HTTPS Merchant webhook per Store and save the XPayments signing secret at creation. Verify `x-nexflowx-signature` over the exact raw request body.

See [Merchant Webhooks](guides/webhooks.md).

## 5. Choose an integration surface

| Surface | Best for | Merchant frontend | Server credential |
|---|---|---|---|
| Native S2S | PIX and approved direct/local methods | Merchant-owned | `xp_test_*` / `xp_live_*` |
| Checkout XPay | Hosted or embedded checkout | XPayments-hosted payment UI | `xp_test_*` / `xp_live_*` |
| Stripe-compatible Direct | Stripe PaymentIntents / Elements | Stripe.js / Payment Element | `xp_test_*` / `xp_live_*` |

See [Payment Methods](payment-methods/overview.md) for method availability by surface.

## 6. Money representation

Payment create APIs use **minor units** unless a method-specific guide says otherwise:

```text
EUR 5.00 → 500
BRL 5.00 → 500
```

Merchant webhook payloads currently expose Transaction amounts in **major units**:

```text
EUR 5.00 → 5
```

## 7. Financial finality

Never mark an Order paid from a redirect, QR generation, `client_secret` or browser callback.

```text
payment create
→ provider processing
→ provider webhook reaches XPayments
→ verified Transaction state
→ Finance / WalletMovement
→ XPayments Merchant webhook
→ Merchant Order state
```

## 8. TEST before LIVE

Recommended release sequence:

```text
Store draft
→ routing/provider configured
→ TEST API key
→ TEST E2E
→ webhook verification
→ LIVE key/route
→ controlled LIVE certification
→ production traffic
```

Keep TEST and LIVE keys/routes isolated.

## Production base URLs

```text
API                    https://api.xpayments.digital
Native / Merchant v1   https://api.xpayments.digital/api/v1
Stripe-compatible v1   https://api.xpayments.digital/api/stripe/v1
Checkout                https://checkout.xpayments.digital
```

## Continue

- [Stripe-compatible Direct](guides/stripe-direct.md)
- [Checkout XPay](guides/checkout.md)
- [PIX S2S](guides/pix.md)
- [Native S2S Method Examples](payment-methods/native-s2s-examples.md)
- [API Reference](reference/api-reference.md)
