# Idempotency & Errors

Reliable payment integrations must treat retries as a normal part of distributed systems.

## Idempotency-Key

For Stripe-compatible PaymentIntent creation, send a stable `Idempotency-Key` for the same logical payment attempt.

Example:

```text
order:ORD-123456:payment:1
```

Use the same key when retrying the same request after a timeout or uncertain network result.

Do not create a fresh random key for every retry. A new idempotency key represents a new logical payment attempt.

## Conflict protection

XPayments binds an idempotent create to the Store, amount and currency. Reusing the same logical reference/idempotency key with incompatible parameters can be rejected with a conflict response instead of silently creating another payment.

## Recommended retry algorithm

```text
1. Persist PaymentAttempt + idempotency key locally.
2. Send create request.
3. If response succeeds, persist pi_* and nexflowx_transaction_id.
4. If network result is uncertain, retry with the SAME key.
5. Do not generate a second payment attempt until the merchant explicitly starts one.
```

## Common HTTP classes

| HTTP | Meaning | Merchant action |
|---|---|---|
| `400` | Invalid request or parameters | Fix request; do not blindly retry |
| `401` | Missing/invalid Store API key | Fix credentials/environment |
| `404` | Resource not found or not owned by this Store | Check Store/payment binding |
| `409` | Idempotency/configuration conflict | Inspect existing attempt and parameters |
| `5xx` / `502` | XPayments/provider temporary failure | Retry safely with same idempotency key |

Provider error bodies can include additional Stripe-compatible fields. Applications should preserve useful error codes for observability without logging secrets or full sensitive payloads.

## Browser retries

Prevent duplicate submits in the checkout UI. A page refresh or double click must not automatically create a new PaymentIntent when a valid PaymentAttempt already exists.

## Webhook idempotency

Create idempotency protects payment creation. Merchant webhook idempotency protects fulfillment.

Treat them as separate controls:

```text
Create idempotency  → one logical PaymentIntent/Transaction
Webhook idempotency → one logical order transition/fulfillment
```

See [Merchant Webhooks](webhooks.md).
