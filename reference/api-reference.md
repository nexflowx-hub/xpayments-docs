# API Reference

XPayments publishes two OpenAPI specifications because payment traffic and Merchant management use different authentication models.

## Payments API

Specification:

```text
openapi/xpayments.yaml
```

Covers:

- health;
- Native S2S `/api/v1/payments/charge`;
- Checkout session creation;
- Stripe-compatible Direct `/api/stripe/v1/payment_intents` operations.

Authentication is primarily a Store-scoped `xp_test_*` / `xp_live_*` key.

## Merchant Platform API

Specification:

```text
openapi/merchant-platform.yaml
```

Covers:

- Merchant authentication;
- Store creation/list/detail;
- API Key management;
- Merchant Webhooks;
- Transactions;
- Wallets.

Authentication uses the Merchant JWT.

## Base URLs

```text
Merchant / Native v1     https://api.xpayments.digital/api/v1
Stripe-compatible v1     https://api.xpayments.digital/api/stripe/v1
Checkout UI              https://checkout.xpayments.digital
```

## Contract precedence

Use the following order when resolving ambiguity:

1. published versioned OpenAPI contract;
2. method/integration guide in this repository;
3. current XPayments dashboard configuration for the Store;
4. XPayments support/engineering confirmation for provider-dependent capabilities.

Do not infer undocumented provider capabilities from a single Store or PaymentIntent.