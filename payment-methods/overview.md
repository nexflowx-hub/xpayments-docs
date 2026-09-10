# Payment Methods

XPayments exposes payment methods through different integration surfaces. Availability depends on Store configuration, provider account capabilities, currency, customer country and the selected XPayments surface.

## Support matrix

| Method | Native S2S | Checkout XPay | Stripe Direct / Elements | Notes |
|---|---|---|---|---|
| Cards | Supported | Supported | Supported / certified | Use Stripe.js/Elements for browser card collection |
| PIX | Supported for enabled BRL Stores | Supported when routed | Provider-dependent | Native PIX returns QR/copy-paste action |
| MB WAY | Supported on configured EUR Stripe Stores | Supported | Available when Stripe account is eligible | Portuguese phone required in Native flow |
| Multibanco | Supported on configured EUR Stripe Stores | Supported | Available when eligible | Async payment; bank instructions/action |
| Bizum | Supported on configured EUR Stripe Stores | Supported | Available when eligible | Spanish phone; Native amount rules apply |
| Bancontact | Provider-dependent | Supported when routed | Automatic/explicit Stripe method | Belgium/EUR eligibility rules apply at provider |
| BLIK | Provider-dependent | Supported when routed | Automatic/explicit Stripe method | Provider/account eligibility |
| iDEAL / Wero | Provider-dependent | Supported when routed | Automatic/explicit Stripe method | Availability can evolve with provider migration |
| EPS | Provider-dependent | Supported when routed | Automatic/explicit Stripe method | Austria-focused provider method |
| Klarna | Provider-dependent | Supported when routed | Automatic/explicit Stripe method | Eligibility depends on amount/customer/provider |
| Amazon Pay | Provider-dependent | Supported when routed | Automatic/explicit Stripe method | Provider capability required |
| Link | No dedicated Native contract | Provider-dependent | Automatic Stripe method | Rendered by Payment Element when eligible |
| Apple Pay | No dedicated Native contract | Supported when configured | Through Stripe Elements / wallets | Domain/device/provider eligibility required |
| Google Pay | No dedicated Native contract | Supported when configured | Through Stripe Elements / wallets | Device/browser/provider eligibility required |
| Satispay | No dedicated Native contract | Provider-dependent | Automatic Stripe method when eligible | Dynamic eligibility |
| Revolut Pay | No dedicated Native contract | Provider-dependent | Automatic/provider-dependent | Dynamic eligibility |
| TWINT | No dedicated Native contract | Provider-dependent | Automatic/provider-dependent | Switzerland/provider eligibility |
| Samsung Pay | No dedicated Native contract | Provider-dependent | Provider-dependent | Wallet availability varies by provider/account |
| Kakao Pay | No dedicated Native contract | Provider-dependent | Provider-dependent | Korea/provider eligibility |
| Naver Pay | No dedicated Native contract | Provider-dependent | Provider-dependent | Korea/provider eligibility |
| PAYCO | No dedicated Native contract | Provider-dependent | Provider-dependent | Korea/provider eligibility |

**Important:** `Supported when routed` means the XPayments Checkout/Store model knows the method code and can expose it when a valid provider route exists. It is not a promise that every Merchant account has the provider capability enabled.

## Recommended integration strategy

### Broadest Stripe method coverage

Use Stripe-compatible Direct with Payment Element and:

```text
automatic_payment_methods[enabled]=true
```

Do not hardcode the list returned by one PaymentIntent as a permanent platform contract.

### Provider-neutral hosted UX

Use Checkout XPay. The Store's active routing rules determine which methods are displayed.

### Native method-specific UX

Use `/api/v1/payments/charge` when the Merchant wants to own the UI and consume an XPayments action directly, such as PIX QR/copy-paste or MB WAY/Bizum/Multibanco flows.

## Certification labels

Documentation uses these labels:

- **CERTIFIED** — tested end-to-end through XPayments, provider event, Transaction and finance/ledger path.
- **SUPPORTED** — implemented in the XPayments contract for configured Stores.
- **PROVIDER-DEPENDENT** — XPayments can route/render it, but provider capability and account configuration decide real availability.
- **NOT YET CERTIFIED** — do not treat as production-certified until an end-to-end validation is completed.

## Amounts and asynchronous methods

Creation APIs generally use minor units. Many bank/wallet methods are asynchronous. A successful create response or customer redirect is not payment finality. Fulfill only after XPayments reports the final transaction state through the verified Merchant webhook.