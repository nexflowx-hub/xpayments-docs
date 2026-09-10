# Merchant Webhooks

XPayments sends normalized merchant webhooks after provider events have been verified and the XPayments transaction state has been processed.

Do not consume provider-native Stripe webhooks directly in the merchant application unless you deliberately maintain a separate provider integration. For the XPayments integration, use the normalized XPayments contract below.

## Endpoint

Configure one HTTPS endpoint per Store, for example:

```text
POST https://www.example.com/api/webhooks/xpayments
```

The endpoint should respond directly and should not rely on an HTTP redirect.

## Signature header

When a webhook secret is configured, XPayments sends:

```http
Content-Type: application/json
x-nexflowx-signature: <hex-hmac-sha256>
```

The signature is:

```text
HMAC-SHA256(raw_request_body, XPAYMENTS_WEBHOOK_SECRET)
```

encoded as lowercase hexadecimal.

## Verify the raw body first

The signature must be computed over the exact bytes/text received. Do not parse the JSON and then reserialize it before verification.

Next.js App Router example:

```ts
import crypto from "crypto";
import { NextResponse } from "next/server";

export async function POST(request: Request) {
  const rawBody = await request.text();

  const signature =
    request.headers.get("x-nexflowx-signature") || "";

  const secret = process.env.XPAYMENTS_WEBHOOK_SECRET;

  if (!secret || !signature) {
    return NextResponse.json(
      { error: "Missing webhook signature" },
      { status: 401 }
    );
  }

  const expected = crypto
    .createHmac("sha256", secret)
    .update(rawBody)
    .digest("hex");

  const suppliedBuffer = Buffer.from(signature, "utf8");
  const expectedBuffer = Buffer.from(expected, "utf8");

  const valid =
    suppliedBuffer.length === expectedBuffer.length &&
    crypto.timingSafeEqual(suppliedBuffer, expectedBuffer);

  if (!valid) {
    return NextResponse.json(
      { error: "Invalid signature" },
      { status: 401 }
    );
  }

  const event = JSON.parse(rawBody);

  // Process idempotently.

  return NextResponse.json({ received: true });
}
```

## Current payload

```json
{
  "event": "payment_intent.succeeded",
  "transaction_id": "4a558fd4-0000-0000-0000-000000000000",
  "reference": "SC-0123456789abcdef0123456789abcdef",
  "amount": 25.99,
  "currency": "EUR",
  "status": "succeeded",
  "method": "card",
  "timestamp": "2026-09-10T17:16:31.826Z"
}
```

Fields:

| Field | Meaning |
|---|---|
| `event` | XPayments event type |
| `transaction_id` | XPayments Transaction UUID; recommended correlation key |
| `reference` | XPayments transaction reference; may be an internal deterministic `SC-*` reference |
| `amount` | Transaction amount in major units |
| `currency` | ISO-style currency code such as `EUR` |
| `status` | XPayments transaction status |
| `method` | Payment method known to XPayments |
| `timestamp` | Dispatch timestamp |

### Important amount difference

Payment creation APIs use minor units. Merchant webhook payloads currently use the XPayments transaction amount in major units.

Example EUR:

```text
Create PaymentIntent: amount=2599
Merchant webhook:     amount=25.99
```

## Correlate by transaction_id

For Stripe-compatible creates, XPayments injects `metadata[nexflowx_transaction_id]` into the provider PaymentIntent. Persist that value with the merchant Order or PaymentAttempt.

When the webhook arrives, resolve the order by:

```text
transaction_id → local xpaymentsTransactionId
```

Do not assume the webhook `reference` is the merchant order id. XPayments can use an internal deterministic `SC-*` transaction reference.

## Events

Prepare for at least:

```text
payment_intent.succeeded
payment_intent.payment_failed
payment_intent.processing
payment_intent.canceled
```

Suggested order transitions:

```text
payment_intent.succeeded      → PAID
payment_intent.payment_failed → PAYMENT_FAILED
payment_intent.processing     → PAYMENT_PROCESSING
payment_intent.canceled       → PAYMENT_CANCELED
```

The merchant application should define transitions that fit its own order state machine.

## Idempotency

Webhook handling must be idempotent. A duplicate success delivery must not duplicate:

- order completion;
- fulfillment;
- inventory decrement;
- invoice creation;
- email delivery;
- accounting side effects.

Use a database transaction and a conditional state transition or a dedicated processed-event record.

## Response codes

Return a `2xx` response only after the event is accepted for processing. Invalid signatures should return `401` or another explicit `4xx`.

A `2xx` response is recorded by XPayments as a successful delivery. Non-2xx responses are delivery failures and should be investigated.

## Secret management

`XPAYMENTS_WEBHOOK_SECRET` is a merchant secret. Store it in the application's server-side environment manager. Never expose it with `NEXT_PUBLIC_`, commit it to Git, or paste it into client-side code.
