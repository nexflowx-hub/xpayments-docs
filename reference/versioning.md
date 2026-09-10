# Versioning & Compatibility

XPayments has multiple version dimensions. Keep them separate when integrating or reporting incidents.

## Engine version

`GET /api/health` currently identifies the XPayments engine version, for example:

```json
{
  "success": true,
  "version": "3.1.0",
  "engine": "XPayments",
  "status": "ONLINE"
}
```

This is the backend engine release and is not the same thing as a URL API version.

## Native / Merchant API version

Canonical path:

```text
/api/v1
```

This includes Merchant Platform resources, Checkout and Native payment creation.

## Stripe-compatible API version

Canonical path:

```text
/api/stripe/v1
```

The relay intentionally mirrors Stripe v1 PaymentIntent request semantics for supported operations. A merchant may optionally send `Stripe-Version`; XPayments forwards/preserves it where supported by the configured provider route.

## Processing generation

Stores can also have a processing generation independent of URL version:

```text
VNEXT
LEGACY
```

VNEXT can operate in modes such as ORCHESTRATED or OBSERVED. New payment integrations should target active VNEXT/ORCHESTRATED Stores.

## Compatibility policy

- New optional response fields may be added without changing the URL version.
- Clients must ignore unknown response fields.
- Existing documented field semantics should not change silently.
- Breaking request/response changes should receive a new API version or an explicit migration window.
- Provider-dependent fields can evolve as providers add/deprecate payment methods.
- Legacy endpoints are maintained for compatibility but should not be the basis of new integrations.

## Documentation labels

Pages may use:

```text
CERTIFIED
SUPPORTED
PROVIDER-DEPENDENT
EXPERIMENTAL
LEGACY
DEPRECATED
```

`CERTIFIED` means an end-to-end path has been validated. It does not mean every payment method or Merchant account has identical provider capabilities.

## Stripe-compatible certification

The card-based Stripe-compatible flow has been validated through TEST and a controlled LIVE payment for:

```text
PaymentIntent create
→ provider confirmation
→ Stripe webhook verification
→ XPayments Transaction succeeded
→ finance fee/net
→ WalletMovement exactly once
```

Merchant webhook delivery is independently dependent on the Merchant endpoint and its XPayments signing-secret configuration.

## Change management

Every material contract change should update:

1. this repository;
2. OpenAPI specification(s);
3. GitBook publication;
4. the `xpayments.digital/doc` quick-start surface where relevant;
5. the changelog.

The Git repository is the canonical history.