# Stripe-compatible Direct

The Stripe-compatible Direct surface lets a merchant keep the Stripe Payment Intents / Stripe.js integration model while XPayments owns the provider routing, GatewayVault, finance binding, ledger and merchant webhook finality.

## Base URL

```text
https://api.xpayments.digital/api/stripe/v1
```

## Authentication

Use a Store-scoped XPayments API key server-side.

Supported forms:

```http
Authorization: Bearer xp_live_...
```

or Stripe-style HTTP Basic:

```text
xp_live_...:
```

`x-api-key` is also supported for server integrations.

Never expose `xp_test_*` or `xp_live_*` in browser JavaScript.

## Available PaymentIntent endpoints

```text
POST /payment_intents
GET  /payment_intents/:id
POST /payment_intents/:id
POST /payment_intents/:id/confirm
POST /payment_intents/:id/cancel
POST /payment_intents/:id/capture
```

POST requests use `application/x-www-form-urlencoded`, like Stripe v1.

## Recommended e-commerce flow

1. Create an Order or PaymentAttempt locally.
2. Recalculate the final amount on the merchant backend.
3. Create the PaymentIntent through XPayments.
4. Persist both the Stripe `pi_*` and XPayments `nexflowx_transaction_id`.
5. Return only the `client_secret` and public identifiers to the browser.
6. Confirm the payment using Stripe.js / Payment Element.
7. Treat redirect/browser state as UX only.
8. Mark the order paid only when the XPayments merchant webhook confirms success.

## Create a PaymentIntent

```bash
curl https://api.xpayments.digital/api/stripe/v1/payment_intents \
  -u "xp_live_xxxxxxxxx:" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Idempotency-Key: order-12345-payment-1" \
  --data-urlencode "amount=2599" \
  --data-urlencode "currency=eur" \
  --data-urlencode "automatic_payment_methods[enabled]=true" \
  --data-urlencode "metadata[merchant_reference]=order-12345"
```

`2599` means €25.99.

### Automatic payment methods

For a broad Stripe Payment Element integration, prefer:

```text
automatic_payment_methods[enabled]=true
```

The provider decides which methods are actually eligible according to account configuration, currency, country, capabilities, amount and customer context. Do not hardcode a method list as a platform guarantee.

For a card-only integration, use:

```text
payment_method_types[]=card
```

## Server-side example

```ts
const amount = 2599;
const orderId = "ORD-123456";

const body = new URLSearchParams();
body.set("amount", String(amount));
body.set("currency", "eur");
body.set("automatic_payment_methods[enabled]", "true");
body.set("metadata[merchant_reference]", orderId);

const response = await fetch(
  `${process.env.XPAYMENTS_STRIPE_BASE_URL}/payment_intents`,
  {
    method: "POST",
    headers: {
      Authorization: `Bearer ${process.env.XPAYMENTS_API_KEY}`,
      "Content-Type": "application/x-www-form-urlencoded",
      "Idempotency-Key": `order:${orderId}:payment:1`,
    },
    body,
  }
);

const intent = await response.json();

if (!response.ok) {
  throw new Error(
    intent?.error?.message || "XPAYMENTS_PAYMENT_INTENT_FAILED"
  );
}
```

Persist at minimum:

```text
intent.id
intent.metadata.nexflowx_transaction_id
intent.metadata.merchant_reference
idempotency key
```

Do not persist provider secret credentials.

## Response to the browser

A merchant backend may return a reduced object such as:

```json
{
  "clientSecret": "pi_..._secret_...",
  "paymentIntentId": "pi_...",
  "xpaymentsTransactionId": "uuid",
  "orderId": "ORD-123456"
}
```

The complete `client_secret` should not be logged.

## Payment Element

Use the Store's browser-safe Stripe publishable key (`pk_test_*` or `pk_live_*`).

```ts
const stripe = Stripe(
  process.env.NEXT_PUBLIC_XPAYMENTS_STRIPE_PUBLISHABLE_KEY!
);

const elements = stripe.elements({ clientSecret });
const paymentElement = elements.create("payment");
paymentElement.mount("#payment-element");
```

Never expose the XPayments `xp_*` server key in the browser.

## Confirm the payment

```ts
const result = await stripe.confirmPayment({
  elements,
  confirmParams: {
    return_url: "https://www.example.com/checkout/complete",
  },
  redirect: "if_required",
});
```

Your frontend must support asynchronous and action-required states such as:

```text
requires_action
processing
succeeded
```

Redirect-based methods can leave the page and return later. The return page is not proof of financial success.

## Order finality

The authoritative flow is:

```text
Stripe provider event
→ XPayments verifies provider webhook
→ XPayments Transaction state
→ XPayments Finance Core / WalletMovement
→ XPayments merchant webhook
→ Merchant order state
```

The merchant webhook is documented in [Merchant Webhooks](webhooks.md).

## Idempotency

Use a stable `Idempotency-Key` for the same logical payment attempt.

Good:

```text
order:ORD-123456:payment:1
```

Do not generate a fresh random idempotency key for every network retry.

A replay of the same create request with the same parameters and key returns the existing PaymentIntent rather than creating another XPayments Transaction.

See [Idempotency & Errors](idempotency-and-errors.md).

## Security boundary

The merchant application receives only the credentials it needs:

```text
Merchant backend      xp_test_* / xp_live_*
Merchant browser      pk_test_* / pk_live_*
XPayments GatewayVault sk_* / rk_* / provider webhook secret
```

PAN/CVV must never be posted to the merchant backend or XPayments. Stripe.js / Elements handles card data directly with Stripe.

## Certification status

The Stripe-compatible card flow has been validated end-to-end in TEST and LIVE for create → confirm → provider webhook → XPayments Transaction → Finance Core → WalletMovement. Merchant webhook delivery depends on the merchant endpoint being correctly configured with the XPayments signing secret and raw-body verification contract.
