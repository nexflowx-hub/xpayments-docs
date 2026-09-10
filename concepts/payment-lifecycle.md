# Payment Lifecycle

XPayments is asynchronous by design. A successful create call means a payment object exists; it does not necessarily mean funds have been received.

## Canonical lifecycle

```text
created / pending
→ requires_payment_method / requires_action / processing
→ succeeded
```

Alternative terminal paths include `failed` and `canceled`.

Exact provider states vary by integration surface. Merchant business logic should map them into a small local order state machine.

## Recommended merchant order states

```text
PAYMENT_PENDING
PAYMENT_PROCESSING
PAID
PAYMENT_FAILED
PAYMENT_CANCELED
```

Only authoritative server-side confirmation should transition an order to `PAID`.

## Stripe-compatible Direct

```text
Merchant backend
→ POST /api/stripe/v1/payment_intents
→ XPayments Transaction created/bound
→ client_secret returned
→ Stripe.js / Payment Element confirms
→ Stripe event
→ XPayments provider webhook verification
→ Transaction succeeded
→ Finance / WalletMovement
→ Merchant webhook
```

Persist both the provider `pi_*` and `metadata.nexflowx_transaction_id`.

## Native S2S

Native methods may return an action instead of immediate success. Examples include a PIX QR code/copy-paste string, bank-payment instructions or redirect/action data.

`pending` means the payment was initiated, not paid.

## Checkout XPay

The Checkout session has its own UX lifecycle. The Merchant still treats the underlying XPayments Transaction/webhook as financial authority. Returning the customer to a `successUrl` is not sufficient proof of payment.

## Money representation

Payment creation APIs use minor units unless a method guide explicitly states otherwise:

```text
EUR 25.99 → 2599
BRL 5.00   → 500
```

Merchant webhook payloads currently expose the normalized Transaction amount in major units:

```text
EUR 25.99 → 25.99
```

## Exactly-once business effects

Payments and networks can retry. Your application must tolerate repeated creates and repeated webhooks.

Use two independent controls:

```text
Payment create idempotency → one logical payment attempt
Webhook idempotency        → one logical fulfillment/order transition
```

A duplicate webhook must never duplicate inventory decrement, invoice issuance, fulfillment, emails or accounting entries.

## Financial availability vs payment success

A `succeeded` payment means XPayments recognized the provider payment as successful and recorded the corresponding financial movement. It does not necessarily mean funds are immediately withdrawable. Available balance, release date, reserves and payout rules are separate settlement concepts.