# Merchant Platform API

The Merchant Platform API powers authenticated dashboard and management operations. It is distinct from Store-scoped payment APIs.

Base:

```text
https://api.xpayments.digital/api/v1
```

## Authentication

Login:

```http
POST /auth/login
Content-Type: application/json
```

```json
{
  "email": "merchant@example.com",
  "password": "your-password"
}
```

Success returns a Merchant JWT. Use it for private management endpoints:

```http
Authorization: Bearer <merchant-jwt>
```

## Merchant profile

```text
GET /merchant/profile
```

## Stores

```text
GET  /merchant/stores
POST /merchant/stores
GET  /merchant/stores/{storeId}
```

Store creation returns `draft` by default. Provider routing must be configured/activated before accepting payments.

## API Keys

```text
GET    /api-keys
POST   /api-keys
DELETE /api-keys/{apiKeyId}
```

Payment keys are Store-scoped and environment-specific. `payments_write` is the recommended payment scope for ORCHESTRATED Stores.

## Merchant Webhooks

```text
GET    /webhooks
POST   /webhooks
PUT    /webhooks/{webhookId}
PATCH  /webhooks/{webhookId}
DELETE /webhooks/{webhookId}
```

The create operation generates a Merchant signing secret. Normal list operations do not expose it.

## Transactions

```text
GET /transactions
GET /transactions/stats
GET /transactions/{transactionId}
```

Common filters include page, limit, status, gateway, currency and reference.

## Wallets

```text
GET /wallets
GET /wallets/movements
GET /wallets/payouts
GET /wallets/deposits
```

Wallet accounting and Store attribution are separate concepts. A Merchant may have one wallet per currency while multiple Stores contribute transactions.

## Analytics

```text
GET /analytics/overview
```

## Finance

The Merchant application also consumes finance/release/payout resources under the v3.1/v4 finance contract. Availability depends on the enabled Merchant modules.

Typical resources include:

```text
GET /finance/overview
GET /finance/releases
GET /finance/stores
GET /payout-statements
```

Do not use dashboard projections as payment confirmation. Payment finality remains Transaction/webhook based.

## Gateway Vault management

Authenticated GatewayVault metadata/management endpoints exist under:

```text
/api/v1/gateway-vault
```

Provider secrets are privileged configuration. Public integration docs intentionally do not provide instructions for extracting secret provider credentials from GatewayVault.

## Standard response envelope

Most Merchant Platform endpoints use:

```json
{
  "success": true,
  "data": {}
}
```

Errors generally use:

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message"
  }
}
```

Some older/legacy endpoints can still return a simpler `message` field. Integrations should prefer the canonical envelope where documented.