# WestraPay MoMo Merchant Integration Guide

This guide explains how a merchant can integrate with the WestraPay MoMo API for collections and payouts in Kenya and Rwanda.

## Base URL

```text
https://connect.westrapay.com/api/v1
```

## API Credentials

WestraPay will issue each merchant an API username and API password.

| Field | Example |
| --- | --- |
| API username | `your_api_username` |
| API password | `your_api_password` |

Keep the API password private. Do not expose it in frontend or mobile applications. Generate tokens from your backend server.

## Generate Token

Use HTTP Basic Authentication to generate a bearer token.

```bash
curl --location 'https://connect.westrapay.com/api/v1/' \
  --user 'your_api_username:your_api_password'
```

Successful response:

```json
{
  "expires_at": "2026-06-30T12:17:08Z",
  "token": "YOUR_TOKEN_HERE"
}
```

Use the returned token in payment requests:

```http
Authorization: Bearer YOUR_TOKEN_HERE
```

## Service Codes

### Payin Service Codes

Use payin service codes to collect money from a customer.

| Country | Currency | Service Code | Use |
| --- | --- | --- | --- |
| Kenya | KES | `WP_PAYIN_KE` | Kenya mobile money collection |
| Kenya | KES | `WP_PAYIN_MPESA_KE` | M-Pesa Kenya collection |
| Rwanda | RWF | `WP_PAYIN_RW` | Rwanda mobile money collection |
| Rwanda | RWF | `WP_PAYIN_MTN_RW` | MTN Rwanda collection |
| Rwanda | RWF | `WP_PAYIN_AIRTEL_RW` | Airtel Rwanda collection |

### Payout Service Codes

Use payout service codes to send money to a recipient.

| Country | Currency | Service Code | Use |
| --- | --- | --- | --- |
| Kenya | KES | `WP_PAYOUT_KE` | Kenya mobile money payout |
| Kenya | KES | `WP_PAYOUT_MPESA_KE` | M-Pesa Kenya payout |
| Rwanda | RWF | `WP_PAYOUT_RW` | Rwanda mobile money payout |
| Rwanda | RWF | `WP_PAYOUT_MTN_RW` | MTN Rwanda payout |
| Rwanda | RWF | `WP_PAYOUT_AIRTEL_RW` | Airtel Rwanda payout |

## Initiate MoMo Transaction

Endpoint:

```http
POST /momo/transaction
```

Full URL:

```text
https://connect.westrapay.com/api/v1/momo/transaction
```

Request fields:

| Field | Required | Description |
| --- | --- | --- |
| `phone_number` | Yes | Customer or recipient phone number. Use international format where possible. |
| `amount` | Yes | Amount in the currency implied by the service code. |
| `service_code` | Yes | Payin or payout service code. |
| `callback_url` | Yes | Merchant webhook URL for final transaction status. |
| `external_id` | Yes | Merchant unique transaction reference. Must be unique for every request. |

## Kenya KES Payin Example

Collect KES from a customer in Kenya.

```bash
curl --location 'https://connect.westrapay.com/api/v1/momo/transaction' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_TOKEN_HERE' \
  --data '{
    "phone_number": "+254708000000",
    "amount": 100,
    "service_code": "WP_PAYIN_KE",
    "callback_url": "https://your-domain.com/westrapay/callback",
    "external_id": "merchant-ke-payin-10001"
  }'
```

## Kenya KES Payout Example

Send KES to a recipient in Kenya.

```bash
curl --location 'https://connect.westrapay.com/api/v1/momo/transaction' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_TOKEN_HERE' \
  --data '{
    "phone_number": "+254708000000",
    "amount": 100,
    "service_code": "WP_PAYOUT_KE",
    "callback_url": "https://your-domain.com/westrapay/callback",
    "external_id": "merchant-ke-payout-10001"
  }'
```

## Rwanda RWF Payin Example

Collect RWF from a customer in Rwanda.

```bash
curl --location 'https://connect.westrapay.com/api/v1/momo/transaction' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_TOKEN_HERE' \
  --data '{
    "phone_number": "+250790030353",
    "amount": 100,
    "service_code": "WP_PAYIN_RW",
    "callback_url": "https://your-domain.com/westrapay/callback",
    "external_id": "merchant-rw-payin-10001"
  }'
```

## Rwanda RWF Payout Example

Send RWF to a recipient in Rwanda.

```bash
curl --location 'https://connect.westrapay.com/api/v1/momo/transaction' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_TOKEN_HERE' \
  --data '{
    "phone_number": "+250790030353",
    "amount": 100,
    "service_code": "WP_PAYOUT_RW",
    "callback_url": "https://your-domain.com/westrapay/callback",
    "external_id": "merchant-rw-payout-10001"
  }'
```

## Initiation Responses

### Successful Initiation

This means WestraPay accepted the request for processing. The transaction is not final until a callback is received.

```json
{
  "message": "Payment initiation successful",
  "secureId": "c753bfa9b6ffdd45efef86660bc7adb8",
  "externalId": "merchant-rw-payin-10001"
}
```

### Failed Initiation

```json
{
  "error": "Failed to initiate transaction",
  "details": "Unable to initiate transaction at this time. Please try again."
}
```

For payouts, a failed initiation may also happen when the merchant payout wallet has insufficient balance.

## Callback Webhook

WestraPay sends a POST request to the `callback_url` supplied in the initiation request.

Your webhook should:

- Accept `POST` requests.
- Accept `Content-Type: application/json`.
- Return HTTP 200 after processing the callback.
- Use `externalId` and `secureId` for reconciliation.

### Successful Payin Callback

```json
{
  "transactionStatus": "COMPLETE",
  "transactionReport": "COMPLETE",
  "currency": "RWF",
  "amount": 100.00,
  "grossAmount": 100.00,
  "fee": 0.00,
  "netAmount": 100.00,
  "secureId": "c753bfa9b6ffdd45efef86660bc7adb8",
  "externalId": "merchant-rw-payin-10001"
}
```

### Failed Payin Callback

```json
{
  "transactionStatus": "FAILED",
  "transactionReport": "FAILED",
  "currency": "RWF",
  "amount": 100.00,
  "grossAmount": 100.00,
  "fee": 0.00,
  "netAmount": 100.00,
  "secureId": "c753bfa9b6ffdd45efef86660bc7adb8",
  "externalId": "merchant-rw-payin-10001"
}
```

### Successful Payout Callback

```json
{
  "transactionStatus": "COMPLETE",
  "transactionReport": "COMPLETE",
  "currency": "KES",
  "amount": 100.00,
  "grossAmount": 100.00,
  "fee": 0.00,
  "netAmount": 100.00,
  "secureId": "9af0b6c6d5a24f62a2f1cdb78c3a1001",
  "externalId": "merchant-ke-payout-10001"
}
```

### Failed Payout Callback

```json
{
  "transactionStatus": "FAILED",
  "transactionReport": "FAILED",
  "currency": "KES",
  "amount": 100.00,
  "grossAmount": 100.00,
  "fee": 0.00,
  "netAmount": 100.00,
  "secureId": "9af0b6c6d5a24f62a2f1cdb78c3a1001",
  "externalId": "merchant-ke-payout-10001"
}
```

## Reconciliation Rules

- Treat initiation success as pending.
- Mark payins as paid only after receiving `transactionStatus: COMPLETE`.
- Mark payouts as sent only after receiving `transactionStatus: COMPLETE`.
- Mark transactions as failed when `transactionStatus: FAILED`.
- Store both `externalId` and `secureId`.
- Use a new `external_id` for every request.
- Do not expose provider-specific errors to customers.

## Recommended Integration Flow

1. Generate a token using Basic Auth.
2. Initiate payin or payout using the correct service code.
3. Store `secureId`, `externalId`, amount, currency, and status as pending.
4. Wait for the callback.
5. Update the final transaction status from the callback.
6. Reconcile using `externalId`, `secureId`, `currency`, and `amount`.
