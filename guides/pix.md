# PIX S2S

PIX uses the Native XPayments S2S surface.

## Endpoint

```text
POST https://api.xpayments.digital/api/v1/payments/charge
```

Authentication is server-side with a Store-scoped XPayments API key.

```http
x-api-key: xp_live_...
Content-Type: application/json
```

Never expose `xp_*` in browser code.

## Request

```json
{
  "amount": 500,
  "currency": "BRL",
  "payment_method_types": ["pix"],
  "reference": "ORDER-12345",
  "customer": {
    "name": "Customer Name",
    "document": "12345678900"
  },
  "metadata": {
    "order_id": "ORDER-12345",
    "description": "Order payment"
  }
}
```

`500` means R$5.00.

Use a unique logical reference per payment attempt.

## Response

A successfully-created PIX payment is normally still pending. The response can include an `action` object with fields such as:

```text
copyPaste
pixString
qrCode
qrCodeBase64
qrCodeUrl
```

Use the returned action to render the QR Code and Copia e Cola. Do not create another PIX just to obtain a QR Code.

## Finality

`pending` means the PIX was created, not paid.

Fulfill the order only after the XPayments transaction/webhook reports `succeeded`.

## Merchant UI

A typical PIX checkout should provide:

- QR Code;
- Copia e Cola value;
- copy button;
- pending indicator;
- expiration guidance when available;
- backend-driven status refresh or merchant webhook processing.

## Security

Do not trust amount, order totals or customer document data blindly from the browser. Validate/recompute server-side according to your application rules.
