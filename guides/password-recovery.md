# Password Recovery

XPayments password recovery is designed to avoid account enumeration and token reuse.

## Request a reset link

```http
POST /api/v1/auth/forgot
Content-Type: application/json

{
  "email": "merchant@example.com"
}
```

The API returns the same accepted response whether or not the address belongs to an active Merchant. This prevents callers from using the endpoint to discover registered accounts.

The reset token is **never returned by the API** and must never be written to application logs.

## Reset link

When email delivery is configured, XPayments sends a link similar to:

```text
https://xpayments.digital/reset-password?token=<signed-token>
```

The token:

- expires after 30 minutes;
- is signed with the XPayments authentication secret;
- is scoped specifically to password recovery;
- is bound to a fingerprint of the Merchant's current password hash.

Because the token is bound to the current password hash, a successful password change automatically invalidates previously-issued reset links for that password state.

## Set the new password

```http
POST /api/v1/auth/reset
Content-Type: application/json

{
  "token": "<signed-token>",
  "password": "new-password"
}
```

Current password policy for this recovery contract:

```text
minimum: 8 characters
maximum: 128 characters
```

The Merchant frontend uses `/reset-password` so the user does not manually copy or paste the reset token. After a successful reset, the page removes the token from browser history.

## Email delivery

Password recovery email is a server-side operation. Production requires:

```text
JWT_SECRET
RESEND_API_KEY
XPAYMENTS_MAIL_FROM
XPAYMENTS_APP_URL=https://xpayments.digital
```

`RESEND_API_KEY` and mail-provider credentials must never be exposed through `NEXT_PUBLIC_*`, committed to Git, included in screenshots or placed in the Merchant application.

## Failure handling

An expired, malformed, already-invalidated or otherwise invalid reset link returns an explicit reset-token error to the reset page, but the initial `forgot` request continues to use a generic response.

## Operational validation before enabling LIVE

Validate the complete path with a controlled TEST Merchant:

```text
forgot request
→ generic HTTP accepted response
→ email received
→ /reset-password link opens
→ new password accepted
→ old password rejected
→ new password login succeeds
→ same reset link rejected on reuse
```

Do not enable production password recovery until the mail-sender domain and server-side environment variables are configured and this flow has passed end-to-end.
