# Example — PHP Native S2S

Example using PHP cURL for a Store-scoped Native payment.

## PIX

```php
<?php

$apiKey = getenv('XPAYMENTS_API_KEY');
$endpoint = 'https://api.xpayments.digital/api/v1/payments/charge';

$payload = [
    'amount' => 500,
    'currency' => 'BRL',
    'payment_method_types' => ['pix'],
    'reference' => 'ORDER-BR-1001',
    'customer' => [
        'name' => 'Customer Name',
        'document' => '12345678900',
    ],
    'metadata' => [
        'order_id' => 'ORDER-BR-1001',
        'description' => 'Order payment',
    ],
];

$ch = curl_init($endpoint);
curl_setopt_array($ch, [
    CURLOPT_POST => true,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => [
        'Content-Type: application/json',
        'x-api-key: ' . $apiKey,
    ],
    CURLOPT_POSTFIELDS => json_encode($payload, JSON_UNESCAPED_SLASHES),
    CURLOPT_TIMEOUT => 30,
]);

$body = curl_exec($ch);
$status = curl_getinfo($ch, CURLINFO_HTTP_CODE);

if ($body === false) {
    throw new RuntimeException(curl_error($ch));
}

curl_close($ch);
$response = json_decode($body, true, flags: JSON_THROW_ON_ERROR);

if ($status < 200 || $status >= 300) {
    throw new RuntimeException(
        $response['error']['message'] ?? 'XPAYMENTS_PAYMENT_CREATE_FAILED'
    );
}

$transactionId = $response['transactionId'] ?? $response['data']['transactionId'] ?? null;
$action = $response['action'] ?? $response['data']['action'] ?? null;

// Persist transactionId with the local order.
// Render QR/copy-paste from $action.
// Do not mark the order PAID until the verified Merchant webhook arrives.
```

## Server-only secrets

Store `XPAYMENTS_API_KEY` in the hosting/server secret manager. Never embed it into generated JavaScript, HTML, mobile application bundles or public repositories.

## Webhook verification

PHP can verify the Merchant webhook using the raw request body:

```php
$raw = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_NEXFLOWX_SIGNATURE'] ?? '';
$secret = getenv('XPAYMENTS_WEBHOOK_SECRET');
$expected = hash_hmac('sha256', $raw, $secret);

if (!hash_equals($expected, $signature)) {
    http_response_code(401);
    exit('Invalid signature');
}

$event = json_decode($raw, true, flags: JSON_THROW_ON_ERROR);
```

Process the event idempotently using `transaction_id` as the primary XPayments correlation key.