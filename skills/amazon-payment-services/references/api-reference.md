# Amazon Payment Services - API Reference

Complete API reference for Amazon Payment Services integration.

---

## Table of Contents

1. [Endpoints](#endpoints)
2. [Authentication](#authentication)
3. [Payment Commands](#payment-commands)
4. [Management Commands](#management-commands)
5. [Service Commands](#service-commands)
6. [Response Codes](#response-codes)

---

## Endpoints

### Sandbox Environment

| Service | URL |
|---------|-----|
| Payment Page (Hosted Checkout, Tokenization) | `https://sbcheckout.payfort.com/FortAPI/paymentPage` |
| Payment API (Server-to-Server) | `https://sbpaymentservices.payfort.com/FortAPI/paymentApi` |

### Production Environment

| Service | URL |
|---------|-----|
| Payment Page (Hosted Checkout, Tokenization) | `https://checkout.payfort.com/FortAPI/paymentPage` |
| Payment API (Server-to-Server) | `https://paymentservices.payfort.com/FortAPI/paymentApi` |

---

## Authentication

### Required Credentials

| Credential | Location | Description |
|------------|----------|-------------|
| `merchant_identifier` | Dashboard → Integration Settings | Unique merchant ID |
| `access_code` | Dashboard → Security Settings | API authentication token |
| `signature` | Calculated | SHA-256 HMAC signature |

### Signature Calculation

See `signature-calculation.md` for detailed signature generation guide.

---

## Payment Commands

### PURCHASE

Process immediate payment capture.

**Endpoint:** `POST /FortAPI/paymentApi`

**Request:**
```json
{
  "command": "PURCHASE",
  "access_code": "string",
  "merchant_identifier": "string",
  "merchant_reference": "string",
  "amount": "string",
  "currency": "string",
  "language": "string",
  "customer_email": "string",
  "signature": "string",
  "token_name": "string (optional)",
  "card_number": "string (optional, PCI only)",
  "expiry_date": "string (optional, PCI only)",
  "card_security_code": "string (optional)",
  "customer_ip": "string (optional)",
  "order_description": "string (optional)",
  "phone_number": "string (optional)",
  "remember_me": "YES|NO (optional)",
  "eci": "ECOMMERCE|RECURRING (optional)",
  "agreement_id": "string (optional, for recurring)"
}
```

**Response:**
```json
{
  "command": "PURCHASE",
  "response_code": "14000",
  "response_message": "Success",
  "status": "14",
  "fort_id": "string",
  "authorization_code": "string",
  "merchant_reference": "string",
  "amount": "string",
  "currency": "string",
  "token_name": "string (if tokenization enabled)",
  "agreement_id": "string (if recurring)",
  "3ds_url": "string (if 3DS required)"
}
```

### AUTHORIZATION

Place hold on funds without capturing.

**Request:**
```json
{
  "command": "AUTHORIZATION",
  "access_code": "string",
  "merchant_identifier": "string",
  "merchant_reference": "string",
  "amount": "string",
  "currency": "string",
  "language": "string",
  "customer_email": "string",
  "signature": "string"
}
```

**Note:** Must capture within 5-7 days.

---

## Management Commands

### CAPTURE

Capture previously authorized amount.

**Endpoint:** `POST /FortAPI/paymentApi`

**Request:**
```json
{
  "command": "CAPTURE",
  "access_code": "string",
  "merchant_identifier": "string",
  "merchant_reference": "string",
  "amount": "string",
  "currency": "string",
  "language": "string",
  "signature": "string",
  "fort_id": "string (optional)",
  "order_description": "string (optional)"
}
```

**Response:**
```json
{
  "command": "CAPTURE",
  "response_code": "04000",
  "response_message": "Success",
  "status": "04",
  "fort_id": "string",
  "merchant_reference": "string",
  "amount": "string"
}
```

**Important:**
- Must be within 5-7 days of authorization
- Partial captures supported
- Amount cannot exceed authorized amount

### REFUND

Refund captured transaction.

**Request:**
```json
{
  "command": "REFUND",
  "access_code": "string",
  "merchant_identifier": "string",
  "merchant_reference": "string",
  "amount": "string",
  "currency": "string",
  "language": "string",
  "signature": "string",
  "fort_id": "string (optional)",
  "maintenance_reference": "string (optional)",
  "order_description": "string (optional)"
}
```

**Response:**
```json
{
  "command": "REFUND",
  "response_code": "06000",
  "response_message": "Success",
  "status": "06",
  "fort_id": "string",
  "merchant_reference": "string",
  "amount": "string"
}
```

**Important:**
- Full or partial refunds supported
- Amount cannot exceed captured amount
- merchant_reference must match original transaction

### VOID_AUTH

Void an authorization before capture.

**Request:**
```json
{
  "command": "VOID_AUTHORIZATION",
  "access_code": "string",
  "merchant_identifier": "string",
  "merchant_reference": "string",
  "language": "string",
  "signature": "string",
  "fort_id": "string (optional)"
}
```

---

## Service Commands

### TOKENIZATION

Securely tokenize card details (Non-PCI integration).

**Endpoint:** `POST /FortAPI/paymentPage` (HTML Form)

**Request (Form Data):**
```html
<input type="hidden" name="service_command" value="TOKENIZATION">
<input type="hidden" name="access_code" value="string">
<input type="hidden" name="merchant_identifier" value="string">
<input type="hidden" name="merchant_reference" value="string">
<input type="hidden" name="language" value="en|ar">
<input type="hidden" name="return_url" value="string">
<input type="hidden" name="signature" value="string">
<input type="hidden" name="card_number" value="string">
<input type="hidden" name="expiry_date" value="YYMM">
<input type="hidden" name="card_security_code" value="string">
<input type="hidden" name="card_holder_name" value="string (optional)">
<input type="hidden" name="remember_me" value="YES|NO (optional)">
```

**Response:**
```json
{
  "service_command": "TOKENIZATION",
  "response_code": "18000",
  "response_message": "Success",
  "status": "18",
  "token_name": "string",
  "card_number": "455701******8902",
  "card_bin": "455701",
  "expiry_date": "2505",
  "card_holder_name": "string"
}
```

**Signature Exclusions:**
- `card_number`
- `expiry_date`
- `card_security_code`
- `card_holder_name`
- `remember_me`

### CREATE_TOKEN

Create permanent token without payment (not recommended).

**Request:**
```html
<input type="hidden" name="service_command" value="CREATE_TOKEN">
<input type="hidden" name="access_code" value="string">
<input type="hidden" name="merchant_identifier" value="string">
<input type="hidden" name="merchant_reference" value="string">
<input type="hidden" name="language" value="en|ar">
<input type="hidden" name="return_url" value="string">
<input type="hidden" name="signature" value="string">
<input type="hidden" name="card_number" value="string">
<input type="hidden" name="expiry_date" value="YYMM">
<input type="hidden" name="currency" value="string">
<input type="hidden" name="token_name" value="string (optional)">
<input type="hidden" name="card_holder_name" value="string (optional)">
```

### UPDATE_TOKEN

Manage saved card tokens.

**Endpoint:** `POST /FortAPI/paymentApi`

**Request:**
```json
{
  "service_command": "UPDATE_TOKEN",
  "access_code": "string",
  "merchant_identifier": "string",
  "merchant_reference": "string",
  "language": "string",
  "token_name": "string",
  "signature": "string",
  "token_status": "ACTIVE|INACTIVE (optional)",
  "new_token_name": "string (optional)",
  "expiry_date": "YYMM (optional)",
  "card_holder_name": "string (optional)"
}
```

### GET_INSTALLMENTS_PLANS

Retrieve available installment plans.

**Request:**
```json
{
  "access_code": "string",
  "query_command": "GET_INSTALLMENTS_PLANS",
  "merchant_identifier": "string",
  "language": "string",
  "signature": "string"
}
```

**Response:**
```json
{
  "response_code": "20000",
  "response_message": "Success",
  "service_command": "GET_INSTALLMENTS_PLANS",
  "installment_detail": {
    "issuer_detail": [
      {
        "issuer_code": "ADCB",
        "issuer_name_en": "Abu Dhabi Commercial Bank",
        "bin_details": []
      }
    ]
  }
}
```

---

## Response Codes

### Status Codes (First 2 Digits)

| Code | Description |
|------|-------------|
| 00 | Invalid Request |
| 02 | Authorization Success |
| 03 | Authorization Failed |
| 04 | Capture Success |
| 05 | Capture Failed |
| 06 | Refund Success |
| 07 | Refund Failed |
| 13 | Purchase Failure |
| 14 | Purchase Success |
| 18 | Tokenization Success |
| 19 | Transaction Pending |
| 20 | On Hold |

### Message Codes (Last 3 Digits)

| Code | Message |
|------|---------|
| 000 | Success |
| 001 | Missing parameter |
| 002 | Invalid parameter format |
| 008 | Signature mismatch |
| 009 | Invalid merchant identifier |
| 010 | Invalid access code |
| 029 | Insufficient funds |
| 030 | Authentication failed |
| 048 | Operation amount exceeds captured amount |
| 049 | Operation not valid for this payment option |
| 064 | 3D Secure check requested |

See `error-codes.md` for complete error code reference.

---

## Parameter Specifications

### Special Characters Allowed

| Parameter | Allowed Characters |
|-----------|-------------------|
| `merchant_reference` | `- _ .` |
| `token_name` | `. @ - _` |
| `card_holder_name` | `' - .` and spaces |
| `return_url` | `$ ! = ? # & - _ / : .` |
| `customer_email` | `_ - . @ +` |
| `customer_ip` | `. :` |
| `order_description` | `' / . _ - # : $` and spaces |
| `phone_number` | `+ - ( )` and spaces |

### Amount Formatting

Multiply by currency decimal factor:

| Currency | Factor | Example |
|----------|--------|---------|
| AED, USD, EUR (2 decimals) | ×100 | 500 → 50000 |
| KWD, BHD, OMR (3 decimals) | ×1000 | 500 → 500000 |

**VISA Note:** For 3-decimal currencies, round to zero in final decimal.

### Date Formats

| Field | Format | Example |
|-------|--------|---------|
| `expiry_date` | YYMM | 2505 (May 2025) |
| `recurring_expiry_date` | YYYY-MM-DD | 2025-12-31 |

---

## Best Practices

1. **Always use HTTPS** for all API communications
2. **Store credentials securely** as environment variables
3. **Validate signatures** on all responses
4. **Use unique merchant_reference** for each transaction
5. **Implement idempotency** using maintenance_reference for retries
6. **Log all transactions** with fort_id for reconciliation
7. **Monitor webhook deliveries** and implement retry logic
8. **Test thoroughly in sandbox** before going live
