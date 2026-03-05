---
name: amazon-payment-services
description: Amazon Payment Services (APS) integration and API usage. Use this skill whenever the user mentions payment integration, accepting payments, payment APIs, checkout flows, payment methods (cards, Apple Pay, KNET, NAPS, STC Pay, MADA, etc.), recurring payments, refunds, captures, 3D Secure, PCI compliance, or any Amazon Payment Services documentation questions. This skill covers all APS integration types: Hosted Checkout, Custom Integration (PCI and Non-PCI), Mobile SDK, Payment Links, and e-commerce plugins.
---

# Amazon Payment Services Integration Skill

This skill provides comprehensive assistance with Amazon Payment Services (APS) integration, API usage, and best practices for accepting and managing payments.

## When to Use This Skill

Use this skill whenever the user needs help with:
- Setting up Amazon Payment Services integration
- Understanding payment APIs and endpoints
- Implementing payment flows (authorization, capture, refund, void)
- Working with payment methods (cards, digital wallets, local payment methods, BNPL)
- Configuring security features (3D Secure, PCI compliance, signature calculation)
- Managing recurring payments and subscriptions
- Using services like Save Card, Installments, Currency Conversion, Network Tokenization
- Debugging payment errors and response codes
- Going live with payment integration

## Core Integration Types

### 1. Hosted Checkout
Redirects customers to Amazon Payment Services' secure payment page.

**Best for:** Quick setup, minimal development, reduced PCI scope

**API Endpoint:**
- Sandbox: `https://sbcheckout.payfort.com/FortAPI/paymentPage`
- Production: `https://checkout.payfort.com/FortAPI/paymentPage`

**Key Features:**
- Multi-payment options (cards, wallets, local methods)
- Save Card feature
- 3D Secure authentication
- Customizable themes
- Multi-language support (Arabic/English)

### 2. Custom Integration
Build fully customized payment forms on your website.

**Two approaches:**

#### Non-PCI Custom Integration (Recommended)
- Card data never touches your server
- Uses secure tokenization
- Reduced PCI compliance requirements

**Flow:**
1. Create payment form (client-side)
2. Submit card data directly to APS for tokenization
3. Receive token and process payment server-to-server
4. Handle 3D Secure if required

**Tokenization Endpoint:**
- Sandbox: `https://sbcheckout.payfort.com/FortAPI/paymentPage`
- Payment API: `https://sbpaymentservices.payfort.com/FortAPI/paymentApi`

#### PCI-Certified Custom Integration
- Direct server-to-server payment processing
- Requires PCI DSS Level 1 compliance
- Full control over card data

### 3. Mobile Integration
Native SDKs for iOS, Android, React Native, and Flutter.

**Features:**
- In-app payment processing
- No browser redirects
- Apple Pay support
- SDK token-based authentication

### 4. Payment Links
No-code solution for generating shareable payment links.

**Best for:**
- Merchants without websites
- Invoice-based billing
- Quick payment requests

### 5. E-commerce Plugins
Pre-built plugins for popular platforms (WooCommerce, Shopify, Magento, etc.)

## Payment Methods

### Cards
- Visa, Mastercard, American Express (Global)
- MADA (Saudi Arabia)
- Meeza (Egypt)

### Digital Wallets
- Apple Pay (60+ countries)

### Local Payment Methods
| Method | Country | Currency |
|--------|---------|----------|
| KNET | Kuwait | KWD |
| NAPS | Qatar | QAR |
| OmanNet | Oman | OMR |
| BENEFIT | Bahrain | BHD |
| STC Pay | Saudi Arabia | SAR |

### BNPL (Buy Now Pay Later)
| Provider | Regions | Integration Types |
|----------|---------|-------------------|
| Tabby | UAE, KSA | Hosted Checkout, Payment Links |
| Tamara | UAE, KSA | Hosted Checkout |
| valU | Egypt | Custom Integration |

## API Reference

### Environment URLs

| Environment | Payment Page | Payment API |
|-------------|--------------|-------------|
| Sandbox | `https://sbcheckout.payfort.com/FortAPI/paymentPage` | `https://sbpaymentservices.payfort.com/FortAPI/paymentApi` |
| Production | `https://checkout.payfort.com/FortAPI/paymentPage` | `https://paymentservices.payfort.com/FortAPI/paymentApi` |

### Common Commands

#### PURCHASE
Capture payment immediately.

```json
{
  "command": "PURCHASE",
  "access_code": "your_access_code",
  "merchant_identifier": "your_merchant_id",
  "merchant_reference": "ORDER_123",
  "amount": "10000",
  "currency": "AED",
  "language": "en",
  "customer_email": "customer@example.com",
  "token_name": "token_from_tokenization",
  "signature": "calculated_signature"
}
```

#### AUTHORIZATION
Place hold on funds, capture later.

```json
{
  "command": "AUTHORIZATION",
  "access_code": "your_access_code",
  "merchant_identifier": "your_merchant_id",
  "merchant_reference": "ORDER_123",
  "amount": "10000",
  "currency": "AED",
  "language": "en",
  "customer_email": "customer@example.com",
  "signature": "calculated_signature"
}
```

#### CAPTURE
Capture previously authorized amount.

```json
{
  "command": "CAPTURE",
  "access_code": "your_access_code",
  "merchant_identifier": "your_merchant_id",
  "merchant_reference": "ORDER_123",
  "amount": "10000",
  "currency": "AED",
  "language": "en",
  "signature": "calculated_signature"
}
```

**Note:** Must be done within 5-7 days of authorization.

#### REFUND
Refund captured transaction (full or partial).

```json
{
  "command": "REFUND",
  "access_code": "your_access_code",
  "merchant_identifier": "your_merchant_id",
  "merchant_reference": "ORDER_123",
  "amount": "5000",
  "currency": "AED",
  "language": "en",
  "signature": "calculated_signature"
}
```

### Recurring Payments

For recurring/subscription payments:

```json
{
  "command": "PURCHASE",
  "access_code": "your_access_code",
  "merchant_identifier": "your_merchant_id",
  "merchant_reference": "REC_UNIQUE_123",
  "amount": "11050",
  "currency": "AED",
  "language": "en",
  "customer_email": "customer@example.com",
  "eci": "RECURRING",
  "token_name": "stored_token",
  "agreement_id": "agreement_from_initial_payment",
  "signature": "calculated_signature"
}
```

**Required from initial payment:**
- `token_name`: Secure token for customer's card
- `agreement_id`: Agreement identifier

### Save Card Service

Enable customers to save cards for future use:

**Tokenization with remember_me:**
```json
{
  "service_command": "TOKENIZATION",
  "access_code": "your_access_code",
  "merchant_identifier": "your_merchant_id",
  "merchant_reference": "TOKEN_123",
  "language": "en",
  "card_number": "4557012345678902",
  "expiry_date": "2505",
  "card_security_code": "123",
  "remember_me": "YES",
  "return_url": "https://yoursite.com/callback",
  "signature": "calculated_signature"
}
```

**Using saved card:**
```json
{
  "command": "PURCHASE",
  "access_code": "your_access_code",
  "merchant_identifier": "your_merchant_id",
  "merchant_reference": "ORDER_456",
  "amount": "5000",
  "currency": "AED",
  "token_name": "saved_token",
  "card_security_code": "123",
  "signature": "calculated_signature"
}
```

### Installments

**Step 1: Get Installment Plans**
```json
{
  "access_code": "your_access_code",
  "query_command": "GET_INSTALLMENTS_PLANS",
  "merchant_identifier": "your_merchant_id",
  "language": "en",
  "signature": "calculated_signature"
}
```

**Step 2: Process with Installments**
```json
{
  "command": "PURCHASE",
  "access_code": "your_access_code",
  "merchant_identifier": "your_merchant_id",
  "merchant_reference": "ORDER_789",
  "amount": "10000",
  "currency": "AED",
  "installments": "HOSTED",
  "issuer_code": "ADCB",
  "plan_code": "PLAN_6M",
  "signature": "calculated_signature"
}
```

**Note:** Only PURCHASE command supported (not AUTHORIZATION).

## Security

### Signature Calculation

**5-Step Process:**

1. **Collect Parameters:** Gather all request parameters
2. **Sort Alphabetically:** Sort by parameter name (case-sensitive)
3. **Concatenate:** Join as `key=value` without separators
4. **Wrap with Phrase:** Add SHA Request Phrase at start and end
5. **Generate Hash:** Apply SHA-256

**Example (Python):**
```python
import hashlib

def calculate_signature(params, sha_phrase):
    # Sort parameters alphabetically
    sorted_params = sorted(params.items())
    
    # Build signature string
    signature_string = sha_phrase
    for key, value in sorted_params:
        signature_string += f'{key}={value}'
    signature_string += sha_phrase
    
    # Generate SHA-256 hash
    return hashlib.sha256(signature_string.encode('utf-8')).hexdigest()
```

**Excluded from Tokenization:**
- `card_number`
- `expiry_date`
- `card_security_code`
- `card_holder_name`
- `remember_me`

### 3D Secure

**Standard 3D Secure:**
- Enabled by default
- Automatically detects enrolled cards
- Redirects for authentication when required

**Downgrading 3DS:**
Add `check_3ds: "NO"` to payment request (requires bank approval).

**Note:** Not supported for MADA and Meeza cards (regulatory requirement).

### PCI Compliance

**Non-PCI Integration (Recommended):**
- Use Hosted Checkout or Custom Integration with tokenization
- Card data never touches your server
- No PCI audit required

**PCI-Certified Integration:**
- Requires PCI DSS Level 1 compliance
- Direct card data handling
- Full control over payment flow

## Important Technical Notes

### Currency Amount Formatting

**Multiply by currency decimal factor:**

| Currency | Decimals | Multiplier | Example |
|----------|----------|------------|---------|
| AED | 2 | ×100 | 500 AED → `50000` |
| USD | 2 | ×100 | 500 USD → `50000` |
| KWD | 3 | ×1000 | 500 KWD → `500000` |

**VISA Note:** For 3-decimal currencies, round to zero in final decimal place.

### Merchant Reference

**Must be unique per transaction.** Allowed characters: alphanumeric, `-`, `_`, `.`

### Response Code Structure

5-digit response codes:
- **First 2 digits:** Status (operation result)
- **Last 3 digits:** Message (specific details)

**Common Status Codes:**
- `14`: Purchase Success
- `04`: Capture Success
- `06`: Refund Success
- `02`: Authorization Success
- `03`: Authorization Failed
- `13`: Purchase Failure

**Common Message Codes:**
- `000`: Success
- `001`: Missing parameter
- `002`: Invalid parameter format
- `008`: Signature mismatch
- `029`: Insufficient funds
- `064`: 3D Secure check requested

### Local Payment Methods

**Multi-Select Feature:**
- Omit `payment_option` to show all available methods
- Currency determines displayed methods:
  - SAR → STC Pay
  - KWD → KNET
  - QAR → NAPS
  - OMR → OmanNet
  - BHD → BENEFIT

**Critical:** Use `PURCHASE` command (not `AUTHORIZATION`) for local payment methods.

## Error Handling

### Common Errors

| Code | Message | Resolution |
|------|---------|------------|
| 00008 | Signature mismatch | Verify signature calculation |
| 00009 | Invalid merchant identifier | Check merchant ID |
| 00010 | Invalid access code | Verify access code |
| 03029 | Insufficient funds | Customer needs different payment method |
| 03012 | Card expired | Customer needs to update card |
| 00048 | Operation amount exceeds captured amount | Refund amount cannot exceed captured amount |
| 00049 | Operation not valid for this payment option | Use PURCHASE for local payment methods |

### Retry Attempts

| Payment Method | Max Retries |
|----------------|-------------|
| Visa, Mastercard, Amex | 9 |
| MADA, Meeza | 9 |
| KNET, NAPS, STC Pay, Tabby, Tamara, valU, BENEFIT, OmanNet | 1 |

## Getting Started Checklist

1. **Obtain Credentials:**
   - Access Code
   - Merchant Identifier
   - SHA Request Phrase
   - SHA Response Phrase

2. **Choose Integration Type:**
   - Hosted Checkout (fastest)
   - Custom Integration (most control)
   - Mobile SDK (native apps)
   - Payment Links (no-code)

3. **Test in Sandbox:**
   - Use sandbox endpoints
   - Test with provided test cards
   - Verify signature calculation
   - Test all payment flows

4. **Configure Webhooks:**
   - Set up HTTPS endpoint
   - Configure in merchant dashboard
   - Implement signature validation

5. **Go Live:**
   - Switch to production credentials
   - Update endpoints to production URLs
   - Monitor first transactions closely

## Support

**Technical Support:** merchantsupport-ps@amazon.com

**Integration Assistance:** integration-ps@amazon.com

## Related Resources

- **API Reference:** Complete parameter specifications
- **Test Cards:** Testing card numbers for all scenarios
- **Error Codes:** Full error code reference
- **Go-Live Checklist:** Production deployment guide
- **Signature Calculation:** Detailed signature guide
