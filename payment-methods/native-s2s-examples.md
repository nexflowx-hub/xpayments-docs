# Native S2S — Method Examples

Native payments use:

```text
POST https://api.xpayments.digital/api/v1/payments/charge
```

Authentication:

```http
Authorization: Bearer xp_test_...
```

or:

```http
x-api-key: xp_test_...
```

All examples use minor units. Replace references with unique merchant payment-attempt references.

## Card

Card collection should normally use a PCI-appropriate provider UI/tokenization layer. Do not post raw PAN/CVV to XPayments.

```json
{
  "amount": 2599,
  "currency": "EUR",
  "payment_method_types": ["card"],
  "reference": "ORDER-1001",
  "metadata": { "order_id": "ORDER-1001" }
}
```

For a new web checkout using Stripe cards, prefer [Stripe-compatible Direct](../guides/stripe-direct.md) with Stripe.js / Payment Element.

## PIX

```json
{
  "amount": 500,
  "currency": "BRL",
  "payment_method_types": ["pix"],
  "reference": "ORDER-BR-1001",
  "customer": {
    "name": "Customer Name",
    "document": "12345678900"
  },
  "metadata": {
    "order_id": "ORDER-BR-1001",
    "description": "Order payment"
  }
}
```

Use the returned `action` to render QR Code / Copia e Cola. Never create a second PIX only to get another representation of the same payment.

## MB WAY

```json
{
  "amount": 1299,
  "currency": "EUR",
  "payment_method_types": ["mb_way"],
  "reference": "ORDER-PT-1001",
  "customer": {
    "name": "Customer Name",
    "email": "customer@example.com",
    "phone": "+351911111111"
  },
  "metadata": { "order_id": "ORDER-PT-1001" }
}
```

Native validation accepts Portuguese numbers normalized to `+351` format. The method is asynchronous; do not mark the order paid at create time.

## Multibanco

```json
{
  "amount": 2500,
  "currency": "EUR",
  "payment_method_types": ["multibanco"],
  "reference": "ORDER-MB-1001",
  "customer": {
    "name": "Customer Name",
    "email": "customer@example.com"
  },
  "metadata": { "order_id": "ORDER-MB-1001" }
}
```

Present provider instructions/action returned by XPayments. Settlement is asynchronous.

## Bizum

```json
{
  "amount": 1999,
  "currency": "EUR",
  "payment_method_types": ["bizum"],
  "reference": "ORDER-ES-1001",
  "customer": {
    "name": "Customer Name",
    "email": "customer@example.com",
    "phone": "+34600000001"
  },
  "metadata": {
    "order_id": "ORDER-ES-1001",
    "return_url": "https://shop.example.com/checkout/complete"
  }
}
```

Current Native validation requires EUR, a Spanish `+34` phone number and an amount between EUR 0.50 and EUR 5,000.00.

## Bancontact, BLIK, iDEAL, EPS, Klarna, Amazon Pay

These codes are recognized in the current Stripe-backed Checkout/payment service when the Store routing/provider supports them:

```text
bancontact
blik
ideal
eps
klarna
amazon_pay
```

For new browser integrations, prefer Stripe-compatible Direct + Payment Element or Checkout XPay rather than building method-specific browser logic against Native S2S without an explicitly certified Store contract.

## Wallet methods

Apple Pay and Google Pay should be integrated through a supported provider wallet surface (for example Stripe Payment Element) or Checkout XPay. Do not attempt to collect wallet credentials through a custom Native JSON payload.

## Finality

All methods share the same rule:

```text
create/action/redirect ≠ paid
verified XPayments Transaction succeeded + Merchant webhook = payment finality
```

See [Payment Lifecycle](../concepts/payment-lifecycle.md) and [Merchant Webhooks](../guides/webhooks.md).