# PII Storage Service

## Metadata

| Property | Value |
|----------|-------|
| **Version** | 2.0.0 |
| **Last Updated** | December 2025 |
| **Status** | Stable |

## Framework Versions Tested

| Framework | Version | Tested Date |
|-----------|---------|-------------|
| Laravel | 12.x | 20/12/2025 |
| Django | 6.x | 20/12/2025 |
| Next.js | 16.x | 20/12/2025 |
| React Native | 0.83.x | 20/12/2025 |
| Prisma | 6.x | 20/12/2025 |
| cryptography (Python) | 43.x | 20/12/2025 |
| PHP | 8.4 | 20/12/2025 |
| Python | 3.14 | 20/12/2025 |
| Node.js | 24 LTS | 20/12/2025 |
| TypeScript | 5.9 | 20/12/2025 |
| MariaDB | 12.x | 20/12/2025 |
| PostgreSQL | 18.x | 20/12/2025 |

---

## Table of Contents

- [PII Storage Service](#pii-storage-service)
  - [Metadata](#metadata)
  - [Framework Versions Tested](#framework-versions-tested)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Hashing vs Encryption Decision](#hashing-vs-encryption-decision)
  - [Database Schema Examples](#database-schema-examples)
    - [MariaDB Schema](#mariadb-schema)
    - [PostgreSQL Schema](#postgresql-schema)
  - [TALL Stack (Laravel/PHP/MariaDB)](#tall-stack-laravelphpmariadb)
    - [Service Class](#service-class)
    - [Model Example](#model-example)
    - [Migration Example](#migration-example)
  - [Django/Wagtail (Python/PostgreSQL)](#djangowagtail-pythonpostgresql)
    - [Service Class](#service-class-1)
    - [Model Example](#model-example-1)
    - [Migration Example](#migration-example-1)
    - [Required Settings](#required-settings)
  - [React/Next.js (TypeScript/Prisma)](#reactnextjs-typescriptprisma)
    - [Required Environment Variables](#required-environment-variables)


## Overview

PII storage services handle secure storage of personally identifiable information using:
1. **Hashing** (HMAC-SHA256) for lookup columns - irreversible
2. **Encryption** (AES-256-GCM / Fernet) for storage - reversible with key

## Hashing vs Encryption Decision

| Use Case | Method | Example |
|----------|--------|---------|
| Authentication (passwords) | **Hashing** (Argon2id/bcrypt) | Never reversible |
| Lookup/matching (email search) | **Hashing** (HMAC-SHA256) | `email_hash` column |
| Display to authorised users | **Encryption** (AES-256) | `email_encrypted` column |
| Financial/legal records | **Encryption** (AES-256-GCM) | `ssn_encrypted` column |

---

## Database Schema Examples

### MariaDB Schema

```sql
-- MariaDB 12.x schema for PII storage
-- Used by TALL Stack (Laravel/PHP)

CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,

    -- Hashed columns for lookup (irreversible)
    email_hash VARCHAR(64) NOT NULL UNIQUE COMMENT 'HMAC-SHA256 hash for email lookup',
    phone_hash VARCHAR(64) NULL COMMENT 'HMAC-SHA256 hash for phone lookup',

    -- Encrypted columns for display (reversible with key)
    email_encrypted TEXT NOT NULL COMMENT 'AES-256-GCM encrypted email address',
    phone_encrypted TEXT NULL COMMENT 'AES-256-GCM encrypted phone number',
    first_name_encrypted TEXT NOT NULL COMMENT 'AES-256-GCM encrypted first name',
    last_name_encrypted TEXT NOT NULL COMMENT 'AES-256-GCM encrypted last name',

    -- Non-PII columns
    password VARCHAR(255) NOT NULL COMMENT 'Argon2id hashed password',
    status ENUM('active', 'suspended', 'deleted') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL COMMENT 'Soft delete timestamp',

    INDEX idx_email_hash (email_hash),
    INDEX idx_phone_hash (phone_hash),
    INDEX idx_status (status),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Audit log for PII access tracking
CREATE TABLE pii_access_logs (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    accessed_by BIGINT UNSIGNED NOT NULL COMMENT 'User who accessed the PII',
    field_accessed VARCHAR(100) NOT NULL COMMENT 'Which PII field was accessed',
    access_reason TEXT NULL COMMENT 'Reason for accessing PII',
    ip_address VARCHAR(45) NULL,
    user_agent TEXT NULL,
    accessed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_accessed_by (accessed_by),
    INDEX idx_accessed_at (accessed_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### PostgreSQL Schema

```sql
-- PostgreSQL 18.x schema for PII storage
-- Used by Django/Wagtail and Next.js/Prisma

CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,

    -- Hashed columns for lookup (irreversible)
    email_hash VARCHAR(64) NOT NULL UNIQUE,
    phone_hash VARCHAR(64) NULL,

    -- Encrypted columns for display (reversible with key)
    email_encrypted TEXT NOT NULL,
    phone_encrypted TEXT NULL,
    first_name_encrypted TEXT NOT NULL,
    last_name_encrypted TEXT NOT NULL,

    -- Non-PII columns
    password VARCHAR(255) NOT NULL,
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'suspended', 'deleted')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE NULL
);

-- Indices for performance
CREATE INDEX idx_users_email_hash ON users(email_hash);
CREATE INDEX idx_users_phone_hash ON users(phone_hash) WHERE phone_hash IS NOT NULL;
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Audit log for PII access tracking
CREATE TABLE pii_access_logs (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    accessed_by BIGINT NOT NULL,
    field_accessed VARCHAR(100) NOT NULL,
    access_reason TEXT NULL,
    ip_address INET NULL,
    user_agent TEXT NULL,
    accessed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Indices for audit logs
CREATE INDEX idx_pii_access_logs_user_id ON pii_access_logs(user_id);
CREATE INDEX idx_pii_access_logs_accessed_by ON pii_access_logs(accessed_by);
CREATE INDEX idx_pii_access_logs_accessed_at ON pii_access_logs(accessed_at);

-- Trigger to update updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- Comments for documentation
COMMENT ON COLUMN users.email_hash IS 'HMAC-SHA256 hash for email lookup (irreversible)';
COMMENT ON COLUMN users.phone_hash IS 'HMAC-SHA256 hash for phone lookup (irreversible)';
COMMENT ON COLUMN users.email_encrypted IS 'Fernet/AES-256-GCM encrypted email address (reversible)';
COMMENT ON COLUMN users.phone_encrypted IS 'Fernet/AES-256-GCM encrypted phone number (reversible)';
COMMENT ON COLUMN users.first_name_encrypted IS 'Fernet/AES-256-GCM encrypted first name (reversible)';
COMMENT ON COLUMN users.last_name_encrypted IS 'Fernet/AES-256-GCM encrypted last name (reversible)';
COMMENT ON COLUMN users.password IS 'Argon2id hashed password (never reversible)';
COMMENT ON COLUMN users.deleted_at IS 'Soft delete timestamp for GDPR compliance';
```

---

## TALL Stack (Laravel/PHP/MariaDB)

### Service Class

```php
<?php
/**
 * PII Storage Service for Laravel 12.x
 *
 * Handles secure storage of Personally Identifiable Information.
 * All PII is either hashed (for lookup) or encrypted (for retrieval).
 * Hashed values are irreversible; encrypted values require the encryption key.
 *
 * @package App\Services
 * @version 2.0.0
 */

namespace App\Services;

use Illuminate\Support\Facades\Crypt;
use Illuminate\Support\Facades\Log;
use Illuminate\Encryption\Encrypter;

class PiiStorageService
{
    /**
     * Generates an irreversible hash for lookup purposes.
     * Used for searching by email/phone without exposing the actual value.
     *
     * @param string $value The PII value to hash
     * @return string The hashed value (irreversible, 64 characters hex)
     */
    public function hashForLookup(string $value): string
    {
        // Normalise input: lowercase and trim whitespace
        $normalised = strtolower(trim($value));

        // Use HMAC-SHA256 with app key for consistent, irreversible hashing
        return hash_hmac('sha256', $normalised, config('app.key'));
    }

    /**
     * Encrypts PII for secure storage with later retrieval.
     * Uses Laravel's AES-256-GCM encryption (secure and authenticated).
     *
     * @param string $value The PII value to encrypt
     * @return string The encrypted value (reversible with key)
     */
    public function encrypt(string $value): string
    {
        return Crypt::encryptString($value);
    }

    /**
     * Decrypts previously encrypted PII.
     * Returns null if decryption fails (tampered data or wrong key).
     *
     * @param string $encryptedValue The encrypted PII value
     * @return string|null The decrypted value or null on failure
     */
    public function decrypt(string $encryptedValue): ?string
    {
        try {
            return Crypt::decryptString($encryptedValue);
        } catch (\Illuminate\Contracts\Encryption\DecryptException $e) {
            Log::error('PII decryption failed', [
                'error' => $e->getMessage(),
                'trace' => $e->getTraceAsString(),
            ]);
            return null;
        }
    }

    /**
     * Logs access to PII fields for audit compliance.
     *
     * @param int $userId The user whose PII was accessed
     * @param int $accessedBy The user who accessed the PII
     * @param string $fieldAccessed The field that was accessed
     * @param string|null $reason Optional reason for access
     * @return void
     */
    public function logPiiAccess(
        int $userId,
        int $accessedBy,
        string $fieldAccessed,
        ?string $reason = null
    ): void {
        \DB::table('pii_access_logs')->insert([
            'user_id' => $userId,
            'accessed_by' => $accessedBy,
            'field_accessed' => $fieldAccessed,
            'access_reason' => $reason,
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'accessed_at' => now(),
        ]);
    }
}
```

### Model Example

```php
<?php
/**
 * User Model with PII Storage
 *
 * Demonstrates automatic hashing and encryption of PII fields.
 *
 * @package App\Models
 */

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use App\Services\PiiStorageService;

class User extends Model
{
    use SoftDeletes;

    protected $fillable = [
        'email_hash',
        'email_encrypted',
        'phone_hash',
        'phone_encrypted',
        'first_name_encrypted',
        'last_name_encrypted',
        'password',
        'status',
    ];

    protected $hidden = [
        'password',
        'email_encrypted',
        'phone_encrypted',
        'first_name_encrypted',
        'last_name_encrypted',
    ];

    /**
     * Sets the user's email, automatically hashing and encrypting.
     *
     * @param string $email The email address
     * @return void
     */
    public function setEmail(string $email): void
    {
        $pii = app(PiiStorageService::class);
        $this->email_hash = $pii->hashForLookup($email);
        $this->email_encrypted = $pii->encrypt($email);
    }

    /**
     * Gets the user's decrypted email address.
     *
     * @return string|null The decrypted email or null on failure
     */
    public function getEmail(): ?string
    {
        $pii = app(PiiStorageService::class);

        // Log PII access
        $pii->logPiiAccess(
            $this->id,
            auth()->id(),
            'email',
            'Email retrieval'
        );

        return $pii->decrypt($this->email_encrypted);
    }

    /**
     * Finds a user by their email address using the hash.
     *
     * @param string $email The email to search for
     * @return User|null The user or null if not found
     */
    public static function findByEmail(string $email): ?User
    {
        $pii = app(PiiStorageService::class);
        $emailHash = $pii->hashForLookup($email);

        return static::where('email_hash', $emailHash)->first();
    }
}
```

### Migration Example

```php
<?php
/**
 * Create Users Table Migration
 * Demonstrates PII storage schema for MariaDB 12.x
 */

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();

            // Hashed columns for lookup (irreversible)
            $table->string('email_hash', 64)->unique()->comment('HMAC-SHA256 hash for email lookup');
            $table->string('phone_hash', 64)->nullable()->comment('HMAC-SHA256 hash for phone lookup');

            // Encrypted columns for display (reversible with key)
            $table->text('email_encrypted')->comment('AES-256-GCM encrypted email address');
            $table->text('phone_encrypted')->nullable()->comment('AES-256-GCM encrypted phone number');
            $table->text('first_name_encrypted')->comment('AES-256-GCM encrypted first name');
            $table->text('last_name_encrypted')->comment('AES-256-GCM encrypted last name');

            // Non-PII columns
            $table->string('password')->comment('Argon2id hashed password');
            $table->enum('status', ['active', 'suspended', 'deleted'])->default('active');
            $table->timestamps();
            $table->softDeletes();

            // Indices
            $table->index('email_hash');
            $table->index('phone_hash');
            $table->index('status');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

---

## Django/Wagtail (Python/PostgreSQL)

### Service Class

```python
"""
services/pii_storage.py

PII Storage Service for Django 6.x / Wagtail

Handles secure storage of Personally Identifiable Information.
All PII is either hashed (for lookup) or encrypted (for retrieval).
Hashed values are irreversible; encrypted values require the encryption key.

Version: 2.0.0
"""

import hashlib
import hmac
import logging
from typing import Optional
from datetime import datetime

from cryptography.fernet import Fernet, InvalidToken
from django.conf import settings
from django.db import connection

logger = logging.getLogger(__name__)


class PiiStorageService:
    """
    Service for secure PII storage with hashing and encryption.
    Uses HMAC-SHA256 for lookup hashes and Fernet (AES-128-CBC) for reversible encryption.
    """

    def __init__(self):
        """Initialises the Fernet cipher with the application encryption key."""
        self._fernet = Fernet(settings.ENCRYPTION_KEY.encode())

    def hash_for_lookup(self, value: str) -> str:
        """
        Generates an irreversible hash for lookup purposes.
        Used for searching by email/phone without exposing the actual value.

        Args:
            value: The PII value to hash

        Returns:
            str: The hashed value (irreversible, 64 characters hex)
        """
        normalised = value.lower().strip()
        return hmac.new(
            settings.SECRET_KEY.encode(),
            normalised.encode(),
            hashlib.sha256
        ).hexdigest()

    def encrypt(self, value: str) -> str:
        """
        Encrypts PII for secure storage with later retrieval.
        Uses Fernet symmetric encryption (AES-128-CBC with HMAC).
        Only authorised users can decrypt via the application.

        Args:
            value: The PII value to encrypt

        Returns:
            str: The encrypted value (reversible with key)
        """
        return self._fernet.encrypt(value.encode()).decode()

    def decrypt(self, encrypted_value: str) -> Optional[str]:
        """
        Decrypts previously encrypted PII.
        Returns None if decryption fails (tampered data or wrong key).

        Args:
            encrypted_value: The encrypted PII value

        Returns:
            str | None: The decrypted value or None on failure
        """
        try:
            return self._fernet.decrypt(encrypted_value.encode()).decode()
        except InvalidToken:
            logger.error('PII decryption failed: invalid token or tampered data')
            return None
        except Exception as e:
            logger.error(f'PII decryption failed: {str(e)}')
            return None

    def log_pii_access(
        self,
        user_id: int,
        accessed_by: int,
        field_accessed: str,
        access_reason: Optional[str] = None,
        ip_address: Optional[str] = None,
        user_agent: Optional[str] = None
    ) -> None:
        """
        Logs access to PII fields for audit compliance.

        Args:
            user_id: The user whose PII was accessed
            accessed_by: The user who accessed the PII
            field_accessed: The field that was accessed
            access_reason: Optional reason for access
            ip_address: IP address of the accessor
            user_agent: User agent string of the accessor
        """
        with connection.cursor() as cursor:
            cursor.execute(
                """
                INSERT INTO pii_access_logs
                (user_id, accessed_by, field_accessed, access_reason, ip_address, user_agent, accessed_at)
                VALUES (%s, %s, %s, %s, %s, %s, %s)
                """,
                [user_id, accessed_by, field_accessed, access_reason, ip_address, user_agent, datetime.now()]
            )
```

### Model Example

```python
"""
models/user.py

User Model with PII Storage

Demonstrates automatic hashing and encryption of PII fields using Django ORM.
"""

from django.db import models
from django.contrib.auth.models import AbstractBaseUser
from django.utils import timezone
from typing import Optional

from services.pii_storage import PiiStorageService


class User(AbstractBaseUser):
    """
    User model with encrypted PII fields.
    Stores hashed versions for lookup and encrypted versions for display.
    """

    # Hashed columns for lookup (irreversible)
    email_hash = models.CharField(
        max_length=64,
        unique=True,
        db_index=True,
        help_text='HMAC-SHA256 hash for email lookup (irreversible)'
    )
    phone_hash = models.CharField(
        max_length=64,
        null=True,
        blank=True,
        db_index=True,
        help_text='HMAC-SHA256 hash for phone lookup (irreversible)'
    )

    # Encrypted columns for display (reversible with key)
    email_encrypted = models.TextField(
        help_text='Fernet encrypted email address (reversible)'
    )
    phone_encrypted = models.TextField(
        null=True,
        blank=True,
        help_text='Fernet encrypted phone number (reversible)'
    )
    first_name_encrypted = models.TextField(
        help_text='Fernet encrypted first name (reversible)'
    )
    last_name_encrypted = models.TextField(
        help_text='Fernet encrypted last name (reversible)'
    )

    # Non-PII columns
    password = models.CharField(
        max_length=255,
        help_text='Argon2id hashed password (never reversible)'
    )
    status = models.CharField(
        max_length=20,
        choices=[
            ('active', 'Active'),
            ('suspended', 'Suspended'),
            ('deleted', 'Deleted'),
        ],
        default='active',
        db_index=True
    )
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
    updated_at = models.DateTimeField(auto_now=True)
    deleted_at = models.DateTimeField(null=True, blank=True)

    USERNAME_FIELD = 'email_hash'

    class Meta:
        db_table = 'users'
        verbose_name = 'User'
        verbose_name_plural = 'Users'

    def set_email(self, email: str) -> None:
        """
        Sets the user's email, automatically hashing and encrypting.

        Args:
            email: The email address to store
        """
        pii_service = PiiStorageService()
        self.email_hash = pii_service.hash_for_lookup(email)
        self.email_encrypted = pii_service.encrypt(email)

    def get_email(self, accessed_by: Optional[int] = None, reason: Optional[str] = None) -> Optional[str]:
        """
        Gets the user's decrypted email address.
        Logs access for audit compliance.

        Args:
            accessed_by: ID of user accessing the PII (for audit log)
            reason: Reason for accessing PII (for audit log)

        Returns:
            str | None: The decrypted email or None on failure
        """
        pii_service = PiiStorageService()

        # Log PII access if accessor is specified
        if accessed_by:
            pii_service.log_pii_access(
                user_id=self.id,
                accessed_by=accessed_by,
                field_accessed='email',
                access_reason=reason
            )

        return pii_service.decrypt(self.email_encrypted)

    @classmethod
    def find_by_email(cls, email: str) -> Optional['User']:
        """
        Finds a user by their email address using the hash.

        Args:
            email: The email to search for

        Returns:
            User | None: The user or None if not found
        """
        pii_service = PiiStorageService()
        email_hash = pii_service.hash_for_lookup(email)

        try:
            return cls.objects.get(email_hash=email_hash)
        except cls.DoesNotExist:
            return None
```

### Migration Example

```python
"""
migrations/0001_initial.py

Creates users table with PII storage schema for PostgreSQL 18.x
"""

from django.db import migrations, models


class Migration(migrations.Migration):

    initial = True

    dependencies = []

    operations = [
        migrations.CreateModel(
            name='User',
            fields=[
                ('id', models.BigAutoField(primary_key=True, serialize=False)),
                ('email_hash', models.CharField(
                    max_length=64,
                    unique=True,
                    db_index=True,
                    help_text='HMAC-SHA256 hash for email lookup (irreversible)'
                )),
                ('phone_hash', models.CharField(
                    max_length=64,
                    null=True,
                    blank=True,
                    db_index=True,
                    help_text='HMAC-SHA256 hash for phone lookup (irreversible)'
                )),
                ('email_encrypted', models.TextField(
                    help_text='Fernet encrypted email address (reversible)'
                )),
                ('phone_encrypted', models.TextField(
                    null=True,
                    blank=True,
                    help_text='Fernet encrypted phone number (reversible)'
                )),
                ('first_name_encrypted', models.TextField(
                    help_text='Fernet encrypted first name (reversible)'
                )),
                ('last_name_encrypted', models.TextField(
                    help_text='Fernet encrypted last name (reversible)'
                )),
                ('password', models.CharField(
                    max_length=255,
                    help_text='Argon2id hashed password (never reversible)'
                )),
                ('status', models.CharField(
                    max_length=20,
                    choices=[
                        ('active', 'Active'),
                        ('suspended', 'Suspended'),
                        ('deleted', 'Deleted'),
                    ],
                    default='active',
                    db_index=True
                )),
                ('created_at', models.DateTimeField(auto_now_add=True, db_index=True)),
                ('updated_at', models.DateTimeField(auto_now=True)),
                ('deleted_at', models.DateTimeField(null=True, blank=True)),
            ],
            options={
                'db_table': 'users',
                'verbose_name': 'User',
                'verbose_name_plural': 'Users',
            },
        ),
    ]
```

### Required Settings

```python
# config/settings/base.py

# Generate with: python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
ENCRYPTION_KEY = env('ENCRYPTION_KEY')

# Ensure Argon2 is used for password hashing
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
]
```

---

## React/Next.js (TypeScript/Prisma)

```typescript
/**
 * pii-storage.service.ts
 *
 * Handles secure storage of Personally Identifiable Information.
 * All PII is either hashed (for lookup) or encrypted (for retrieval).
 * Hashed values are irreversible; encrypted values require the encryption key.
 */

import { Injectable, Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import * as crypto from 'crypto';

@Injectable()
export class PiiStorageService {
  private readonly logger = new Logger(PiiStorageService.name);
  private readonly algorithm = 'aes-256-gcm';
  private readonly secretKey: Buffer;

  constructor(private configService: ConfigService) {
    const key = this.configService.get<string>('ENCRYPTION_KEY');
    this.secretKey = crypto.scryptSync(key, 'salt', 32);
  }

  /**
   * Generates an irreversible hash for lookup purposes.
   * Used for searching by email/phone without exposing the actual value.
   *
   * @param value - The PII value to hash
   * @returns The hashed value (irreversible)
   */
  hashForLookup(value: string): string {
    const normalised = value.toLowerCase().trim();
    const appKey = this.configService.get<string>('APP_KEY');
    return crypto
      .createHmac('sha256', appKey)
      .update(normalised)
      .digest('hex');
  }

  /**
   * Encrypts PII for secure storage with later retrieval.
   * Only authorised users can decrypt via the application.
   *
   * @param value - The PII value to encrypt
   * @returns The encrypted value (reversible with key)
   */
  encrypt(value: string): string {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(this.algorithm, this.secretKey, iv);

    let encrypted = cipher.update(value, 'utf8', 'hex');
    encrypted += cipher.final('hex');

    const authTag = cipher.getAuthTag();

    // Combine IV, auth tag, and encrypted data
    return `${iv.toString('hex')}:${authTag.toString('hex')}:${encrypted}`;
  }

  /**
   * Decrypts previously encrypted PII.
   * Returns null if decryption fails (tampered or wrong key).
   *
   * @param encryptedValue - The encrypted PII value
   * @returns The decrypted value or null on failure
   */
  decrypt(encryptedValue: string): string | null {
    try {
      const [ivHex, authTagHex, encrypted] = encryptedValue.split(':');
      const iv = Buffer.from(ivHex, 'hex');
      const authTag = Buffer.from(authTagHex, 'hex');

      const decipher = crypto.createDecipheriv(this.algorithm, this.secretKey, iv);
      decipher.setAuthTag(authTag);

      let decrypted = decipher.update(encrypted, 'hex', 'utf8');
      decrypted += decipher.final('utf8');

      return decrypted;
    } catch (error) {
      this.logger.error('PII decryption failed', { error: error.message });
      return null;
    }
  }
}
```

### Required Environment Variables

```bash
# .env
ENCRYPTION_KEY=your-32-character-encryption-key
APP_KEY=your-app-secret-key
```
