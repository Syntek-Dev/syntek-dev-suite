# Data Anonymisation

## Overview

Data anonymisation patterns for GDPR compliance including pseudonymisation, data masking, and cookie consent implementations.

## Metadata

| Property            | Value                             |
| ------------------- | --------------------------------- |
| **Example Version** | 1.0.0                             |
| **Last Updated**    | 2025-01                           |
| **Stacks**          | All (TALL, Django, React, Mobile) |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [Anonymisation Strategies](#anonymisation-strategies)
  - [Pseudonymisation SQL Example](#pseudonymisation-sql-example)
- [Cookie Consent Implementation](#cookie-consent-implementation)
  - [JavaScript Cookie Consent Manager](#javascript-cookie-consent-manager)
  - [Cookie Banner HTML](#cookie-banner-html)
- [Breach Notification Template](#breach-notification-template)
  - [72-Hour Breach Notification Procedure](#72-hour-breach-notification-procedure)
  - [Breach Detection Logging](#breach-detection-logging)


## Anonymisation Strategies

| Data Type       | Strategy                 | Example                           |
| --------------- | ------------------------ | --------------------------------- |
| User IDs        | Replace with hash        | `ANON-a1b2c3d4`                   |
| Email           | Remove entirely          | `null`                            |
| Name            | Remove entirely          | `null`                            |
| Transaction IDs | Keep (legal requirement) | `TXN-12345`                       |
| IP addresses    | Hash                     | `hash(ip + salt)`                 |
| Dates           | Generalise               | `2025-01` instead of `2025-01-15` |

### Pseudonymisation SQL Example

```sql
-- Anonymise user data while keeping transaction history
UPDATE users
SET
    email_hash = SHA2(CONCAT(id, 'anonymisation-salt'), 256),
    email_encrypted = NULL,
    full_name_encrypted = NULL,
    phone_encrypted = NULL,
    address_encrypted = NULL,
    password = NULL,
    status = 'anonymised',
    anonymised_at = NOW()
WHERE id = :user_id;

-- Keep order history with anonymised user reference
-- Orders table retains: order_id, total, date, products
-- User reference becomes: 'ANON-{hash}'
```

---

## Cookie Consent Implementation

### JavaScript Cookie Consent Manager

```javascript
/**
 * cookieConsent.js
 *
 * GDPR-compliant cookie consent management.
 * Stores user preferences and controls third-party script loading.
 */

const CookieConsent = {
  // Cookie categories
  categories: {
    necessary: true,      // Always enabled, no consent needed
    functional: false,    // Preferences, language settings
    analytics: false,     // Google Analytics, Mixpanel
    marketing: false,     // Ad tracking, retargeting
  },

  /**
   * Initialises cookie consent from stored preferences.
   * Loads saved preferences or shows consent banner.
   */
  init() {
    const saved = this.getSavedPreferences();
    if (saved) {
      this.categories = { ...this.categories, ...saved };
      this.applyPreferences();
    } else {
      this.showBanner();
    }
  },

  /**
   * Retrieves saved cookie preferences from localStorage.
   *
   * @returns Saved preferences object or null
   */
  getSavedPreferences() {
    const saved = localStorage.getItem('cookie_consent');
    return saved ? JSON.parse(saved) : null;
  },

  /**
   * Saves cookie preferences to localStorage.
   *
   * @param preferences - Object with category consent states
   */
  savePreferences(preferences) {
    this.categories = { ...this.categories, ...preferences };
    localStorage.setItem('cookie_consent', JSON.stringify(this.categories));
    localStorage.setItem('cookie_consent_date', new Date().toISOString());
    this.applyPreferences();
    this.hideBanner();
  },

  /**
   * Applies current preferences by loading/blocking scripts.
   */
  applyPreferences() {
    // Analytics
    if (this.categories.analytics) {
      this.loadScript('https://www.googletagmanager.com/gtag/js?id=GA_ID');
    }

    // Marketing
    if (this.categories.marketing) {
      this.loadScript('https://connect.facebook.net/en_US/fbevents.js');
    }
  },

  /**
   * Dynamically loads a third-party script.
   *
   * @param src - Script URL to load
   */
  loadScript(src) {
    if (document.querySelector(`script[src="${src}"]`)) return;

    const script = document.createElement('script');
    script.src = src;
    script.async = true;
    document.head.appendChild(script);
  },

  /**
   * Accepts all cookie categories.
   */
  acceptAll() {
    this.savePreferences({
      functional: true,
      analytics: true,
      marketing: true,
    });
  },

  /**
   * Accepts only necessary cookies.
   */
  acceptNecessary() {
    this.savePreferences({
      functional: false,
      analytics: false,
      marketing: false,
    });
  },

  /**
   * Shows the cookie consent banner.
   */
  showBanner() {
    document.getElementById('cookie-consent')?.classList.remove('hidden');
  },

  /**
   * Hides the cookie consent banner.
   */
  hideBanner() {
    document.getElementById('cookie-consent')?.classList.add('hidden');
  },
};

// Initialise on page load
document.addEventListener('DOMContentLoaded', () => CookieConsent.init());
```

### Cookie Banner HTML

```html
<!-- GDPR-compliant cookie banner -->
<div id="cookie-consent" class="cookie-banner hidden">
  <div class="cookie-content">
    <p>
      We use cookies to improve your experience. Some are necessary for the site
      to work, while others help us understand how you use the site.
    </p>
    <div class="cookie-actions">
      <button onclick="CookieConsent.acceptAll()" class="btn-primary">
        Accept All
      </button>
      <button onclick="CookieConsent.acceptNecessary()" class="btn-secondary">
        Necessary Only
      </button>
      <button onclick="showCookieSettings()" class="btn-link">
        Customise
      </button>
    </div>
  </div>
</div>
```

---

## Breach Notification Template

### 72-Hour Breach Notification Procedure

```markdown
# Data Breach Notification

**Date of Discovery:** [DD/MM/YYYY HH:MM]
**Reported to ICO:** [Yes/No - Must be within 72 hours]
**ICO Reference:** [If reported]

## 1. Nature of the Breach

**Type of breach:**
- [ ] Confidentiality breach (unauthorised access)
- [ ] Integrity breach (data altered)
- [ ] Availability breach (data lost/destroyed)

**Description:**
[Brief description of what happened]

## 2. Categories of Data Affected

- [ ] Names
- [ ] Email addresses
- [ ] Phone numbers
- [ ] Addresses
- [ ] Financial data
- [ ] Health data
- [ ] Other: [specify]

## 3. Number of Data Subjects Affected

- Approximate number: [X]
- User groups affected: [describe]

## 4. Likely Consequences

[Describe potential impact on data subjects]

## 5. Measures Taken

**Immediate actions:**
1. [Action taken]
2. [Action taken]

**Preventive measures:**
1. [Measure planned]
2. [Measure planned]

## 6. Communication to Data Subjects

**Required:** [Yes/No - required if high risk to rights and freedoms]
**Method:** [Email/Post/Website notice]
**Date sent:** [DD/MM/YYYY]

## 7. Internal Sign-off

**DPO:** [Name] - [Date]
**IT Security:** [Name] - [Date]
**Legal:** [Name] - [Date]
```

### Breach Detection Logging

```php
// Log access to sensitive data for breach detection
DB::table('data_access_logs')->insert([
    'user_id' => auth()->id(),
    'resource_type' => 'user_pii',
    'resource_id' => $userId,
    'action' => 'view',
    'ip_hash' => hash_hmac('sha256', request()->ip(), config('app.key')),
    'user_agent' => request()->userAgent(),
    'created_at' => now(),
]);

// Alert on unusual patterns
if ($this->detectAnomalousAccess($userId)) {
    Log::channel('security')->warning('Anomalous PII access detected', [
        'user_id' => auth()->id(),
        'target_user_id' => $userId,
        'access_count' => $accessCount,
    ]);

    // Notify security team
    Notification::route('slack', config('security.slack_webhook'))
        ->notify(new SuspiciousActivityNotification($details));
}
```
