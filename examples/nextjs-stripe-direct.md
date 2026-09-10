# Example — Next.js + Stripe Direct

This example uses Next.js App Router, XPayments Stripe-compatible Direct on the server and Stripe Payment Element in the browser.

## Environment variables

```env
XPAYMENTS_API_KEY=xp_test_...
XPAYMENTS_STRIPE_BASE_URL=https://api.xpayments.digital/api/stripe/v1
XPAYMENTS_WEBHOOK_SECRET=...
NEXT_PUBLIC_XPAYMENTS_STRIPE_PUBLISHABLE_KEY=pk_test_...
```

Only the `pk_*` variable is browser-safe.

## Create PaymentIntent route

`app/api/payments/create-intent/route.ts`

```ts
import { NextResponse } from "next/server";

export async function POST(request: Request) {
  const { orderId } = await request.json();

  // Load the Order from your DB and recompute amount server-side.
  const amount = 2599;

  const body = new URLSearchParams();
  body.set("amount", String(amount));
  body.set("currency", "eur");
  body.set("automatic_payment_methods[enabled]", "true");
  body.set("metadata[merchant_reference]", orderId);

  const upstream = await fetch(
    `${process.env.XPAYMENTS_STRIPE_BASE_URL}/payment_intents`,
    {
      method: "POST",
      headers: {
        Authorization: `Bearer ${process.env.XPAYMENTS_API_KEY}`,
        "Content-Type": "application/x-www-form-urlencoded",
        "Idempotency-Key": `order:${orderId}:payment:1`
      },
      body,
      cache: "no-store"
    }
  );

  const intent = await upstream.json();

  if (!upstream.ok) {
    return NextResponse.json(intent, { status: upstream.status });
  }

  // Persist intent.id + intent.metadata.nexflowx_transaction_id here.

  return NextResponse.json({
    clientSecret: intent.client_secret,
    paymentIntentId: intent.id,
    xpaymentsTransactionId: intent.metadata?.nexflowx_transaction_id
  });
}
```

## Browser Payment Element

Use `@stripe/stripe-js` and `@stripe/react-stripe-js` or Stripe.js directly. The browser receives only the PaymentIntent `client_secret` and the Store's publishable `pk_*`.

```ts
const result = await stripe.confirmPayment({
  elements,
  confirmParams: {
    return_url: `${window.location.origin}/checkout/complete`
  },
  redirect: "if_required"
});
```

Do not mark the order paid from this browser result.

## Merchant webhook route

`app/api/webhooks/xpayments/route.ts`

```ts
import crypto from "crypto";
import { NextResponse } from "next/server";

export async function POST(request: Request) {
  const rawBody = await request.text();
  const signature = request.headers.get("x-nexflowx-signature") || "";
  const secret = process.env.XPAYMENTS_WEBHOOK_SECRET || "";

  const expected = crypto
    .createHmac("sha256", secret)
    .update(rawBody)
    .digest("hex");

  const a = Buffer.from(signature);
  const b = Buffer.from(expected);
  const valid = a.length === b.length && crypto.timingSafeEqual(a, b);

  if (!valid) {
    return NextResponse.json({ error: "Invalid signature" }, { status: 401 });
  }

  const event = JSON.parse(rawBody);

  // Use event.transaction_id to find the local PaymentAttempt.
  // Apply an idempotent state transition in a DB transaction.

  return NextResponse.json({ received: true });
}
```

## Return page

The return page should poll/read your own backend Order state. It must not issue a second PaymentIntent or infer success from query parameters alone.

## Acceptance test

```text
create once
→ refresh/double click does not duplicate PaymentIntent
→ Payment Element confirms
→ optional SCA/redirect works
→ XPayments webhook validates
→ Order becomes PAID exactly once
```