# PII Database Protection & Access Control

## Overview

Database-level protection for Personally Identifiable Information (PII). Includes permission-gated access, audit logging, and verification patterns for security reviews.

**CRITICAL:** Coordinate with the GDPR agent for full PII protection. This section covers security enforcement.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
| **Laravel** | 12.x |
| **Django** | 6.x |
| **Next.js** | 16.x |
| **React Native** | 0.83.x |
| **PHP** | 8.4 |
| **Python** | 3.14 |
| **Node.js** | 24.x |
| **TypeScript** | 5.9 |
| **Stacks** | TALL, Django/Wagtail, React/Next.js, React Native |

---

## Table of Contents

- [PII Database Protection \& Access Control](#pii-database-protection--access-control)
  - [Overview](#overview)
  - [Metadata](#metadata)
  - [Table of Contents](#table-of-contents)
  - [PII Table Schema](#pii-table-schema)
    - [MySQL/MariaDB](#mysqlmariadb)
  - [TALL Stack - Laravel PII Access](#tall-stack---laravel-pii-access)
  - [PII Access Service - Laravel](#pii-access-service---laravel)
    - [app/Services/PiiAccessService.php](#appservicespiiaccessservicephp)
  - [PII Audit Command - Laravel](#pii-audit-command---laravel)
    - [app/Console/Commands/AuditPiiHandling.php](#appconsolecommandsauditpiihandlingphp)
  - [PII Permissions Matrix](#pii-permissions-matrix)
  - [PII Verification Checklist](#pii-verification-checklist)
    - [Database Schema Verification](#database-schema-verification)
    - [Code Verification Patterns](#code-verification-patterns)
      - [Correct: Hash Before Lookup](#correct-hash-before-lookup)
      - [Incorrect: Plaintext Lookup](#incorrect-plaintext-lookup)
      - [Correct: Encrypt Before Storage](#correct-encrypt-before-storage)
      - [Incorrect: Plaintext Storage](#incorrect-plaintext-storage)
    - [PII Verification During Code Review](#pii-verification-during-code-review)
    - [Verify PII Permissions](#verify-pii-permissions)


## PII Table Schema

### MySQL/MariaDB

```sql
-- Separate PII into dedicated table with restricted access
CREATE TABLE user_pii (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT UNSIGNED NOT NULL UNIQUE,

    -- Hashed columns for lookup (irreversible - cannot recover original)
    email_hash VARCHAR(64) NOT NULL,
    phone_hash VARCHAR(64),

    -- Encrypted columns for storage (reversible with encryption key)
    email_encrypted TEXT NOT NULL,
    phone_encrypted TEXT,
    full_name_encrypted TEXT NOT NULL,
    address_encrypted TEXT,
    dob_encrypted TEXT,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_email_hash (email_hash),
    INDEX idx_phone_hash (phone_hash)
);

-- Create restricted database user for PII access
CREATE USER 'pii_reader'@'%' IDENTIFIED BY 'strong_password_here';
GRANT SELECT ON database.user_pii TO 'pii_reader'@'%';

-- Application default user should NOT have direct PII access
REVOKE ALL ON database.user_pii FROM 'app_user'@'%';
```

---

## TALL Stack - Laravel PII Access

## PII Access Service - Laravel

### app/Services/PiiAccessService.php

```php
<?php

/**
 * PII Access Gate
 *
 * Controls which users can access personally identifiable information.
 * All PII access is logged for audit purposes.
 *
 * This service enforces permission-based access to PII data, ensuring
 * that only authorised users can retrieve sensitive information.
 */

namespace App\Services;

use App\Models\User;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Crypt;

class PiiAccessService
{
    /**
     * Retrieves PII for a target user if the requesting user has permission.
     * All access attempts are logged.
     *
     * @param User $requestingUser The user requesting PII access
     * @param User $targetUser The user whose PII is being accessed
     * @return array|null The decrypted PII or null if access denied
     */
    public function getPiiWithPermission(User $requestingUser, User $targetUser): ?array
    {
        // Check if user can access their own PII
        if ($requestingUser->id === $targetUser->id) {
            if (!$requestingUser->hasPermission('pii.access')) {
                $this->logAccessDenied($requestingUser, $targetUser, 'own_data');
                return null;
            }
        } else {
            // Check if user can access others' PII
            if (!$requestingUser->hasPermission('pii.access.others')) {
                $this->logAccessDenied($requestingUser, $targetUser, 'others_data');
                return null;
            }
        }

        // Log successful access for audit trail
        $this->logAccess($requestingUser, $targetUser);

        // Use separate database connection with PII read permissions
        $encryptedPii = DB::connection('pii_readonly')
            ->table('user_pii')
            ->where('user_id', $targetUser->id)
            ->first();

        if (!$encryptedPii) {
            return null;
        }

        // Decrypt PII fields
        return $this->decryptPiiFields($encryptedPii);
    }

    /**
     * Decrypts encrypted PII fields.
     *
     * @param object $encryptedPii The encrypted PII record
     * @return array The decrypted PII data
     */
    protected function decryptPiiFields(object $encryptedPii): array
    {
        return [
            'user_id' => $encryptedPii->user_id,
            'email' => $encryptedPii->email_encrypted
                ? Crypt::decryptString($encryptedPii->email_encrypted)
                : null,
            'phone' => $encryptedPii->phone_encrypted
                ? Crypt::decryptString($encryptedPii->phone_encrypted)
                : null,
            'full_name' => $encryptedPii->full_name_encrypted
                ? Crypt::decryptString($encryptedPii->full_name_encrypted)
                : null,
            'address' => $encryptedPii->address_encrypted
                ? Crypt::decryptString($encryptedPii->address_encrypted)
                : null,
            'dob' => $encryptedPii->dob_encrypted
                ? Crypt::decryptString($encryptedPii->dob_encrypted)
                : null,
        ];
    }

    /**
     * Finds a user by hashed email.
     *
     * @param string $email The plaintext email to search for
     * @return User|null The user if found
     */
    public function findUserByEmail(string $email): ?User
    {
        $emailHash = hash_hmac('sha256', strtolower($email), config('app.key'));

        $piiRecord = DB::connection('pii_readonly')
            ->table('user_pii')
            ->where('email_hash', $emailHash)
            ->first();

        if (!$piiRecord) {
            return null;
        }

        return User::find($piiRecord->user_id);
    }

    /**
     * Logs successful PII access for audit compliance.
     *
     * @param User $requestingUser The user who accessed PII
     * @param User $targetUser The user whose PII was accessed
     */
    protected function logAccess(User $requestingUser, User $targetUser): void
    {
        DB::table('pii_access_logs')->insert([
            'requesting_user_id' => $requestingUser->id,
            'target_user_id' => $targetUser->id,
            'access_type' => 'read',
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'created_at' => now(),
        ]);
    }

    /**
     * Logs denied PII access attempts for security monitoring.
     *
     * @param User $requestingUser The user who attempted access
     * @param User $targetUser The user whose PII was requested
     * @param string $reason The reason for denial
     */
    protected function logAccessDenied(User $requestingUser, User $targetUser, string $reason): void
    {
        logger()->warning('PII access denied', [
            'requesting_user_id' => $requestingUser->id,
            'target_user_id' => $targetUser->id,
            'reason' => $reason,
            'ip' => request()->ip(),
        ]);

        DB::table('pii_access_logs')->insert([
            'requesting_user_id' => $requestingUser->id,
            'target_user_id' => $targetUser->id,
            'access_type' => 'denied',
            'reason' => $reason,
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'created_at' => now(),
        ]);
    }
}
```

---

## PII Audit Command - Laravel

### app/Console/Commands/AuditPiiHandling.php

```php
<?php

/**
 * PII Audit Command
 *
 * Scans the codebase for potential PII handling issues.
 * Run this during security audits to verify PII protection.
 */

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\File;

class AuditPiiHandling extends Command
{
    protected $signature = 'security:audit-pii';
    protected $description = 'Audit codebase for PII handling issues';

    public function handle(): int
    {
        $issues = [];

        $this->info('Scanning codebase for PII handling issues...');

        // Check for plaintext email queries
        $issues = array_merge($issues, $this->checkPlaintextQueries());

        // Check for PII in logs
        $issues = array_merge($issues, $this->checkLogging());

        // Check for PII in URLs
        $issues = array_merge($issues, $this->checkUrls());

        // Check for proper hash usage
        $issues = array_merge($issues, $this->checkHashUsage());

        if (empty($issues)) {
            $this->info('✓ No PII handling issues found');
            return 0;
        }

        $this->error('Found ' . count($issues) . ' potential PII issues:');
        foreach ($issues as $issue) {
            $this->line("  - {$issue}");
        }

        return 1;
    }

    /**
     * Checks for plaintext PII queries in the codebase.
     *
     * @return array Found issues
     */
    protected function checkPlaintextQueries(): array
    {
        $issues = [];
        $patterns = [
            "where('email'," => 'Plaintext email query',
            "where('phone'," => 'Plaintext phone query',
            "->email = " => 'Direct email assignment',
            "->phone = " => 'Direct phone assignment',
        ];

        $files = File::allFiles(app_path());

        foreach ($files as $file) {
            if ($file->getExtension() !== 'php') {
                continue;
            }

            $content = $file->getContents();

            foreach ($patterns as $pattern => $description) {
                if (str_contains($content, $pattern)) {
                    $issues[] = "{$description} in {$file->getRelativePathname()}";
                }
            }
        }

        return $issues;
    }

    /**
     * Checks for PII being logged.
     *
     * @return array Found issues
     */
    protected function checkLogging(): array
    {
        $issues = [];
        $logPatterns = [
            "Log::info(['email'" => 'Email logged',
            "logger()->info(['email'" => 'Email logged',
            "Log::debug(['phone'" => 'Phone logged',
        ];

        $files = File::allFiles(app_path());

        foreach ($files as $file) {
            if ($file->getExtension() !== 'php') {
                continue;
            }

            $content = $file->getContents();

            foreach ($logPatterns as $pattern => $description) {
                if (str_contains($content, $pattern)) {
                    $issues[] = "{$description} in {$file->getRelativePathname()}";
                }
            }
        }

        return $issues;
    }

    /**
     * Checks for PII in URLs.
     *
     * @return array Found issues
     */
    protected function checkUrls(): array
    {
        $issues = [];

        // Check route files for email/phone in URLs
        $routeFiles = [
            base_path('routes/web.php'),
            base_path('routes/api.php'),
        ];

        foreach ($routeFiles as $file) {
            if (!File::exists($file)) {
                continue;
            }

            $content = File::get($file);

            if (preg_match('/{email}|{phone}/', $content)) {
                $issues[] = "PII in URL parameter in {$file}";
            }
        }

        return $issues;
    }

    /**
     * Checks for proper hash usage.
     *
     * @return array Found issues
     */
    protected function checkHashUsage(): array
    {
        $issues = [];

        // Check that hash_hmac is used, not md5/sha1 alone
        $files = File::allFiles(app_path());

        foreach ($files as $file) {
            if ($file->getExtension() !== 'php') {
                continue;
            }

            $content = $file->getContents();

            // Check for weak hashing
            if (preg_match('/md5\(\$.*email/i', $content)) {
                $issues[] = "Weak MD5 hash for email in {$file->getRelativePathname()}";
            }

            if (preg_match('/sha1\(\$.*email/i', $content)) {
                $issues[] = "Weak SHA1 hash for email in {$file->getRelativePathname()}";
            }
        }

        return $issues;
    }
}
```

---

## PII Permissions Matrix

| Permission | Can View | Can Export | Can Delete | Typical Roles |
|------------|----------|------------|------------|---------------|
| `pii.access` | Own PII | No | No | All users |
| `pii.access.others` | Others' PII | No | No | Support |
| `pii.export` | All PII | Yes | No | Admin, DPO |
| `pii.delete` | All PII | Yes | Yes | Admin, DPO |
| `pii.audit` | Access logs | Logs only | No | Security, DPO |

---

## PII Verification Checklist

### Database Schema Verification

```bash
# Check for plaintext PII columns in schema
grep -r "email VARCHAR\|email TEXT\|phone VARCHAR" database/migrations/
# Should return empty or only show *_encrypted columns

# Verify hash columns exist
grep -r "email_hash\|phone_hash" database/migrations/
# Should find hash columns for lookups
```

### Code Verification Patterns

#### Correct: Hash Before Lookup

```php
// CORRECT: Hash before lookup
$hash = hash_hmac('sha256', strtolower($email), config('app.key'));
$user = UserPii::where('email_hash', $hash)->first();
```

#### Incorrect: Plaintext Lookup

```php
// INCORRECT: Plaintext lookup (SECURITY ISSUE)
$user = User::where('email', $email)->first();  // BAD!
```

#### Correct: Encrypt Before Storage

```php
// CORRECT: Encrypt before storage
$encrypted = Crypt::encryptString($email);
$pii->email_encrypted = $encrypted;
```

#### Incorrect: Plaintext Storage

```php
// INCORRECT: Plaintext storage (SECURITY ISSUE)
$user->email = $email;  // BAD if storing in DB as plaintext
```

### PII Verification During Code Review

| Pattern | Status | Action Required |
|---------|--------|-----------------|
| `->email = $value` directly to User model | Warning | Verify PII service is used |
| `User::where('email', $value)` | Critical | Must use hash lookup |
| `logger()->info(['email' => $user->email])` | Critical | PII in logs |
| `return response()->json($user)` | Warning | Check hidden fields |
| `Crypt::encryptString($pii)` | Good | Correct pattern |
| `hash_hmac('sha256', $value, $key)` | Good | Correct pattern |

### Verify PII Permissions

```php
// Check permission is required before PII access
if (!$user->hasPermission('pii.access')) {
    abort(403, 'PII access denied');
}

// Verify audit logging exists
logger()->info('PII accessed', [
    'user_id' => $requestingUser->id,  // Log who accessed
    'target_user_id' => $targetUser->id,  // Log whose PII
    'ip' => request()->ip(),
    'timestamp' => now(),
]);
```
