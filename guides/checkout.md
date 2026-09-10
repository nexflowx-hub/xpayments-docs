# Checkout XPay

Checkout XPay is the provider-neutral payment UI for Merchants that want XPayments to own the payment interaction layer while the Merchant owns the Order/business flow.

## 1. Create a session

```http
POST https://api.xpayments.digital/api/v1/checkout/session
Authorization: Bearer xp_live_...
Content-Type: application/json
```

```json
{
  "amount": 2599,
  "currency": "EUR",
  "reference": "ORDER-1001",
  "customerEmail": "customer@example.com",
  "metadata": {
    "order_id": "ORDER-1001"
  }
}
```

`2599` means EUR 25.99.

Success is `201` and returns data such as:

```json
{
  "success": true,
  "data": {
    "sessionId": "uuid",
    "checkoutUrl": "https://checkout.xpayments.digital/pay/uuid",
    "storeCode": "STORE-CODE",
    "expiresAt": "2026-09-10T18:00:00.000Z"
  }
}
```

Sessions are Store-scoped and currently expire after the configured checkout lifetime (30 minutes in the v3.1 implementation).

## 2. Hosted checkout

Redirect/open:

```text
https://checkout.xpayments.digital/pay/{sessionId}
```

## 3. Load session

Public checkout UI can load:

```http
GET /api/v1/checkout/session/{sessionId}
```

The response includes Store branding/config, amount, currency, reference, status and eligible payment methods derived from Store routing.

## 4. Initiate the selected method

Checkout UI calls:

```http
POST /api/v1/checkout/initiate
Content-Type: application/json
```

```json
{
  "sessionId": "uuid",
  "paymentMethod": "mb_way",
  "customer": {
    "name": "Customer Name",
    "email": "customer@example.com",
    "phone": "+351911111111"
  }
}
```

XPayments resolves the configured Store route and creates/reuses the underlying Transaction/provider payment.

## Checkout method codes

The current checkout contract recognizes method codes including:

```text
card
mb_way
multibanco
bizum
pix
apple_pay
google_pay
bancontact
blik
ideal
eps
klarna
amazon_pay
```

A code is displayed only when the Store has a corresponding routing rule. Provider/account eligibility still applies.

## Embedded checkout

Where enabled, XPay can be embedded as a payment-only surface. The merchant still creates the session server-side. Treat iframe/embedded completion as UX; financial finality remains the XPayments Transaction/webhook.

## Recommended Merchant flow

```text
Order created
→ backend creates XPay Session
→ browser opens Hosted/Embedded checkout
→ customer selects payment method
→ Checkout calls /checkout/initiate
→ provider payment/action
→ provider webhook → XPayments
→ Transaction final state
→ Finance / WalletMovement
→ Merchant webhook
→ Order PAID/FAILED/etc.
```

## Duplicate protection

Do not create a new checkout session on every browser refresh. Persist the session with the Order/PaymentAttempt and reuse it while valid. The checkout API rejects already-paid sessions and expired sessions.

## Security

- `xp_*` stays server-side.
- The public load-session endpoint must not reveal API keys or GatewayVault secrets.
- Do not fulfill from `successUrl` or client-side redirects.
- Keep the Merchant webhook idempotent.

## Choosing Checkout vs Stripe Direct

Use Checkout XPay when you want provider-neutral UI and centralized method presentation. Use Stripe-compatible Direct when you deliberately want Stripe.js/Payment Element in the Merchant frontend.