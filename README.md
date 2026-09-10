# XPayments Developer Documentation

Official technical documentation for **XPayments** — Store onboarding, payment APIs, Stripe-compatible routing, Checkout XPay, Native S2S, webhooks, financial finality and Merchant Platform management.

## Start here

New Merchant integration:

```text
Merchant account
→ create/configure Store
→ activate VNEXT/ORCHESTRATED route
→ create Store API key
→ configure Merchant webhook
→ choose payment surface
→ certify TEST
→ enable LIVE
```

Read [Getting Started](getting-started.md), then [Stores & Onboarding](concepts/stores-and-onboarding.md).

## Integration surfaces

| Surface | Use case | Server credential | Browser/provider UI |
|---|---|---|---|
| Native S2S | PIX, MB WAY, Multibanco, Bizum and approved direct methods | `xp_test_*` / `xp_live_*` | Merchant-owned UI/action |
| Checkout XPay | Hosted/embedded provider-neutral checkout | `xp_test_*` / `xp_live_*` | XPayments Checkout |
| Stripe-compatible Direct | PaymentIntents + Stripe.js/Payment Element | `xp_test_*` / `xp_live_*` | `pk_test_*` / `pk_live_*` |

## Management APIs

The Merchant Platform API uses a Merchant JWT for Store, API Key, Webhook, Transaction, Wallet and Finance operations. Payment APIs use Store-scoped `xp_*` keys. These are intentionally separate security boundaries.

## Production endpoints

```text
API                  https://api.xpayments.digital
Native / Merchant v1 https://api.xpayments.digital/api/v1
Stripe-compatible v1 https://api.xpayments.digital/api/stripe/v1
Checkout             https://checkout.xpayments.digital
Portal                https://xpayments.digital
```

## Payment methods

See [Payment Methods](payment-methods/overview.md) for the current support matrix and [Native S2S Method Examples](payment-methods/native-s2s-examples.md) for method-specific request examples.

XPayments distinguishes between **CERTIFIED**, **SUPPORTED** and **PROVIDER-DEPENDENT** methods. A method appearing in a provider account does not automatically mean every XPayments Store can use it.

## Security model

- `xp_test_*` / `xp_live_*` are server-side Store secrets.
- `pk_test_*` / `pk_live_*` are browser-safe only for Stripe.js / Elements.
- Merchant webhook signing secrets are server-side and verify `x-nexflowx-signature`.
- Provider `sk_*`, `rk_*` and provider inbound webhook secrets stay inside XPayments GatewayVaults.
- Never send PAN/CVV to XPayments or a merchant custom backend; use a provider tokenization/Element surface.
- Never treat redirects, QR generation or browser state as financial finality.

## Financial finality

```text
provider event
→ XPayments verified inbound webhook
→ Transaction state
→ Finance / WalletMovement
→ Merchant webhook
→ Merchant order state
```

See [Payment Lifecycle](concepts/payment-lifecycle.md) and [Merchant Webhooks](guides/webhooks.md).

## API specifications

Two OpenAPI 3.1 specifications are maintained:

- `openapi/xpayments.yaml` — Native, Checkout and Stripe-compatible payment APIs.
- `openapi/merchant-platform.yaml` — Merchant-authenticated management/observability API.

See [API Reference](reference/api-reference.md).

## Settlement and Premium rails

Payment success and fund availability are separate. See [Settlement, Releases & D0 Rails](operations/settlement-and-d0.md) for finance concepts and Premium D0 positioning.

## AI-ready integration

Use [AI Integration Playbooks](guides/ai-integration.md) to hand the XPayments contract to ChatGPT, Claude, Cursor, Gemini, Z.AI or another coding agent without sharing real secrets.

## Source of truth

This repository is the canonical public developer documentation and OpenAPI source. GitBook is the publication layer; `xpayments.digital/doc` is the visual quick-start/onboarding surface. Material contract changes should be synchronized across all three.