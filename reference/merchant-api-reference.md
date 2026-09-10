# Merchant Platform API Reference

Canonical OpenAPI 3.1 specification:

`openapi/merchant-platform.yaml`

Merchant Platform operations use the Merchant JWT and are separate from Store-scoped payment API keys.

## Authentication

### Login

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/auth/login" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

### Current Merchant

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/auth/me" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

## Stores

### List Stores

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/merchant/stores" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

### Create Store

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/merchant/stores" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

### Retrieve Store

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/merchant/stores/{storeId}" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

## API Keys

### List API Keys

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/api-keys" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

### Create API Key

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/api-keys" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

### Revoke API Key

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/api-keys/{apiKeyId}" method="delete" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

## Merchant Webhooks

### List Webhooks

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/webhooks" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

### Create Webhook

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/webhooks" method="post" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

### Update Webhook

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/webhooks/{webhookId}" method="put" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

### Delete Webhook

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/webhooks/{webhookId}" method="delete" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

## Transactions

### List Transactions

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/transactions" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

### Transaction statistics

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/transactions/stats" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

### Retrieve Transaction

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/transactions/{transactionId}" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

## Wallets

### List wallets

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/wallets" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}

### Wallet movements

{% openapi src="https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml" path="/wallets/movements" method="get" %}
https://raw.githubusercontent.com/nexflowx-hub/xpayments-docs/main/openapi/merchant-platform.yaml
{% endopenapi %}
