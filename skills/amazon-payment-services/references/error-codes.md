# Error Codes Reference

Complete reference for Amazon Payment Services error codes.

---

## Response Code Structure

Response codes are 5 digits:
- **First 2 digits:** Status code (operation result)
- **Last 3 digits:** Message code (specific details)

**Example:** `14000` = Status `14` (Purchase Success) + Message `000` (Success)

---

## Status Codes

| Code | Description |
|------|-------------|
| 00 | Invalid Request |
| 01 | Order Stored |
| 02 | Authorization Success |
| 03 | Authorization Failed |
| 04 | Capture Success |
| 05 | Capture Failed |
| 06 | Refund Success |
| 07 | Refund Failed |
| 08 | Authorization Voided Successfully |
| 09 | Authorization Void Failed |
| 10 | Incomplete |
| 11 | Check Status Failed |
| 12 | Check Status Success |
| 13 | Purchase Failure |
| 14 | Purchase Success |
| 15 | Uncertain Transaction |
| 17 | Tokenization Failed |
| 18 | Tokenization Success |
| 19 | Transaction Pending |
| 20 | On Hold |
| 21 | SDK Token Creation Failure |
| 22 | SDK Token Creation Success |
| 23 | Digital Wallet Service Failed |
| 24 | Digital Wallet Order Success |
| 27 | Check Card Balance Failed |
| 28 | Check Card Balance Success |
| 29 | Redemption Failed |
| 30 | Redemption Success |
| 31 | Reverse Redemption Failed |
| 32 | Reverse Redemption Success |
| 40 | Transaction In Review |
| 42 | Currency Conversion Success |
| 43 | Currency Conversion Failed |
| 44 | 3DS Success |
| 45 | 3DS Failed |
| 46 | Bill Creation Success |
| 47 | Bill Creation Failed |
| 48 | Invoice Payment Link Generation Success |
| 49 | Invoice Payment Link Generation Failed |
| 50 | Batch File Upload Success |
| 51 | Batch File Upload Failed |
| 52 | Token Created Successfully |
| 53 | Token Creation Failed |
| 54 | Get Tokens Success |
| 55 | Get Tokens Failed |
| 56 | Reporting Request Success |
| 57 | Reporting Request Failed |
| 58 | Token Updated Successfully |
| 59 | Token Updated Failed |
| 62 | Get Installment Plans Success |
| 63 | Get Installment Plans Failed |
| 66 | Delete Token Success |
| 70 | Get Batch Results Success |
| 71 | Get Batch Results Failed |
| 72 | Batch Processing Success |
| 73 | Batch Processing Failed |
| 74 | Bank Transfer Success |
| 75 | Bank Transfer Failed |
| 76 | Batch Validation Success |
| 77 | Batch Validation Failed |
| 80 | Credit Card Verification Success |
| 81 | Credit Card Verification Failed |
| 88 | OTP Generate Success |
| 89 | OTP Generate Failure |

---

## Common Message Codes

### Validation Errors (001-020)

| Code | Message | Resolution |
|------|---------|------------|
| 001 | Missing parameter | Check required parameters |
| 002 | Invalid parameter format | Verify parameter format |
| 003 | Payment option not available | Enable in merchant account |
| 004 | Invalid command | Use valid command value |
| 005 | Invalid amount | Check amount formatting |
| 006 | Technical problem | Contact support |
| 007 | Duplicate order number | Use unique merchant_reference |
| 008 | Signature mismatch | Verify signature calculation |
| 009 | Invalid merchant identifier | Check merchant ID |
| 010 | Invalid access code | Verify access code |
| 011 | Order not saved | Retry request |
| 012 | Card expired | Customer needs new card |
| 013 | Invalid currency | Use valid ISO 4217 code |
| 014 | Inactive payment option | Contact support |
| 015 | Inactive merchant account | Contact support |
| 016 | Invalid card number | Verify card number |
| 017 | Operation not allowed by acquirer | Contact acquirer |
| 018 | Operation not allowed by processor | Contact processor |
| 019 | Inactive acquirer | Contact support |
| 020 | Processor inactive | Contact support |

### Configuration Errors (021-040)

| Code | Message | Resolution |
|------|---------|------------|
| 021 | Payment option deactivated by acquirer | Contact acquirer |
| 023 | Currency not accepted by acquirer | Use different currency |
| 024 | Currency not accepted by processor | Use different currency |
| 025 | Processor integration settings missing | Contact support |
| 026 | Acquirer integration settings missing | Contact support |
| 027 | Invalid extra parameters | Remove invalid parameters |
| 029 | Insufficient funds | Customer needs different payment method |
| 030 | Authentication failed | Check 3DS configuration |
| 031 | Invalid issuer | Contact support |
| 032 | Invalid parameter length | Check parameter length limits |
| 033 | Parameter value not allowed | Use allowed value |
| 034 | Operation not allowed | Check operation permissions |
| 035 | Order created successfully | Success message |
| 036 | Order not found | Verify merchant_reference |
| 037 | Missing return URL | Add return_url parameter |
| 038 | Token service inactive | Contact support |
| 039 | No active payment option found | Enable payment options |
| 040 | Invalid transaction source | Check integration settings |

### Operation Errors (041-070)

| Code | Message | Resolution |
|------|---------|------------|
| 042 | Operation amount exceeds authorized amount | Reduce capture amount |
| 043 | Inactive Operation | Contact support |
| 044 | Token name does not exist | Verify token_name |
| 046 | Channel not configured for payment option | Configure channel |
| 047 | Order already processed | Use unique merchant_reference |
| 048 | Operation amount exceeds captured amount | Reduce refund amount |
| 049 | Operation not valid for this payment option | Use PURCHASE for local methods |
| 050 | Merchant per transaction limit exceeded | Reduce amount |
| 051 | Technical error | Contact support |
| 062 | Token has been created | Success message |
| 063 | Token has been updated | Success message |
| 064 | 3D Secure check requested | Redirect to 3ds_url |
| 065 | Transaction waiting for customer action | Wait for completion |
| 066 | Merchant reference already exists | Use unique reference |
| 067 | Dynamic Descriptor not configured | Contact support |
| 068 | SDK service inactive | Contact support |
| 069 | Mapping not found for error code | Contact support |
| 070 | device_id mismatch | Verify device_id |

### Payment Method Specific (071-110)

| Code | Message | Resolution |
|------|---------|------------|
| 071 | Failed to initiate connection | Retry request |
| 072 | Transaction cancelled by consumer | Customer cancelled |
| 073 | Invalid request format | Check request format |
| 074 | Transaction failed | Check error details |
| 075 | Transaction failed | Check error details |
| 076 | Transaction not found in OLP | Verify transaction |
| 077 | Error transaction code not found | Contact support |
| 078 | Failed fraud screen check | Contact support |
| 079 | Transaction challenged by fraud rules | Review fraud settings |
| 080 | Invalid payment option | Use valid payment_option |
| 082 | Inactive fraud service | Contact support |
| 083 | Unexpected user behavior | Review integration |
| 084 | Amount outside plan limits | Check installment limits |
| 086 | Installment plan not configured | Contact support |
| 087 | Card BIN does not match issuer | Use different card |
| 088 | Token name not created for transaction | Check tokenization |
| 089 | Failed to retrieve wallet details | Retry request |
| 090 | Transaction in review | Wait for review |
| 092 | Invalid issuer code | Use valid issuer_code |
| 093 | Service inactive | Contact support |
| 094 | Invalid Plan Code | Use valid plan_code |
| 095 | Inactive Issuer | Contact support |
| 096 | Inactive Plan | Contact support |
| 097 | Operation not allowed for service | Check service compatibility |
| 098 | Invalid or expired call_id | Generate new call_id |
| 099 | Failed to execute service | Contact support |

### Additional Errors (100+)

| Code | Message | Resolution |
|------|---------|------------|
| 100 | Invalid expiry date | Use YYMM format |
| 101 | Bill number not found | Verify bill number |
| 102 | Apple Pay order expired | Create new order |
| 103 | Duplicate subscription ID | Use unique ID |
| 104 | No plans valid for request | Check plan configuration |
| 105 | Invalid bank code | Use valid bank code |
| 106 | Inactive bank | Contact support |
| 107 | Invalid transfer_date | Use valid date |
| 110 | Contradicting parameters | Check integration guide |
| 111 | Service not applicable for payment option | Check compatibility |
| 112 | Service not applicable for operation | Check compatibility |
| 113 | Service not applicable for e-commerce indicator | Check eci value |
| 114 | Token already exists | Use different token_name |
| 115 | Expired invoice payment link | Generate new link |
| 116 | Inactive notification type | Contact support |
| 117 | Invoice payment link already processed | Link already used |
| 118 | Order bounced | Retry request |
| 119 | Request dropped | Retry request |
| 120 | Payment link terms not found | Add terms |
| 121 | Card number not verified | Verify card |
| 122 | Invalid date interval | Check date range |
| 123 | Maximum attempts exceeded | Wait and retry |
| 124 | Account successfully created | Success message |
| 125 | Invoice already paid | Invoice settled |
| 126 | Duplicate invoice ID | Use unique invoice ID |
| 127 | Merchant reference not generated | Generate reference |
| 128 | Report still pending | Wait for completion |
| 129 | Report queue full | Wait and retry |
| 134 | Search results exceed maximum | Narrow search |
| 136 | Batch file validation failed | Check file format |
| 137 | Invalid batch execution date | Use valid date |
| 138 | Batch file under validation | Wait for validation |
| 140 | Batch file under processing | Wait for processing |
| 141 | Batch reference does not exist | Verify reference |
| 142 | Invalid batch file header | Check header format |
| 144 | Invalid batch file | Check file format |
| 146 | Batch reference already exists | Use unique reference |
| 147 | Batch process request received | Processing |
| 148 | Batch file will be processed | Processing |
| 149 | Payment link request id not found | Verify request ID |
| 150 | Payment link already open | Link in use |
| 151 | 3ds_id does not exist | Verify 3ds_id |
| 152 | 3DS verification mismatch | Check 3DS details |
| 154 | Maximum upload retries reached | Contact support |
| 155 | Upload retries not configured | Configure retries |
| 181 | Minimum amount not reached | Increase amount |
| 221 | Maximum amount exceeded | Reduce amount |
| 225 | Operation not permitted | Check permissions |
| 662 | Order not confirmed | Confirm order first |
| 666 | Transaction declined | Card declined |
| 746 | Amount exceeds customer limit | Reduce amount |
| 747 | Insufficient wallet balance | Add funds |
| 748 | PARAMETER_MISMATCH | Check OTP parameters |
| 773 | Transaction closed | Transaction complete |
| 777 | Confirmation failed | Contact support |
| 778 | Session timed-out | Retry request |
| 779 | Transformation error | Contact support |
| 780 | Transaction number transformation error | Contact support |
| 781 | Message/response code transformation error | Contact support |
| 783 | Installments service inactive | Contact support |
| 784 | Transaction still processing | Wait for completion |
| 785 | Transaction blocked by fraud | Review fraud settings |
| 787 | Failed to authenticate user | Check authentication |
| 788 | Invalid bill number | Verify bill number |
| 789 | Expired bill number | Generate new bill |
| 790 | Invalid bill type code | Use valid code |
| 983 | Minimum 60s between OTP attempts | Wait 60 seconds |

---

## Retry Attempts by Payment Method

| Payment Method | Code | Max Retries |
|----------------|------|-------------|
| Visa | VISA | 9 |
| MasterCard | MASTERCARD | 9 |
| American Express | AMEX | 9 |
| MADA | MADA | 9 |
| MEEZA | MEEZA | 9 |
| KNET | KNET | 1 |
| NAPS | NAPS | 1 |
| STC Pay | STCPAY | 1 |
| Tabby | TABBY | 1 |
| Tamara | TAMARA | 1 |
| valU | VALU | 1 |
| BENEFIT | BENEFIT | 1 |
| OmanNet | OMANNET | 1 |

---

## Error Handling Best Practices

1. **Log all error codes** with merchant_reference and fort_id
2. **Implement retry logic** based on payment method limits
3. **Display user-friendly messages** to customers
4. **Handle signature errors** by recalculating signature
5. **Check 3DS URL** when response code is 20064
6. **Verify amount formatting** for currency decimals
7. **Use unique merchant_reference** for each transaction
8. **Contact support** for technical errors (codes 006, 051)

---

## Support

**Technical Support:** merchantsupport-ps@amazon.com
