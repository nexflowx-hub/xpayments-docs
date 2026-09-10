# Settlement, Releases & D0 Rails

Payment success and fund availability are separate states.

## Payment success

A Transaction can become `succeeded` after XPayments verifies the provider event and records the payment ledger movement.

## Availability / release

Funds can remain pending until the configured provider/merchant release schedule is reached. Finance views can distinguish concepts such as:

```text
gross payment
provider/platform fees
merchant net
pending release
available balance
reserved balance
payout
```

Do not infer withdrawable balance from the fact that a payment is `succeeded`.

## Finance endpoints

Merchant finance modules can expose resources such as:

```text
GET /api/v1/finance/overview?currency=EUR
GET /api/v1/finance/releases?currency=EUR
GET /api/v1/finance/stores?currency=EUR
GET /api/v1/payout-statements?currency=EUR
```

These endpoints are intended for financial operations/observability, not payment authorization.

## D0 Rails — Premium

XPayments can offer accelerated **D0 operational rails** under eligible Premium arrangements. D0 availability is not a universal API guarantee and must not be hardcoded by merchant integrations.

Eligibility, funding source, risk controls, limits, fees, supported countries/currencies and operational settlement model are agreed/configured for each Merchant/Store.

For commercial/engineering assessment of D0 and advanced payment infrastructure, use the XPay Expert channel:

```text
https://xpay.expert
```

## Integration rule

Merchant checkout logic should remain independent of settlement tier:

```text
Payment finality → Transaction / Merchant webhook
Fund availability → Finance / releases / Premium rail configuration
```

This separation allows XPayments to change release schedules or activate a Premium D0 rail without forcing the merchant to rewrite checkout code.