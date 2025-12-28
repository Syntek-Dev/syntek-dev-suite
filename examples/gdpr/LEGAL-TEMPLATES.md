# Legal Templates (Privacy Policy & Terms)

## Overview

Markdown templates for Privacy Policy and Terms & Conditions. These MUST be stored as `.md` files for client review, with HTML pages rendering from these source files. This document also provides production-ready UI components for cookie consent, privacy policy display, and GDPR compliance across all supported stacks.

## Metadata

| Property            | Value                                                              |
| ------------------- | ------------------------------------------------------------------ |
| **Example Version** | 2.0.0                                                              |
| **Last Updated**    | 2025-12                                                            |
| **Stacks**          | TALL (Laravel 12.x), Django 6.x, Next.js 16.x, React Native 0.83.x |

**CRITICAL:** These templates require legal review before use in production.

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [File Structure](#file-structure)
- [Privacy Policy Template](#privacy-policy-template)
  - [content/legal/privacy-policy.md](#contentlegalprivacy-policymd)
- [Terms and Conditions Template](#terms-and-conditions-template)
  - [content/legal/terms-and-conditions.md](#contentlegalterms-and-conditionsmd)
- [Client Review Workflow](#client-review-workflow)


## File Structure

```
content/
├── legal/
│   ├── privacy-policy.md       # Source markdown for privacy policy
│   └── terms-and-conditions.md # Source markdown for T&Cs
```

---

## Privacy Policy Template

### content/legal/privacy-policy.md

```markdown
---
title: "Privacy Policy"

## Table of Contents

last_updated: "YYYY-MM-DD"
version: "1.0"
---

# Privacy Policy

**Last Updated:** [Date]

## 1. Introduction

[Company Name] ("Company", "we", "us", "our") is committed to protecting your personal data.
This privacy policy explains how we collect, use, and protect your information.

## 2. Data Controller

**Company Name:** [Legal Company Name]
**Address:** [Registered Address]
**Data Protection Officer:** [DPO Name]
**Contact Email:** privacy@[domain].com

## 3. What Data We Collect

### 3.1 Information You Provide
- **Account Information:** Name, email address, password
- **Profile Information:** [List specific fields]
- **Payment Information:** [If applicable]
- **Communications:** Messages sent through our platform

### 3.2 Information Collected Automatically
- **Device Information:** Browser type, operating system, device identifiers
- **Usage Data:** Pages visited, features used, time spent
- **IP Address:** Collected for security purposes (hashed for analytics, encrypted for audit)
- **Cookies:** See our Cookie Policy section below

## 4. How We Use Your Data

| Purpose                     | Legal Basis         | Retention                  |
| --------------------------- | ------------------- | -------------------------- |
| Account management          | Contract            | Account lifetime + 30 days |
| Security & fraud prevention | Legitimate interest | 90 days                    |
| Marketing (with consent)    | Consent             | Until withdrawn            |
| Analytics (anonymised)      | Legitimate interest | 2 years                    |

## 5. Data Sharing

We share your data with:
- **Service Providers:** [List categories: hosting, payment, email]
- **Legal Requirements:** When required by law
- **Business Transfers:** In case of merger or acquisition

We do NOT sell your personal data.

## 6. Your Rights

Under GDPR, you have the right to:
- **Access:** Request a copy of your personal data
- **Rectification:** Correct inaccurate data
- **Erasure:** Request deletion of your data
- **Portability:** Receive your data in machine-readable format
- **Object:** Object to processing based on legitimate interest
- **Withdraw Consent:** Revoke consent at any time

**To exercise your rights:** [Link to data request form]

## 7. Data Security

We protect your data using:
- Encryption at rest (AES-256)
- Transport Layer Security (TLS 1.3)
- Hashed lookups for PII (irreversible)
- Permission-based access controls

## 8. Cookie Policy

See Section 9 for detailed cookie information.

## 9. Contact Us

For privacy concerns, contact us at:
- **Email:** privacy@[domain].com
- **Address:** [Physical address]

## 10. Changes to This Policy

We may update this policy periodically. Significant changes will be notified via email.
```

---

## Terms and Conditions Template

### content/legal/terms-and-conditions.md

```markdown
---
title: "Terms & Conditions"

## Table of Contents

last_updated: "YYYY-MM-DD"
version: "1.0"
---

# Terms & Conditions

**Last Updated:** [Date]

## 1. Introduction

These Terms & Conditions ("Terms") govern your use of [Service Name] ("Service")
operated by [Company Name] ("Company", "we", "us", "our").

By using the Service, you agree to these Terms. If you disagree, please do not use the Service.

## 2. Definitions

- **"Service"**: The [website/application/platform] located at [URL]
- **"User"**, **"you"**: Any person accessing or using the Service
- **"Account"**: A registered user profile on the Service
- **"Content"**: Any data, text, files, or materials uploaded to the Service

## 3. Account Registration

### 3.1 Eligibility
- You must be at least 18 years old (or legal age in your jurisdiction)
- You must provide accurate and complete registration information
- You are responsible for maintaining account security

### 3.2 Account Security
- Keep your password confidential
- Notify us immediately of any unauthorised access
- We are not liable for losses due to compromised credentials

## 4. Acceptable Use

### 4.1 Permitted Use
You may use the Service for lawful purposes in accordance with these Terms.

### 4.2 Prohibited Activities
You must NOT:
- Violate any applicable laws or regulations
- Infringe intellectual property rights
- Upload malicious code or interfere with the Service
- Attempt to gain unauthorised access to our systems
- Use the Service for spam, phishing, or fraudulent activities
- Collect user data without consent

## 5. User Content

### 5.1 Ownership
You retain ownership of Content you upload.

### 5.2 Licence Grant
By uploading Content, you grant us a non-exclusive, worldwide licence to use,
display, and distribute your Content solely for operating the Service.

### 5.3 Content Responsibility
You are solely responsible for Content you upload and must ensure it does not
violate these Terms or applicable laws.

## 6. Intellectual Property

The Service and its original content (excluding User Content) are protected by
copyright, trademark, and other intellectual property laws.

## 7. Payment Terms

[If applicable]
### 7.1 Fees
[Describe fee structure]

### 7.2 Billing
[Describe billing cycle, payment methods]

### 7.3 Refunds
[Describe refund policy]

## 8. Termination

### 8.1 By You
You may terminate your account at any time through [account settings / contact support].

### 8.2 By Us
We may suspend or terminate your access if you violate these Terms.

### 8.3 Effect of Termination
Upon termination:
- Your right to use the Service ceases immediately
- We may delete your data in accordance with our Privacy Policy
- Sections 6, 9, 10, and 11 survive termination

## 9. Disclaimer of Warranties

THE SERVICE IS PROVIDED "AS IS" WITHOUT WARRANTIES OF ANY KIND.
We do not warrant that the Service will be uninterrupted or error-free.

## 10. Limitation of Liability

TO THE MAXIMUM EXTENT PERMITTED BY LAW, WE SHALL NOT BE LIABLE FOR:
- Indirect, incidental, or consequential damages
- Loss of profits, data, or business opportunities
- Damages exceeding the amount paid by you in the preceding 12 months

## 11. Indemnification

You agree to indemnify and hold harmless the Company from any claims arising from:
- Your use of the Service
- Your violation of these Terms
- Your infringement of third-party rights

## 12. Governing Law

These Terms are governed by the laws of [Jurisdiction].
Disputes shall be resolved in the courts of [Jurisdiction].

## 13. Changes to Terms

We may modify these Terms at any time. Continued use after changes constitutes acceptance.

## 14. Contact Us

For questions about these Terms:
- **Email:** legal@[domain].com
- **Address:** [Physical address]
```

---

## Client Review Workflow

1. **Draft Stage:** Create/edit `.md` files in `content/legal/`
2. **Client Review:** Client reviews markdown files directly (readable without rendering)
3. **Legal Review:** Solicitor/lawyer reviews and approves content
4. **Deployment:** Markdown automatically renders to HTML on the live site
5. **Version Control:** All changes tracked in git with full history
