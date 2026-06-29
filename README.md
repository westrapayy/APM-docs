# WestraPay MoMo Merchant Integration Guide

Merchant credentials:

| Field | Value |
| --- | --- |
| API username | `username` |
| API password | `password` |

Base URL:

```text
https://connect.westrapay.com/api/v1
```

## Generate Token

```bash
curl --location 'https://connect.westrapay.com/api/v1/' \
  --user 'username:password'
```

Use the returned token as:

```http
Authorization: Bearer YOUR_TOKEN_HERE
```

## Service Codes

| Country | Currency | Service Code | Use |
| --- | --- | --- | --- |
| Kenya | KES | `WP_PAYIN_KE` | Kenya collection |
| Kenya | KES | `WP_PAYIN_MPESA_KE` | M-Pesa Kenya collection |
| Rwanda | RWF | `WP_PAYIN_RW` | Rwanda collection |
| Rwanda | RWF | `WP_PAYIN_MTN_RW` | MTN Rwanda collection |
| Rwanda | RWF | `WP_PAYIN_AIRTEL_RW` | Airtel Rwanda collection |

## Kenya KES Collection

```bash
curl --location 'https://connect.westrapay.com/api/v1/momo/transaction' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_TOKEN_HERE' \
  --data '{
    "phone_number": "+254708000000",
    "amount": 100,
    "service_code": "WP_PAYIN_KE",
    "callback_url": "https://your-domain.com/westrapay/callback",
    "external_id": "flexify-ke-order-10001"
  }'
```

## Rwanda RWF Collection

```bash
curl --location 'https://connect.westrapay.com/api/v1/momo/transaction' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_TOKEN_HERE' \
  --data '{
    "phone_number": "+250790030353",
    "amount": 100,
    "service_code": "WP_PAYIN_RW",
    "callback_url": "https://your-domain.com/westrapay/callback",
    "external_id": "flexify-rw-order-10001"
  }'
```

## Success Initiation

```json
{
  "message": "Payment initiation successful",
  "secureId": "c753bfa9b6ffdd45efef86660bc7adb8",
  "externalId": "flexify-rw-order-10001"
}
```

## Callback Success

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
  "externalId": "flexify-rw-order-10001"
}
```

## Callback Failed

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
  "externalId": "flexify-rw-order-10001"
}
```

Important notes:

- Use a new `external_id` for every request.
- Initiation success is pending until the final callback is received.
- Mark orders paid only when `transactionStatus` is `COMPLETE`.
- Keep the API password private and generate tokens from your backend.
