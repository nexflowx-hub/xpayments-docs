# Checkout XPay

Checkout XPay is the provider-neutral surface for merchants that do not want to manage provider SDKs directly in their checkout frontend.

## Create a session

```text
POST https://api.xpayments.digital/api/v1/checkout/session
```

Use a Store-scoped XPayments API key server-side. Never expose `xp_test_*` or `xp_live_*` in browser code.

## Hosted checkout

After creating a session, send the customer to:

```text
https://checkout.xpayments.digital/pay/{sessionId}
```

## Embedded checkout

Where enabled by the merchant integration, XPay can be used in an embedded payment-only surface. The merchant should still create the session server-side and treat the checkout UI as a payment interaction layer rather than financial finality.

## Finality

A redirect back to the merchant is not proof of payment. The merchant Order should be finalized from the XPayments transaction/webhook state.

## Recommended flow

```text
Merchant Order
→ backend creates XPay session
→ customer opens Hosted/Embedded checkout
→ payment provider processes payment
→ XPayments verifies provider webhook
→ XPayments Transaction final state
→ Merchant webhook
→ Merchant Order state
```

## Security

Keep Store API keys on the server. Do not expose GatewayVault credentials or provider secret keys to the merchant frontend.
