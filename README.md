# XPayments Developer Documentation

Official developer documentation for **XPayments**.

XPayments exposes three integration surfaces:

1. **Native S2S API** — direct server-to-server payments such as PIX and other provider-neutral methods.
2. **Checkout XPay** — hosted or embedded checkout.
3. **Stripe-compatible Direct API** — Stripe Payment Intents and Stripe.js / Payment Element with XPayments routing, finance and webhook finality.

## Production endpoints

- API: `https://api.xpayments.digital`
- Native API: `https://api.xpayments.digital/api/v1`
- Stripe-compatible relay: `https://api.xpayments.digital/api/stripe/v1`
- Checkout: `https://checkout.xpayments.digital`

## Security model

- `xp_test_*` / `xp_live_*` are **server-side secrets**.
- `pk_test_*` / `pk_live_*` are browser-safe Stripe publishable keys for Stripe.js / Elements only.
- Provider secret keys, restricted keys and provider webhook signing secrets remain inside XPayments GatewayVaults.
- Never treat redirects or browser state as financial finality. Finality comes from XPayments transaction state and merchant webhooks.

## Documentation source of truth

This repository is the canonical source for the public XPayments developer documentation and OpenAPI definition. GitBook is the publication layer.

Start with [Getting Started](getting-started.md) or jump directly to [Stripe-compatible Direct](guides/stripe-direct.md).
