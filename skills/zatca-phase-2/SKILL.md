---
name: ZATCA Phase 2
description: help understand zatca phase 2 integration
metadata:
  tags: []
  category: general
  version: 1.0.0
  public: false
  created: '2026-05-20'
  updated: '2026-05-20'
---
# ZATCA Phase 2 Integration Skill

## Purpose

This skill enables an AI agent to integrate Saudi Arabia ZATCA Phase 2 (FATOORAH) e-invoicing into any ERP, POS, SaaS, ecommerce, or accounting system.

The agent should:
- understand ZATCA Phase 2 architecture
- generate compliant invoice XML
- implement signing and QR generation
- integrate with sandbox and production APIs
- automate onboarding
- validate invoices
- troubleshoot common failures

---

# Core Knowledge

## What is ZATCA Phase 2?

ZATCA Phase 2 ("Integration Phase") requires taxpayer systems to integrate directly with ZATCA systems.

Requirements include:
- UBL 2.1 XML invoices
- invoice hashing
- cryptographic signing
- QR code generation
- API submission
- clearance/reporting workflows
- device onboarding
- CSID certificates

---

# Official References

Developer portal:
https://sandbox.zatca.gov.sa/

Main documentation:
https://www.zatca.gov.sa/en/E-Invoicing/SystemsDevelopers

Educational library:
https://zatca.gov.sa/en/E-Invoicing/Introduction/Guidelines/Pages/default.aspx

Important documents:
- XML Implementation Standard
- Security Features Implementation Standards
- E-Invoicing Detailed Technical Guideline
- Data Dictionary
- Developer Portal Manual

---

# Integration Architecture

## High-Level Flow

1. Seller creates invoice
2. ERP/POS generates UBL XML
3. XML canonicalization
4. Hash invoice
5. Sign invoice using certificate
6. Generate TLV QR
7. Submit to ZATCA API
8. Receive clearance/reporting response
9. Store signed XML and response

---

# Invoice Types

## Standard Tax Invoice (B2B)

Requires:
- real-time clearance
- synchronous approval
- cleared invoice returned from ZATCA

API:
- Clearance API

---

## Simplified Tax Invoice (B2C)

Requires:
- reporting within allowed timeframe
- asynchronous reporting

API:
- Reporting API

---

# Required Technical Components

## XML Format

Use:
- UBL 2.1

Mandatory:
- ZATCA-specific extensions
- namespaces
- business rules

Critical fields:
- UUID
- ICV
- PIH
- seller VAT
- timestamps
- VAT breakdowns

---

# Security Requirements

## Required Features

### Invoice Hashing
SHA256 hash of canonical XML.

### Digital Signature
XAdES-BES signature required.

### Cryptographic Stamp
Generated using compliant certificate.

### QR Code
TLV encoded Base64 QR required.

QR contains:
1. Seller name
2. VAT number
3. Timestamp
4. Invoice total
5. VAT total
6. Invoice hash
7. Signature
8. Public key
9. CA signature

---

# Certificate Lifecycle

## Device Onboarding

Flow:
1. Generate CSR
2. Request compliance CSID
3. Validate sample invoices
4. Request production CSID
5. Store certificates securely

---

# Important Concepts

## CSID
Cryptographic Stamp Identifier.

Types:
- Compliance CSID
- Production CSID

---

## PIH
Previous Invoice Hash.

Creates invoice chain integrity.

---

## ICV
Invoice Counter Value.

Sequential invoice counter.

---

# API Categories

## Compliance APIs

Used during onboarding/testing.

Functions:
- validate XML
- test signatures
- compliance checks

---

## Clearance APIs

Used for B2B invoices.

Behavior:
- synchronous
- returns cleared invoice

---

## Reporting APIs

Used for B2C invoices.

Behavior:
- asynchronous reporting

---

# Sandbox vs Production

## Sandbox

Purpose:
- development
- testing
- onboarding

Uses:
- sandbox endpoints
- test certificates

---

## Production

Requirements:
- approved onboarding
- production CSID
- valid compliance checks

---

# Agent Responsibilities

When integrating ZATCA into a system:

## The agent MUST:

### 1. Detect invoice type
Determine:
- B2B => Clearance
- B2C => Reporting

### 2. Generate valid XML
Must conform to:
- UBL 2.1
- ZATCA business rules

### 3. Generate UUID
Unique invoice UUID required.

### 4. Maintain PIH chain
Store previous invoice hash.

### 5. Maintain ICV counter
Increment sequentially.

### 6. Sign XML
Apply:
- canonicalization
- SHA256 hashing
- XAdES-BES signature

### 7. Generate QR
Produce TLV Base64 QR.

### 8. Submit to APIs
Use correct endpoint.

### 9. Handle retries
Queue failed submissions.

### 10. Persist everything
Store:
- original XML
- signed XML
- API response
- hashes
- QR payload

---

# Recommended System Design

## Suggested Modules

### invoice-builder
Creates compliant XML.

### signing-service
Handles:
- hashing
- canonicalization
- XAdES

### qr-service
Generates TLV QR.

### zatca-client
Handles API communication.

### onboarding-service
Manages:
- CSR
- CSID
- certificates

### validation-service
Runs SDK validations.

---

# Recommended Async Architecture

Avoid synchronous checkout blocking.

Recommended:
- queue invoice jobs
- async submission
- exponential retry
- dead-letter queue

Reason:
- ZATCA outages
- latency
- certificate failures
- rate limits

---

# Common Errors

## Invalid XML namespace
Fix:
- verify namespaces exactly

## Signature mismatch
Fix:
- canonicalization errors
- whitespace changes

## PIH mismatch
Fix:
- invoice ordering issue

## Invalid QR
Fix:
- TLV encoding issue

## Certificate rejected
Fix:
- wrong CSID type
- expired cert
- sandbox/prod mismatch

---

# Best Practices

## Security

- never expose private keys
- use HSM/KMS if possible
- encrypt certificate storage

---

## Reliability

- queue submissions
- implement retries
- support replay/reprocessing

---

## Auditing

Store:
- signed XML
- hashes
- API responses
- timestamps

Retention should comply with Saudi regulations.

---

# Suggested Tech Stack

## Backend Languages

Good options:
- Node.js
- Java
- Kotlin
- C#
- Go
- Python

---

# Useful Libraries

## XML
- lxml
- xmlsec
- JAXB
- xmldsig libraries

## QR
- TLV encoders
- Base64 tools

## Crypto
- OpenSSL
- xmlsec
- BouncyCastle

---

# Integration Checklist

## Minimum Viable Compliance

- [ ] UBL XML generation
- [ ] UUID generation
- [ ] ICV sequencing
- [ ] PIH chaining
- [ ] XML canonicalization
- [ ] SHA256 hashing
- [ ] XAdES signature
- [ ] TLV QR generation
- [ ] Sandbox onboarding
- [ ] Compliance validation
- [ ] Clearance/reporting API calls
- [ ] Retry handling
- [ ] XML archival

---

# Agent Workflow Example

## Example End-to-End Flow

1. Create invoice in ERP
2. Generate UBL XML
3. Add UUID
4. Add ICV
5. Fetch previous PIH
6. Canonicalize XML
7. Generate hash
8. Sign invoice
9. Generate QR
10. Validate locally
11. Submit to ZATCA
12. Store cleared XML
13. Mark invoice compliant

---

# Important Implementation Notes

## B2B invoices
Must be cleared BEFORE sharing with buyer.

## B2C invoices
Can be issued immediately and reported later.

---

# Common Production Pitfalls

## Race conditions

Concurrent invoice generation can break:
- PIH chain
- ICV sequencing

Mitigation:
- locking
- transactional sequencing

---

## Time drift

Incorrect timestamps may fail validation.

Mitigation:
- NTP sync

---

## Certificate rotation

Production CSIDs expire.

Mitigation:
- proactive renewal automation

---

# Validation Strategy

Always validate:
1. XML schema
2. Business rules
3. Signature
4. QR decoding
5. API compliance

before production submission.

---

# What the Agent Should NEVER Do

- Never skip signing
- Never regenerate PIH incorrectly
- Never alter XML after signing
- Never expose private keys
- Never mix sandbox and production certificates
- Never ignore failed submissions

---

# Recommended Agent Output Format

When generating integration code:
- separate XML generation from signing
- isolate certificate handling
- make API endpoints configurable
- support sandbox and production modes

---

# Helpful Community Knowledge

Known engineering pain points:
- XAdES signing complexity
- canonical XML formatting
- PIH race conditions
- async retry design
- certificate onboarding confusion

Robust integrations usually use:
- queues
- async workers
- retry pipelines
- audit storage

---

# Final Goal

A compliant integration should:
- generate valid invoices
- pass ZATCA validation
- survive retries/failures
- maintain invoice chain integrity
- support onboarding and production
- be reusable across ERP/POS/SaaS systems
