# Developer Secret Management

XPayments separates ordinary Developer metadata from sensitive credentials. API key values and Merchant webhook signing secrets must never appear in normal list endpoints, browser logs, analytics payloads or public source code.

## API key lifecycle

A Store-scoped API key is created through the Merchant Platform and returned in full only on sensitive operations.

```text
POST /api/v1/api-keys
POST /api/v1/api-keys/{id}/reveal
POST /api/v1/api-keys/{id}/rotate
DELETE /api/v1/api-keys/{id}
```

All four operations require an authenticated Merchant JWT and are restricted to API keys belonging to one of that Merchant's Stores.

### Reveal

```http
POST /api/v1/api-keys/{id}/reveal
Authorization: Bearer <merchant-jwt>
```

The sensitive response is delivered with cache-prevention headers. Treat the returned `fullKey` as a server-side secret.

Do not expose `xp_test_*` or `xp_live_*` to browser JavaScript, mobile applications or public repositories.

### Rotate

Rotation is intentionally explicit:

```http
POST /api/v1/api-keys/{id}/rotate
Authorization: Bearer <merchant-jwt>
Content-Type: application/json

{
  "confirm": true
}
```

Rotation replaces the current key immediately. The previous key stops authenticating requests as soon as the operation succeeds.

Recommended sequence:

```text
1. Prepare access to the deployment secret manager.
2. Rotate the API key.
3. Copy the new fullKey once.
4. Update the Merchant backend secret.
5. Redeploy/restart the Merchant backend if required.
6. Run a non-financial authentication probe.
7. Hide the secret again.
```

Do not rotate a production key while you are unable to update the consuming backend.

## Merchant webhook signing secret

Merchant webhooks use a Store-level signing secret for the normalized XPayments webhook contract.

```text
POST /api/v1/webhooks/{id}/reveal
POST /api/v1/webhooks/{id}/rotate-secret
```

### Reveal signing secret

```http
POST /api/v1/webhooks/{id}/reveal
Authorization: Bearer <merchant-jwt>
```

The response includes the current `secret` only for a webhook owned by the authenticated Merchant.

### Rotate signing secret

```http
POST /api/v1/webhooks/{id}/rotate-secret
Authorization: Bearer <merchant-jwt>
Content-Type: application/json

{
  "confirm": true
}
```

The old secret becomes invalid immediately. Update `XPAYMENTS_WEBHOOK_SECRET` on the Merchant receiver before expecting further verified deliveries.

## Browser behavior

The XPayments dashboard reveals a sensitive value only temporarily and automatically hides it again. Copy a value directly to the server-side secret manager and avoid keeping it in notes, screenshots or chat logs.

## What these operations do not expose

Merchant secret management does **not** expose provider credentials stored inside GatewayVault, including:

- Stripe `sk_*` / `rk_*`;
- provider-native webhook signing secrets;
- MisticPay credentials;
- other physical provider credentials.

Those credentials remain on the XPayments provider boundary.
