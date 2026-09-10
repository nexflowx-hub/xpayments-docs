# Routing & Orchestration

XPayments keeps provider routing outside merchant application code.

## Routing objects

A production VNEXT Store can be associated with:

```text
Store
→ Store Processing Profile
→ Provider Connection
→ Provider Account
→ GatewayVault
```

The Store identifies the merchant-facing integration. The processing profile decides how XPayments handles traffic. The provider connection points to the operational provider account and target GatewayVault.

## Processing modes

### ORCHESTRATED

XPayments owns payment creation/routing and financial binding. This is the preferred mode for Native S2S, Checkout XPay and Stripe-compatible Direct.

### OBSERVED

XPayments observes/records selected provider traffic. Do not assume that all write/payment endpoints are available.

### LEGACY

Backward-compatible routing used by older Stores. New projects should not deliberately target LEGACY unless instructed by XPayments.

## GatewayVault

GatewayVault is the protected provider credential boundary. It can contain provider secret keys, publishable configuration and provider webhook credentials.

Merchant applications should never receive physical provider secret credentials from the Vault.

## Method routing

A Store can map payment-method codes to provider aliases, for example conceptually:

```json
{
  "card": "stripe-store-eu",
  "mb_way": "stripe-store-eu",
  "multibanco": "stripe-store-eu",
  "pix": "provider-pix-br"
}
```

The routing configuration is managed by XPayments. Merchant code sends the payment method, amount, currency and business reference; it should not hardcode physical provider credentials.

## Stripe-compatible routing

The Stripe-compatible relay resolves the Store from the `xp_*` key, verifies that the active VNEXT route is Stripe-compatible, binds an XPayments Transaction and only then calls the configured Stripe account.

Provider PaymentIntents are Store-bound. A different Store key cannot be used to access a PaymentIntent merely because its `pi_*` id is known.

## Provider webhooks vs Merchant webhooks

These are different flows:

```text
Provider webhook
Stripe / PIX provider → XPayments
purpose: verify provider event and settle internal state

Merchant webhook
XPayments → merchant endpoint
purpose: notify merchant of normalized XPayments state
```

Provider webhook secrets stay in GatewayVault. Merchant webhook signing secrets belong to the Merchant integration and are stored separately.

## Do not route in the browser

The browser should never choose a GatewayVault or provider secret. Routing decisions must remain server-side/XPayments-controlled so they can be changed without redeploying merchant frontend code.