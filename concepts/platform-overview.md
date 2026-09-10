# Platform Overview

XPayments separates merchant applications from provider credentials and provider-specific settlement logic.

## Core model

```text
Merchant
  └─ Store (currency + domain + status)
      ├─ API Keys (Store-scoped)
      ├─ Merchant Webhooks
      ├─ Processing Profile (LEGACY / VNEXT)
      └─ Provider Connection
           └─ GatewayVault

Payment / Checkout Session
  └─ XPayments Transaction
      ├─ Provider payment id
      ├─ Finance / fee
      ├─ WalletMovement
      └─ Merchant webhook delivery
```

A **Store** is the routing and isolation boundary for payment traffic. API keys belong to a Store. Provider credentials never belong in merchant application code.

## Integration surfaces

XPayments currently exposes three payment surfaces:

| Surface | Base path | Best use |
|---|---|---|
| Native S2S | `/api/v1/payments/charge` | PIX and direct provider-neutral/native method flows |
| Checkout XPay | `/api/v1/checkout` + `checkout.xpayments.digital` | Hosted/embedded payment UX |
| Stripe-compatible Direct | `/api/stripe/v1` | Stripe PaymentIntents + Stripe.js / Payment Element |

The Merchant Platform API under `/api/v1` also exposes authenticated management/observability resources such as Stores, API Keys, Webhooks, Transactions, Wallets and Finance.

## Runtime generations

### VNEXT / ORCHESTRATED

Recommended production mode. XPayments owns routing, provider connection selection, provider webhook verification, transaction state, ledger and merchant delivery.

### VNEXT / OBSERVED

Used where XPayments observes provider traffic without owning the entire payment creation path. Capabilities are intentionally more limited.

### LEGACY

Backward-compatibility surface. New integrations should target VNEXT/ORCHESTRATED unless XPayments explicitly assigns another mode.

## Financial finality

Payment creation is not financial finality.

```text
Merchant creates payment
→ Provider processes payment
→ Provider webhook reaches XPayments
→ XPayments verifies provider event
→ Transaction state changes
→ Finance Core / WalletMovement
→ Merchant webhook
→ Merchant Order state
```

A redirect, browser callback, `client_secret`, QR-code creation or `pending` state must never be used alone to fulfill an order.

## Production endpoints

```text
API                  https://api.xpayments.digital
Native API           https://api.xpayments.digital/api/v1
Stripe-compatible    https://api.xpayments.digital/api/stripe/v1
Checkout             https://checkout.xpayments.digital
Dashboard / Docs     https://xpayments.digital
```

## Contract ownership

This repository is the public source of truth for developer-facing contracts. Runtime behavior that has been certified in production is documented as such. Experimental, provider-dependent or not-yet-certified capabilities are explicitly labeled instead of being presented as guaranteed.