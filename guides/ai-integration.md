# AI Integration Playbooks

Use this page when handing a repository to ChatGPT, Claude, Cursor, Gemini, Z.AI or another coding agent.

## Universal integration prompt

```text
You are implementing XPayments in this repository.

First inspect the framework, package manager, routing model, order/cart storage, database models, authentication, environment-variable pattern and existing payment code. Do not modify code until you understand the existing architecture.

Choose exactly one XPayments surface for the primary payment flow:

A) Native S2S
   Base: https://api.xpayments.digital/api/v1
   Use for PIX and provider-neutral direct payment methods.

B) Checkout XPay
   Create session server-side, then use https://checkout.xpayments.digital/pay/{sessionId} or the supported embedded surface.

C) Stripe-compatible Direct
   Base: https://api.xpayments.digital/api/stripe/v1
   Use when the project uses Stripe Payment Intents / Stripe.js / Payment Element.

Security rules:
- xp_test_* / xp_live_* are server-side secrets only.
- pk_test_* / pk_live_* are browser-safe only for Stripe.js / Elements.
- Never request or expose provider sk_*, rk_* or provider webhook secrets.
- Never send PAN/CVV to the merchant backend or XPayments.
- Recompute order totals server-side.

Financial finality:
- Do not mark an order paid from a redirect, client callback or newly-created payment.
- Persist the XPayments transaction identifier.
- Finalize the order only from the verified XPayments merchant webhook / authoritative backend state.

Webhook contract:
- Header: x-nexflowx-signature
- Signature: lowercase hex HMAC-SHA256 of the exact raw request body using XPAYMENTS_WEBHOOK_SECRET.
- Read request.text() first, validate the HMAC, then JSON.parse(rawBody).
- Correlate the local order primarily by transaction_id / stored xpaymentsTransactionId.
- Make processing idempotent.

Stripe-compatible specifics:
- POST bodies are application/x-www-form-urlencoded.
- Preserve a stable Idempotency-Key per logical payment attempt.
- Prefer automatic_payment_methods[enabled]=true for Payment Element when broad method availability is desired.
- Create PaymentIntent on the merchant backend through XPayments.
- Confirm with Stripe.js / Payment Element in the browser.
- Support SCA, requires_action, redirects and processing states.

At the end provide:
1. files changed;
2. migrations/model changes;
3. required environment variables (names only, never real secrets);
4. tests run;
5. build result;
6. an end-to-end validation checklist;
7. any remaining blockers.
```

## Stripe-compatible implementation checklist for an AI agent

The agent should produce a flow equivalent to:

```text
Order
→ PaymentAttempt + stable idempotency key
→ merchant backend creates XPayments PaymentIntent
→ store pi_* + nexflowx_transaction_id
→ return client_secret to browser
→ Stripe Payment Element
→ stripe.confirmPayment()
→ provider event
→ XPayments transaction/finance processing
→ XPayments merchant webhook
→ HMAC verification using raw body
→ idempotent Order transition
```

## Do not give an AI real credentials

Use environment-variable placeholders in prompts and code reviews:

```text
XPAYMENTS_API_KEY
XPAYMENTS_WEBHOOK_SECRET
NEXT_PUBLIC_XPAYMENTS_STRIPE_PUBLISHABLE_KEY
XPAYMENTS_STRIPE_BASE_URL
```

Configure real values directly in the hosting/environment manager.
