# API Keys & Webhook Management

This guide covers the Merchant-facing developer controls available after a Store exists.

## Authentication model

Dashboard/management endpoints use the Merchant JWT:

```http
Authorization: Bearer <merchant-jwt>
```

Payment creation endpoints use a Store-scoped XPayments API key:

```http
Authorization: Bearer xp_test_...
```

or:

```http
x-api-key: xp_test_...
```

Do not confuse the two credentials.

## Create an API key

```http
POST /api/v1/api-keys
Authorization: Bearer <merchant-jwt>
Content-Type: application/json
```

```json
{
  "storeId": "<store-uuid>",
  "name": "Production payment backend",
  "environment": "live",
  "scopes": ["payments_write"]
}
```

For payment creation, `payments_write` is the recommended scope. Production payment keys should only be created for an active ORCHESTRATED Store with a matching provider environment.

The full `xp_*` value is returned at creation. Save it immediately in the server's secret manager.

## List keys

```http
GET /api/v1/api-keys
Authorization: Bearer <merchant-jwt>
```

The list is intended for metadata/preview, environment, scopes and last-used information. Do not design a client that depends on recovering provider credentials.

## Revoke a key

```http
DELETE /api/v1/api-keys/{apiKeyId}
Authorization: Bearer <merchant-jwt>
```

Safe rotation sequence:

```text
create replacement
→ deploy replacement to merchant backend
→ run non-financial/auth probe
→ validate payment path
→ revoke previous key
```

## Create a Merchant webhook

```http
POST /api/v1/webhooks
Authorization: Bearer <merchant-jwt>
Content-Type: application/json
```

```json
{
  "storeId": "<store-uuid>",
  "url": "https://merchant.example/api/webhooks/xpayments",
  "events": [
    "payment_intent.succeeded",
    "payment_intent.payment_failed",
    "payment_intent.processing",
    "payment_intent.canceled"
  ]
}
```

XPayments generates a Merchant signing secret. Save the returned secret immediately. It is separate from any provider/Stripe `whsec_*` used internally by XPayments.

## List webhooks

```http
GET /api/v1/webhooks
Authorization: Bearer <merchant-jwt>
```

The normal list response intentionally does not expose the full signing secret.

## Update a webhook

```http
PUT /api/v1/webhooks/{webhookId}
Authorization: Bearer <merchant-jwt>
Content-Type: application/json
```

Update URL/events without changing provider webhook configuration.

## Delete a webhook

```http
DELETE /api/v1/webhooks/{webhookId}
Authorization: Bearer <merchant-jwt>
```

## Current secret UX

Current Merchant UI behavior is intentionally one-time display when a signing secret is created. A future dedicated reveal/rotate operation may be added, but integrations must not assume a full secret is available from normal list APIs.

## Webhook authentication

When a signing secret exists, XPayments sends:

```http
x-nexflowx-signature: <lowercase-hex-hmac-sha256>
```

Verify the exact raw request body using the Store's Merchant signing secret. See [Merchant Webhooks](webhooks.md).

## Separation from provider webhooks

Never copy the Stripe/provider inbound webhook secret into the Merchant application. Provider webhook credentials belong to the XPayments GatewayVault. Merchant applications only need the XPayments Merchant webhook secret generated for their Store.