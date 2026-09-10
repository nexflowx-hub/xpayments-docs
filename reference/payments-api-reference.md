# Payments API Reference

Canonical OpenAPI 3.1 specification:

`openapi/xpayments.yaml`

This reference covers the Store-scoped payment surfaces currently documented by XPayments.

## Native S2S

### Create payment

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml" path="/api/v1/payments/charge" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml
{% endopenapi %}

## Checkout XPay

### Create checkout session

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml" path="/api/v1/checkout/session" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml
{% endopenapi %}

### Load checkout session

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml" path="/api/v1/checkout/session/{sessionId}" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml
{% endopenapi %}

### Initiate checkout method

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml" path="/api/v1/checkout/initiate" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml
{% endopenapi %}

## Stripe-compatible Direct

### Create PaymentIntent

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml" path="/api/stripe/v1/payment_intents" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml
{% endopenapi %}

### Retrieve PaymentIntent

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml" path="/api/stripe/v1/payment_intents/{paymentIntentId}" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml
{% endopenapi %}

### Update PaymentIntent

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml" path="/api/stripe/v1/payment_intents/{paymentIntentId}" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml
{% endopenapi %}

### Confirm PaymentIntent

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml" path="/api/stripe/v1/payment_intents/{paymentIntentId}/confirm" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml
{% endopenapi %}

### Cancel PaymentIntent

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml" path="/api/stripe/v1/payment_intents/{paymentIntentId}/cancel" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml
{% endopenapi %}

### Capture PaymentIntent

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml" path="/api/stripe/v1/payment_intents/{paymentIntentId}/capture" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/xpayments.yaml
{% endopenapi %}

## Finality

API response success is not always payment success. Use the Merchant webhook / authoritative XPayments Transaction state for final order fulfillment.