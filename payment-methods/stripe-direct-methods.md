# Stripe Direct — Payment Methods

Stripe-compatible Direct keeps the Stripe PaymentIntent/Stripe.js interaction model while XPayments owns the Store route, provider credentials, transaction binding, finance and provider-webhook finality.

## Recommended: automatic payment methods

For most e-commerce Payment Element implementations:

```bash
curl https://api.xpayments.digital/api/stripe/v1/payment_intents \
  -u "xp_live_xxxxxxxxx:" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Idempotency-Key: order-1001-payment-1" \
  --data-urlencode "amount=2599" \
  --data-urlencode "currency=eur" \
  --data-urlencode "automatic_payment_methods[enabled]=true" \
  --data-urlencode "metadata[merchant_reference]=ORDER-1001"
```

Stripe/provider eligibility decides which enabled methods are returned/rendered for that PaymentIntent. This can vary by account, currency, country, amount, customer and provider capabilities.

## Explicit card-only intent

```text
payment_method_types[]=card
```

Use this when the product intentionally wants a card-only UI.

## Explicit provider methods

Where the Store/provider account supports them, the Stripe-compatible relay preserves Stripe-style form fields. Examples of method identifiers that may be used or surfaced include:

```text
card
bancontact
blik
ideal
eps
klarna
amazon_pay
link
mb_way
multibanco
bizum
satispay
```

Additional provider wallets/local methods may become eligible through automatic payment methods. Do not hardcode a provider method as guaranteed without checking the Store's configured provider capabilities.

## Payment Element

Browser:

```ts
const stripe = Stripe(
  process.env.NEXT_PUBLIC_XPAYMENTS_STRIPE_PUBLISHABLE_KEY!
);

const elements = stripe.elements({ clientSecret });
```

`pk_*` is browser-safe for Stripe.js. The Store's `xp_*` key remains server-side.

## Confirmation

```ts
await stripe.confirmPayment({
  elements,
  confirmParams: {
    return_url: "https://shop.example.com/checkout/complete"
  },
  redirect: "if_required"
});
```

Some methods require redirects or asynchronous processing. Your return page must query your own backend/order state rather than mark the order paid.

## Example eligibility from a LIVE certification

A controlled LIVE EUR PaymentIntent on an eligible XPayments Store returned multiple provider methods including card, Bancontact, Klarna, Multibanco, Link, MB WAY, Amazon Pay, Bizum and Satispay. This is an observed certification example, **not a universal static list**.

## Apple Pay / Google Pay

Wallet buttons and availability depend on Stripe.js/Payment Element configuration, browser/device capability, domain/provider configuration and account eligibility. Keep these on the provider UI surface; do not send wallet credentials to XPayments.

## Method-specific business rules

Even when a method appears in `payment_method_types`, your application must support its flow characteristics:

- immediate vs asynchronous settlement;
- redirect/return flows;
- `requires_action` / SCA;
- customer country/phone requirements;
- delayed confirmation;
- cancellation/expiration.

XPayments finality remains the same for every method: provider event → verified XPayments webhook processing → Transaction state → Merchant webhook.