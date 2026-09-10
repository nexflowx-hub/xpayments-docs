# Stores & Onboarding

A Store is the XPayments boundary that binds a merchant integration to a currency, routing profile, provider connection, API keys and merchant webhook.

## Create a Store

Merchant-authenticated API:

```http
POST /api/v1/merchant/stores
Authorization: Bearer <merchant-jwt>
Content-Type: application/json
```

Example:

```json
{
  "name": "Example Store Europe",
  "currency": "EUR",
  "domain": "shop.example.com",
  "theme": "dark",
  "primaryColor": "#111827",
  "logoUrl": "https://shop.example.com/logo.png",
  "successUrl": "https://shop.example.com/checkout/complete",
  "webhookUrl": "https://shop.example.com/api/webhooks/xpayments"
}
```

XPayments generates the `storeCode`. New Stores are created in `draft` and must receive a valid processing/routing configuration before accepting payment traffic.

If `webhookUrl` is supplied, XPayments also creates an active Merchant webhook and returns its signing secret **at creation time**. Store the value immediately in a server-side secret manager.

## List Stores

```http
GET /api/v1/merchant/stores
Authorization: Bearer <merchant-jwt>
```

## Retrieve one Store

```http
GET /api/v1/merchant/stores/{storeId}
Authorization: Bearer <merchant-jwt>
```

The detail response can include Store-scoped API key metadata, Webhooks and GatewayVault metadata. Provider secret credentials are not exposed.

## Store lifecycle

```text
draft
  ↓ configure provider / routing / webhook / API key
active
  ↓ optional operational suspension
paused / inactive (where configured)
```

A Store must be active before payment APIs accept traffic.

## Currency

Treat Store currency as part of the payment contract. Payment requests should match the Store's configured currency. For multi-currency businesses, prefer separate Stores or an explicitly approved routing model rather than silently changing currencies client-side.

## Recommended onboarding sequence

1. Create the Store in `draft`.
2. Confirm domain, currency and return URLs.
3. Configure ProviderAccount / ProviderConnection / GatewayVault through the XPayments control plane.
4. Activate VNEXT/ORCHESTRATED processing.
5. Configure the Merchant webhook and save its signing secret.
6. Create a TEST API key when a sandbox provider route exists.
7. Run a non-financial authentication/routing probe.
8. Run a low-value end-to-end TEST payment.
9. Create/enable LIVE API credentials only after the TEST path passes.
10. Run one controlled LIVE certification payment and audit Transaction + WalletMovement + Merchant webhook.

## Store-scoped isolation

API keys are Store-scoped. A PaymentIntent retrieved through the Stripe-compatible relay must be bound to the same Merchant and Store as the authenticating key. Do not share Store keys between unrelated storefronts.

## Wallet note

Wallet accounting may be Merchant + currency scoped rather than one wallet per Store. Do not assume that each Store has a separate financial wallet merely because Store traffic is isolated. Use Store/Transaction attribution for per-Store reporting.