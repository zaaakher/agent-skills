---
name: amazon-payment-services-integration
description: >-
  Complete guide to integrating Amazon Payment Services (APS) into Next.js
  applications using the amazon-payment-services npm package. Use this skill
  whenever users need to accept payments in Middle East markets (Saudi Arabia,
  UAE, Kuwait, Qatar, Bahrain, Oman, Egypt, Jordan), implement payment links,
  hosted checkout, custom payment forms, card tokenization, or handle APS
  webhooks. This skill covers both sandbox testing and production deployment.
metadata:
  tags: []
  category: general
  version: 1.0.0
  public: false
  created: '2026-03-06'
  updated: '2026-03-06'
---

# Amazon Payment Services Integration Skill

This skill enables complete Amazon Payment Services (APS) integration into Next.js projects using the `amazon-payment-services` npm package. APS is the leading payment processor for Middle East markets, supporting MADA (Saudi Arabia), KNET (Kuwait), NAPS (Qatar), Fawry (Egypt), Apple Pay, and all major credit cards.

## When to Use This Skill

Use this skill when:
- User wants to accept payments in Saudi Arabia, UAE, Kuwait, Qatar, Bahrain, Oman, Egypt, or Jordan
- User needs payment links, hosted checkout, or custom payment forms
- User needs to tokenize cards for recurring payments
- User needs to handle webhooks from APS
- User is migrating from PayFort to Amazon Payment Services
- User mentions "PayFort", "Amazon Payment Services", or "APS"

## Installation

```bash
# Install the package
npm install amazon-payment-services

# Or with pnpm
pnpm add amazon-payment-services
```

## Configuration

### 1. Create `.env.local` file

```bash
# APS Merchant Credentials (from APS dashboard)
APS_MERCHANT_ID=your_merchant_id
APS_ACCESS_CODE=your_access_code
APS_REQUEST_SECRET=your_request_secret
APS_RESPONSE_SECRET=your_response_secret

# APS Configuration
APS_ENVIRONMENT=sandbox  # Use 'production' for live
APS_CURRENCY=SAR         # Default currency (SAR, AED, KWD, etc.)
APS_LANGUAGE=en          # 'en' or 'ar'

# Your App URL
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### 2. Create APS Client Helper

Create `app/api/aps-client.ts`:

```typescript
import APS from 'amazon-payment-services';

export function getAPSClient() {
  return new APS({
    merchantId: process.env.APS_MERCHANT_ID || '',
    accessCode: process.env.APS_ACCESS_CODE || '',
    requestSecret: process.env.APS_REQUEST_SECRET || '',
    responseSecret: process.env.APS_RESPONSE_SECRET || '',
    environment: (process.env.APS_ENVIRONMENT as 'sandbox' | 'production') || 'sandbox',
    currency: process.env.APS_CURRENCY || 'USD',
    language: 'en'
  });
}
```

## Payment Methods

### 1. Payment Links (Simplest)

Generate shareable payment URLs via email, SMS, WhatsApp, etc.

**API Route** (`app/api/payment/payment-link/route.ts`):

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getAPSClient } from '../../aps-client';

export async function POST(request: NextRequest) {
  const body = await request.json();
  const aps = getAPSClient();
  
  const link = await aps.paymentLinks.create({
    order: {
      id: body.orderId || `order_${Date.now()}`,
      amount: Math.round(body.amount * 100), // Convert to fils/cents
      currency: body.currency || 'SAR',
      description: body.description,
      customer: {
        email: body.customerEmail || 'customer@example.com',
        name: body.customerName,
        phone: body.customerPhone
      }
    },
    tokenValidityHours: 24
  });

  return NextResponse.json({
    success: true,
    data: {
      url: link.url,
      linkId: link.linkId,
      orderId: link.orderId
    }
  });
}
```

**Usage from frontend:**

```typescript
const response = await fetch('/api/payment/payment-link', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    amount: 100, // 100.00 SAR
    currency: 'SAR',
    description: 'Product Purchase',
    customerEmail: 'customer@example.com'
  })
});

const { data } = await response.json();
window.open(data.url, '_blank'); // Open payment link
```

### 2. Hosted Checkout (Redirect)

Redirect customers to APS-hosted payment page.

**API Route** (`app/api/payment/hosted-checkout/route.ts`):

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getAPSClient } from '../../aps-client';

export async function POST(request: NextRequest) {
  const body = await request.json();
  const aps = getAPSClient();
  
  const checkout = await aps.hostedCheckout.create({
    order: {
      id: body.orderId || `order_${Date.now()}`,
      amount: Math.round(body.amount * 100),
      currency: body.currency || 'SAR',
      description: body.description,
      customer: {
        email: body.customerEmail || 'customer@example.com',
        name: body.customerName
      }
    },
    returnUrl: body.returnUrl || `${process.env.NEXT_PUBLIC_APP_URL}/payment/result`,
    tokenize: true, // Allow card saving
    allowedPaymentMethods: ['card', 'mada', 'apple_pay']
  });

  return NextResponse.json({
    success: true,
    data: {
      transactionId: checkout.transactionId,
      orderId: checkout.orderId,
      redirectForm: checkout.redirectForm // Contains URL and params for POST
    }
  });
}
```

**Frontend redirect:**

```typescript
const { data } = await fetch('/api/payment/hosted-checkout', {
  method: 'POST',
  body: JSON.stringify({ amount: 100, currency: 'SAR' })
}).then(r => r.json());

// Create and submit form to redirect
const form = document.createElement('form');
form.method = data.redirectForm.method;
form.action = data.redirectForm.url;
Object.entries(data.redirectForm.params).forEach(([key, value]) => {
  const input = document.createElement('input');
  input.type = 'hidden';
  input.name = key;
  input.value = value;
  form.appendChild(input);
});
document.body.appendChild(form);
form.submit();
```

### 3. Custom Payment Page (Non-PCI, Custom UI)

Create your own payment form while APS handles PCI compliance through tokenization.

**Step 1: Get Tokenization Form** (`app/api/payment/custom/tokenize/route.ts`):

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getAPSClient } from '../../../aps-client';

export async function POST(request: NextRequest) {
  const body = await request.json();
  const aps = getAPSClient();
  
  const form = aps.customPaymentPage.getTokenizationForm({
    returnUrl: `${process.env.NEXT_PUBLIC_APP_URL}/api/payment/token-result`,
    merchantReference: body.merchantReference || `token_${Date.now()}`
  });

  return NextResponse.json({
    success: true,
    data: { form }
  });
}
```

**Step 2: Custom Payment Form Component** (`components/custom-payment-form.tsx`):

```typescript
'use client';

import { useState } from 'react';

export default function CustomPaymentForm({ actionUrl, params }) {
  const [loading, setLoading] = useState(false);
  const [cardNumber, setCardNumber] = useState('');
  const [expiryDate, setExpiryDate] = useState('');
  const [cvv, setCvv] = useState('');
  const [cardHolderName, setCardHolderName] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);

    const form = document.createElement('form');
    form.method = 'POST';
    form.action = actionUrl;

    // Add hidden params
    Object.entries(params).forEach(([key, value]) => {
      const input = document.createElement('input');
      input.type = 'hidden';
      input.name = key;
      input.value = value;
      form.appendChild(input);
    });

    // Add card details
    const addHidden = (name: string, value: string) => {
      const input = document.createElement('input');
      input.type = 'hidden';
      input.name = name;
      input.value = value;
      form.appendChild(input);
    };

    addHidden('card_number', cardNumber.replace(/\s/g, ''));
    addHidden('expiry_date', expiryDate.replace('/', ''));
    addHidden('card_security_code', cvv);
    addHidden('card_holder_name', cardHolderName);

    document.body.appendChild(form);
    form.submit();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={cardNumber}
        onChange={(e) => setCardNumber(e.target.value)}
        placeholder="Card Number"
        maxLength={19}
        required
      />
      <input
        value={expiryDate}
        onChange={(e) => setExpiryDate(e.target.value)}
        placeholder="MM/YY"
        maxLength={5}
        required
      />
      <input
        value={cvv}
        onChange={(e) => setCvv(e.target.value)}
        placeholder="CVV"
        maxLength={4}
        required
      />
      <input
        value={cardHolderName}
        onChange={(e) => setCardHolderName(e.target.value)}
        placeholder="Card Holder Name"
        required
      />
      <button type="submit" disabled={loading}>
        {loading ? 'Processing...' : 'Pay Now'}
      </button>
    </form>
  );
}
```

**Step 3: Handle Token Response** (`app/api/payment/token-result/route.ts`):

```typescript
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const formData = await request.formData();
  const params: Record<string, string> = {};
  formData.forEach((value, key) => {
    params[key] = value.toString();
  });

  // Extract token
  const tokenName = params.token_name;
  const status = params.status;
  const responseCode = params.response_code;

  // Check if successful (status: 18, response_code: 18000)
  if (tokenName && (responseCode === '18000' || status === '18')) {
    // Token received - store in database or session
    // Then redirect to success page
    return NextResponse.redirect(
      new URL(`/payment/success?token=${tokenName}`, request.url)
    );
  }

  // Failed
  return NextResponse.redirect(
    new URL(`/payment/failure?error=${params.response_message}`, request.url)
  );
}
```

**Step 4: Charge with Token** (`app/api/payment/custom/charge/route.ts`):

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getAPSClient } from '../../../aps-client';

export async function POST(request: NextRequest) {
  const body = await request.json();
  const aps = getAPSClient();

  const payment = await aps.customPaymentPage.chargeWithToken({
    order: {
      id: body.orderId || `order_${Date.now()}`,
      amount: Math.round(body.amount * 100),
      currency: body.currency || 'SAR',
      description: body.description
    },
    tokenName: body.tokenName, // Token from tokenization
    customerEmail: body.customerEmail || 'customer@example.com',
    customerName: body.customerName,
    customerPhone: body.customerPhone
  });

  return NextResponse.json({
    success: true,
    data: payment
  });
}
```

## Payment Management

### Capture Authorized Payment

```typescript
import { getAPSClient } from './aps-client';

const aps = getAPSClient();

// Full capture
await aps.payments.capture({
  transactionId: 'txn_123'
});

// Partial capture
await aps.payments.capture({
  transactionId: 'txn_123',
  amount: 5000 // 50.00 SAR
});
```

### Refund Payment

```typescript
// Full refund
await aps.payments.refund({
  transactionId: 'txn_123',
  reason: 'Customer request'
});

// Partial refund
await aps.payments.refund({
  transactionId: 'txn_123',
  amount: 5000, // 50.00 SAR
  reason: 'Partial refund'
});
```

### Void Transaction

```typescript
await aps.payments.void({
  transactionId: 'txn_123',
  reason: 'Order cancelled'
});
```

### Query Transaction Status

```typescript
const transaction = await aps.payments.query({
  transactionId: 'txn_123'
  // OR
  orderId: 'order_123'
});

console.log('Status:', transaction.status);
console.log('Amount:', transaction.amount);
```

## Webhooks

### Create Webhook Handler (`app/api/webhook/route.ts`):

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getAPSClient } from '../aps-client';

export async function POST(request: NextRequest) {
  const body = await request.json();
  const signature = request.headers.get('x-aps-signature') || '';
  
  const aps = getAPSClient();
  
  // Verify webhook signature
  const isValid = aps.verifyWebhookSignature(JSON.stringify(body), signature);
  
  if (!isValid) {
    return NextResponse.json(
      { success: false, error: 'Invalid signature' },
      { status: 401 }
    );
  }
  
  // Construct webhook event
  const event = aps.constructWebhookEvent(body);
  
  // Handle different event types
  switch (event.type) {
    case 'payment.success':
      // Update order status, send confirmation email
      console.log('Payment successful:', event.data);
      break;
    case 'payment.failed':
      // Notify customer, retry logic
      console.log('Payment failed:', event.data);
      break;
    case 'refund.success':
      // Process refund confirmation
      console.log('Refund processed:', event.data);
      break;
  }
  
  return NextResponse.json({ success: true, received: true });
}
```

### Configure Webhook URL in APS Dashboard

1. Log into APS merchant dashboard
2. Navigate to Technical Settings → Webhooks
3. Set webhook URL: `https://yoursite.com/api/webhook`
4. Save configuration

## Important Parameter Names

**CRITICAL:** APS is very specific about parameter names. Use these exact names:

✅ **Correct:**
- `merchant_identifier` (NOT `merchant_id`)
- `merchant_reference` (NOT `merchant_order_id`)
- `return_url` (NOT `success_url` or `failure_url`)
- `customer_email` (required for hosted checkout)
- `token_name` (for tokenized payments)

✅ **Correct Endpoints:**
- Payment Links API: `https://sbpaymentservices.payfort.com/FortAPI/paymentApi`
- Hosted Checkout: `https://sbcheckout.payfort.com/FortAPI/paymentPage`
- Production: Replace `sb` prefix with nothing

✅ **Signature Calculation:**
The package handles this automatically, but the algorithm is:
1. Sort all parameters alphabetically by key
2. Concatenate as `param_name=param_value`
3. Wrap with SHA Request Phrase: `phrase + concatenated + phrase`
4. Hash with SHA-256

## Supported Currencies

- **SAR** - Saudi Riyal
- **AED** - UAE Dirham
- **KWD** - Kuwaiti Dinar
- **QAR** - Qatari Riyal
- **BHD** - Bahraini Dinar
- **OMR** - Omani Rial
- **JOD** - Jordanian Dinar
- **EGP** - Egyptian Pound
- **USD** - US Dollar

## Supported Payment Methods

- **card** - Visa, Mastercard, American Express
- **mada** - MADA debit cards (Saudi Arabia)
- **apple_pay** - Apple Pay
- **stc_pay** - STC Pay (Saudi Arabia)
- **knet** - KNET (Kuwait)
- **naps** - NAPS (Qatar)
- **fawry** - Fawry (Egypt)
- **meeza** - Meeza (Egypt)

## Testing

### Test Cards (Sandbox)

| Card Number | Type | CVV | Expiry |
|-------------|------|-----|--------|
| 4111 1111 1111 1111 | Visa | 123 | 12/25 |
| 5297 4100 0000 0002 | MADA | 123 | 12/25 |
| 5100 0000 0000 0008 | Mastercard | 123 | 12/25 |

### Common Response Codes

- `18000` / status `18` - Tokenization successful
- `00000` / status `20` - Payment successful
- `48000` - Payment link created successfully
- `10030` - 3DS authentication failed (expected in sandbox)
- `00008` - Signature mismatch (check signature calculation)
- `00002` - Invalid parameter format

## Production Deployment

### Pre-Launch Checklist

1. ✅ Switch to production credentials:
   ```bash
   APS_ENVIRONMENT=production
   APS_MERCHANT_ID=your_production_merchant_id
   APS_ACCESS_CODE=your_production_access_code
   APS_REQUEST_SECRET=your_production_request_secret
   APS_RESPONSE_SECRET=your_production_response_secret
   ```

2. ✅ Configure production webhook URL in APS dashboard

3. ✅ Test with real cards in small amounts first

4. ✅ Enable HTTPS for all endpoints

5. ✅ Verify all redirect URLs use production domain

6. ✅ Test all payment methods you plan to accept

### Security Best Practices

- ✅ Never expose APS credentials in client-side code
- ✅ Always use environment variables
- ✅ Verify all webhook signatures
- ✅ Use HTTPS for all API calls
- ✅ Validate amounts server-side before creating payments
- ✅ Store tokens securely (encrypted database)
- ✅ Implement rate limiting on payment endpoints

## Troubleshooting

### "Signature mismatch" error
- Check that you're using the correct SHA Request Secret
- Verify parameter names are exact (e.g., `merchant_identifier` not `merchant_id`)
- Ensure parameters are sorted alphabetically before concatenation

### "Missing parameter: customer_email"
- `customer_email` is required for hosted checkout and payment links
- Add fallback: `customer_email: body.customerEmail || 'customer@example.com'`

### "Invalid extra parameters: response_format"
- `response_format` is not valid for hosted checkout (only for API calls)
- Remove it from hosted checkout requests

### "Method Not Allowed" (405)
- Ensure you're POSTing to the correct endpoint
- Hosted Checkout: `/FortAPI/paymentPage` (not just `/`)

### 3DS Authentication fails in sandbox
- This is expected behavior with test cards
- Production will work correctly with real cards

## Example Complete Implementation

See the test app at `/Users/zakhermasri/Desktop/coding/aps-test-app` for a complete working example with:
- All three payment methods
- Custom payment form UI
- Webhook handling
- Payment management
- Documentation pages

## Support

For APS-specific issues:
- Email: merchantsupport-ps@amazon.com
- Documentation: https://paymentservices.amazon.com/docs
