# Signature Calculation Guide

Complete guide for calculating SHA signatures for Amazon Payment Services API requests and responses.

---

## Overview

The signature is a critical security parameter that ensures:
- **Authenticity:** Request/response is from legitimate source
- **Integrity:** Data has not been tampered with
- **Non-repudiation:** Sender cannot deny sending the request

---

## Configuration

### Required Settings

Obtain from Merchant Dashboard → Integration Settings → Security Settings:

| Parameter | Description | Example |
|-----------|-------------|---------|
| **SHA Type** | Hash algorithm (SHA-256 recommended) | `SHA-256` |
| **SHA Request Phrase** | Secret phrase for request signatures | `MySecretKey123` |
| **SHA Response Phrase** | Secret phrase for response validation | `MySecretKey123` |

**Security Note:** Never expose SHA phrases in client-side code or public repositories. Store as environment variables.

---

## Request Signature Generation

### 5-Step Process

#### Step 1: Parameter Collection

Collect all request parameters that will be sent to APS.

**Example Payment Request:**
```json
{
  "command": "PURCHASE",
  "access_code": "SILgpo7pWbmzuURp2qri",
  "merchant_identifier": "MxvOupuG",
  "merchant_reference": "ORD-12345-2024",
  "amount": "2000",
  "currency": "AED",
  "language": "en",
  "customer_email": "customer@example.com"
}
```

#### Step 2: Alphabetical Sorting

Sort all parameters alphabetically by parameter name (case-sensitive).

**Sorted Parameters:**
```
access_code = SILgpo7pWbmzuURp2qri
amount = 2000
command = PURCHASE
currency = AED
customer_email = customer@example.com
language = en
merchant_identifier = MxvOupuG
merchant_reference = ORD-12345-2024
```

#### Step 3: String Concatenation

Concatenate as `param_name=param_value` **without any separators** between pairs.

**Concatenated String:**
```
access_code=SILgpo7pWbmzuURp2qriamount=2000command=PURCHASEcurrency=AEDcustomer_email=customer@example.comlanguage=enmerchant_identifier=MxvOupuGmerchant_reference=ORD-12345-2024
```

#### Step 4: Phrase Wrapping

Wrap with SHA Request Phrase at beginning and end.

**Wrapped String:**
```
MySecretKey123access_code=SILgpo7pWbmzuURp2qriamount=2000command=PURCHASEcurrency=AEDcustomer_email=customer@example.comlanguage=enmerchant_identifier=MxvOupuGmerchant_reference=ORD-12345-2024MySecretKey123
```

#### Step 5: Hash Generation

Apply SHA-256 algorithm to generate final signature.

**Final Signature:**
```
7b8c9d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e
```

---

## Special Cases

### Tokenization Requests

**Exclude these sensitive parameters from signature calculation:**

- `card_number`
- `expiry_date`
- `card_security_code`
- `card_holder_name`
- `remember_me`

**Example Tokenization Request:**
```json
{
  "service_command": "TOKENIZATION",
  "access_code": "SILgpo7pWbmzuURp2qri",
  "merchant_identifier": "MxvOupuG",
  "merchant_reference": "TOKEN-123",
  "language": "en",
  "return_url": "https://yoursite.com/callback",
  "card_number": "4557012345678902",  // EXCLUDED
  "expiry_date": "2505",              // EXCLUDED
  "card_security_code": "123",        // EXCLUDED
  "card_holder_name": "John Doe",     // EXCLUDED
  "remember_me": "YES"                // EXCLUDED
}
```

**Parameters to include in signature:**
```
access_code, language, merchant_identifier, merchant_reference, return_url, service_command
```

### Empty and Null Values

| Case | Handling |
|------|----------|
| **Empty string** (`""`) | Include as `param_name=` |
| **Null/undefined** | Exclude from signature entirely |

---

## Response Signature Validation

### Validation Process

#### Step 1: Receive Response

```json
{
  "command": "PURCHASE",
  "access_code": "SILgpo7pWbmzuURp2qri",
  "merchant_identifier": "MxvOupuG",
  "merchant_reference": "ORD-12345-2024",
  "amount": "2000",
  "currency": "AED",
  "fort_id": "149295435400084008",
  "response_message": "Success",
  "response_code": "14000",
  "status": "14",
  "signature": "7B8C9D2E3F4A5B6C7D8E9F0A1B2C3D4E5F6A7B8C9D0E1F2A3B4C5D6E7F8A9B0C1D2E"
}
```

#### Step 2: Extract and Store Signature

Save the received signature value, then remove it from parameters before validation.

#### Step 3: Calculate Expected Signature

Use **SHA Response Phrase** and follow the same 5-step process as request signature.

#### Step 4: Compare Signatures

```python
if calculated_signature == received_signature:
    # Valid response
else:
    # Invalid - possible tampering
```

---

## Implementation Examples

### Python

```python
import hashlib

def calculate_signature(params, sha_phrase):
    """
    Calculate SHA-256 signature for APS API request.
    
    Args:
        params: Dictionary of request parameters
        sha_phrase: SHA Request/Response phrase
    
    Returns:
        Hex-encoded SHA-256 signature string
    """
    # Parameters to exclude (for tokenization)
    exclude_params = {
        'card_number',
        'expiry_date',
        'card_security_code',
        'card_holder_name',
        'remember_me'
    }
    
    # Filter out excluded and None values
    filtered_params = {
        k: v for k, v in params.items() 
        if k not in exclude_params and v is not None
    }
    
    # Sort parameters alphabetically
    sorted_params = sorted(filtered_params.items())
    
    # Build signature string
    signature_string = sha_phrase
    for key, value in sorted_params:
        signature_string += f'{key}={value}'
    signature_string += sha_phrase
    
    # Generate SHA-256 hash
    signature = hashlib.sha256(signature_string.encode('utf-8')).hexdigest()
    
    return signature

# Example usage
params = {
    'command': 'PURCHASE',
    'access_code': 'SILgpo7pWbmzuURp2qri',
    'merchant_identifier': 'MxvOupuG',
    'merchant_reference': 'ORD-12345-2024',
    'amount': '2000',
    'currency': 'AED',
    'language': 'en',
    'customer_email': 'customer@example.com'
}

signature = calculate_signature(params, 'MySecretKey123')
print(f'Signature: {signature}')
```

### PHP

```php
<?php
function calculateSignature($params, $shaPhrase) {
    /**
     * Calculate SHA-256 signature for APS API request
     *
     * @param array $params Request parameters
     * @param string $shaPhrase SHA Request/Response phrase
     * @return string Hex-encoded signature
     */
    
    // Parameters to exclude (for tokenization)
    $excludeParams = [
        'card_number',
        'expiry_date',
        'card_security_code',
        'card_holder_name',
        'remember_me'
    ];
    
    // Filter out excluded and null values
    $filteredParams = array_filter($params, function($key) use ($excludeParams) {
        return !in_array($key, $excludeParams) && $params[$key] !== null;
    }, ARRAY_FILTER_USE_KEY);
    
    // Sort parameters alphabetically by key
    ksort($filteredParams);
    
    // Build signature string
    $signatureString = $shaPhrase;
    foreach ($filteredParams as $key => $value) {
        $signatureString .= $key . '=' . $value;
    }
    $signatureString .= $shaPhrase;
    
    // Generate SHA-256 hash
    $signature = hash('sha256', $signatureString);
    
    return $signature;
}

// Example usage
$params = [
    'command' => 'PURCHASE',
    'access_code' => 'SILgpo7pWbmzuURp2qri',
    'merchant_identifier' => 'MxvOupuG',
    'merchant_reference' => 'ORD-12345-2024',
    'amount' => '2000',
    'currency' => 'AED',
    'language' => 'en',
    'customer_email' => 'customer@example.com'
];

$signature = calculateSignature($params, 'MySecretKey123');
echo "Signature: " . $signature;
?>
```

### Java

```java
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.*;

public class SignatureCalculator {
    
    /**
     * Calculate SHA-256 signature for APS API request
     * 
     * @param params Request parameters
     * @param shaPhrase SHA Request/Response phrase
     * @return Hex-encoded signature string
     * @throws NoSuchAlgorithmException if SHA-256 not available
     */
    public static String calculateSignature(
        Map<String, String> params, 
        String shaPhrase
    ) throws NoSuchAlgorithmException {
        
        // Parameters to exclude (for tokenization)
        Set<String> excludeParams = new HashSet<>(Arrays.asList(
            "card_number",
            "expiry_date",
            "card_security_code",
            "card_holder_name",
            "remember_me"
        ));
        
        // Filter out excluded and null values
        Map<String, String> filteredParams = new HashMap<>();
        for (Map.Entry<String, String> entry : params.entrySet()) {
            if (!excludeParams.contains(entry.getKey()) 
                && entry.getValue() != null) {
                filteredParams.put(entry.getKey(), entry.getValue());
            }
        }
        
        // Sort parameters alphabetically by key
        List<String> sortedKeys = new ArrayList<>(filteredParams.keySet());
        Collections.sort(sortedKeys);
        
        // Build signature string
        StringBuilder signatureString = new StringBuilder(shaPhrase);
        for (String key : sortedKeys) {
            signatureString.append(key)
                          .append("=")
                          .append(filteredParams.get(key));
        }
        signatureString.append(shaPhrase);
        
        // Generate SHA-256 hash
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        byte[] hashBytes = digest.digest(
            signatureString.toString().getBytes()
        );
        
        // Convert to hex string
        return bytesToHex(hashBytes);
    }
    
    private static String bytesToHex(byte[] bytes) {
        StringBuilder hexString = new StringBuilder();
        for (byte b : bytes) {
            hexString.append(String.format("%02x", b));
        }
        return hexString.toString();
    }
    
    // Example usage
    public static void main(String[] args) throws Exception {
        Map<String, String> params = new HashMap<>();
        params.put("command", "PURCHASE");
        params.put("access_code", "SILgpo7pWbmzuURp2qri");
        params.put("merchant_identifier", "MxvOupuG");
        params.put("merchant_reference", "ORD-12345-2024");
        params.put("amount", "2000");
        params.put("currency", "AED");
        params.put("language", "en");
        params.put("customer_email", "customer@example.com");
        
        String signature = calculateSignature(params, "MySecretKey123");
        System.out.println("Signature: " + signature);
    }
}
```

### Node.js

```javascript
const crypto = require('crypto');

/**
 * Calculate SHA-256 signature for APS API request
 * 
 * @param {Object} params - Request parameters
 * @param {string} shaPhrase - SHA Request/Response phrase
 * @returns {string} Hex-encoded signature
 */
function calculateSignature(params, shaPhrase) {
    // Parameters to exclude (for tokenization)
    const excludeParams = new Set([
        'card_number',
        'expiry_date',
        'card_security_code',
        'card_holder_name',
        'remember_me'
    ]);
    
    // Filter out excluded and null/undefined values
    const filteredParams = {};
    for (const [key, value] of Object.entries(params)) {
        if (!excludeParams.has(key) && value != null) {
            filteredParams[key] = value;
        }
    }
    
    // Sort parameters alphabetically by key
    const sortedKeys = Object.keys(filteredParams).sort();
    
    // Build signature string
    let signatureString = shaPhrase;
    for (const key of sortedKeys) {
        signatureString += `${key}=${filteredParams[key]}`;
    }
    signatureString += shaPhrase;
    
    // Generate SHA-256 hash
    const hash = crypto.createHash('sha256');
    hash.update(signatureString);
    const signature = hash.digest('hex');
    
    return signature;
}

// Example usage
const params = {
    command: 'PURCHASE',
    access_code: 'SILgpo7pWbmzuURp2qri',
    merchant_identifier: 'MxvOupuG',
    merchant_reference: 'ORD-12345-2024',
    amount: '2000',
    currency: 'AED',
    language: 'en',
    customer_email: 'customer@example.com'
};

const signature = calculateSignature(params, 'MySecretKey123');
console.log(`Signature: ${signature}`);

module.exports = { calculateSignature };
```

---

## Common Issues and Solutions

### Issue: Signature Mismatch (Error 00008)

**Possible Causes:**

1. **Incorrect SHA phrase**
   - Solution: Verify SHA Request Phrase in dashboard matches your code

2. **Wrong parameter ordering**
   - Solution: Ensure alphabetical sorting (case-sensitive)

3. **Including excluded parameters**
   - Solution: Remove card details from tokenization signature

4. **Character encoding issues**
   - Solution: Use UTF-8 encoding consistently

5. **Empty vs null handling**
   - Solution: Include empty strings, exclude null values

### Issue: Response Signature Validation Fails

**Possible Causes:**

1. **Using wrong SHA phrase**
   - Solution: Use SHA Response Phrase (may differ from Request Phrase)

2. **Response modified in transit**
   - Solution: Check for man-in-the-middle, use HTTPS only

3. **Different SHA algorithm**
   - Solution: Verify SHA Type in dashboard (SHA-256, SHA-512, etc.)

---

## Testing Your Implementation

### Test Case 1: Basic Payment Request

```python
params = {
    'command': 'PURCHASE',
    'access_code': 'test123',
    'merchant_identifier': 'TEST_MERCHANT',
    'merchant_reference': 'TEST-001',
    'amount': '1000',
    'currency': 'AED',
    'language': 'en'
}
sha_phrase = 'TestPhrase123'

# Expected: Consistent signature every time
signature = calculate_signature(params, sha_phrase)
```

### Test Case 2: Tokenization Request

```python
params = {
    'service_command': 'TOKENIZATION',
    'access_code': 'test123',
    'merchant_identifier': 'TEST_MERCHANT',
    'merchant_reference': 'TOKEN-001',
    'language': 'en',
    'return_url': 'https://yoursite.com/callback',
    # These should be EXCLUDED:
    'card_number': '4557012345678902',
    'expiry_date': '2505',
    'card_security_code': '123'
}
sha_phrase = 'TestPhrase123'

# Signature should NOT include card details
signature = calculate_signature(params, sha_phrase)
```

---

## Best Practices

1. **Use environment variables** for SHA phrases
2. **Validate all response signatures** before processing
3. **Log signature failures** for security monitoring
4. **Use HTTPS only** for all API communications
5. **Rotate SHA phrases periodically** for enhanced security
6. **Test signature calculation** thoroughly before going live
7. **Handle encoding consistently** (UTF-8 recommended)
8. **Never log SHA phrases** or full signature strings

---

## Support

**Technical Support:** merchantsupport-ps@amazon.com
