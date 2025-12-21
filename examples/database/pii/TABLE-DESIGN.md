# PII Table Design

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
| PHP | 8.4 | 20/12/2025 |
| MariaDB | 12.x | 20/12/2025 |
| Django | 6.x | 20/12/2025 |
| Python | 3.14 | 20/12/2025 |
| PostgreSQL | 18.x | 20/12/2025 |
| Next.js | 16.x | 20/12/2025 |
| Node.js | 24.x | 20/12/2025 |
| TypeScript | 5.9 | 20/12/2025 |
| Prisma | 6.x | 20/12/2025 |

---

## Table of Contents

- [Overview](#overview)
- [Schema Design Principles](#schema-design-principles)
- [Stack 1: TALL Stack (Laravel/MariaDB)](#stack-1-tall-stack-laravelmariadb)
- [Stack 2: Django/Wagtail (PostgreSQL)](#stack-2-djangowagtail-postgresql)
- [Stack 3: React/Next.js (Prisma/PostgreSQL)](#stack-3-reactnextjs-prismapostgresql)
- [User Model with PII Protection](#user-model-with-pii-protection)


## Overview

Store PII in a separate table from the main user record with:
- **Hashed columns** for lookup (indexed, unique where needed)
- **Encrypted columns** for storage (retrievable with key)
- **Foreign key** to user table with CASCADE delete

This design ensures:
1. PII data is isolated from the main user table
2. Lookups can be performed without decrypting data
3. Encrypted data can be retrieved when authorised
4. GDPR compliance through easy deletion (CASCADE)

## Schema Design Principles

| Column Type | Purpose | Example |
|-------------|---------|---------|
| `*_hash` | Indexed lookup (irreversible) | `email_hash VARCHAR(64)` |
| `*_encrypted` | Secure storage (reversible) | `email_encrypted TEXT` |
| `user_id` | Foreign key to users table | `BIGINT` with CASCADE |

### PII Column Patterns

**For each PII field, create TWO columns:**

1. **`{field}_hash`** - SHA-256 hash for lookup
   - Used for: Finding records, preventing duplicates
   - Indexed: Yes (unique where appropriate)
   - Searchable: Yes (by hashing search term first)
   - Retrievable: No (irreversible)

2. **`{field}_encrypted`** - AES-256 encrypted value for display
   - Used for: Displaying actual PII data when authorised
   - Indexed: No
   - Searchable: No
   - Retrievable: Yes (with encryption key)

---

## Stack 1: TALL Stack (Laravel/MariaDB)

### MariaDB Schema

```sql
-- MariaDB 12.x schema for user_pii table
-- This schema is optimised for MariaDB's InnoDB storage engine
-- with support for encrypted tablespaces

-- Drop table if exists (development only)
-- DROP TABLE IF EXISTS user_pii;

-- Create user_pii table with InnoDB engine
CREATE TABLE user_pii (
    -- Primary key using BIGINT UNSIGNED for large-scale applications
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,

    -- Foreign key to users table with CASCADE delete for GDPR compliance
    user_id BIGINT UNSIGNED NOT NULL,

    -- Hashed columns for lookup (irreversible)
    -- SHA-256 produces 64 character hex string
    -- These columns are indexed for fast lookups without exposing PII
    email_hash VARCHAR(64) NOT NULL COMMENT 'SHA-256 hash of email for lookup',
    phone_hash VARCHAR(64) NULL COMMENT 'SHA-256 hash of phone for lookup',

    -- Encrypted columns for storage (reversible with encryption key)
    -- Using TEXT to accommodate encrypted data which is larger than plaintext
    -- AES-256 encryption produces base64 encoded strings
    email_encrypted TEXT NOT NULL COMMENT 'AES-256 encrypted email address',
    phone_encrypted TEXT NULL COMMENT 'AES-256 encrypted phone number',
    full_name_encrypted TEXT NOT NULL COMMENT 'AES-256 encrypted full name',
    address_encrypted TEXT NULL COMMENT 'AES-256 encrypted postal address',
    dob_encrypted TEXT NULL COMMENT 'AES-256 encrypted date of birth',

    -- Audit timestamps
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT 'Record creation timestamp',
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Record last update timestamp',

    -- Primary key constraint
    PRIMARY KEY (id),

    -- Unique constraint on email_hash to prevent duplicate registrations
    -- This allows email uniqueness checks without decrypting emails
    UNIQUE KEY unique_email_hash (email_hash),

    -- Index on phone_hash for lookup queries
    KEY idx_phone_hash (phone_hash),

    -- Foreign key constraint with CASCADE delete
    -- When a user is deleted, their PII is automatically removed
    CONSTRAINT fk_user_pii_user_id
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci
  COMMENT='Stores encrypted PII data with hashed lookup columns';

-- Create index on user_id for efficient joins
CREATE INDEX idx_user_id ON user_pii(user_id);
```

### Laravel Migration

```php
<?php
/**
 * Creates the user_pii table for secure PII storage.
 * All PII is stored encrypted with hashed lookup columns.
 *
 * Laravel 12.x / PHP 8.4 / MariaDB 12.x
 *
 * This migration creates a dedicated PII table that stores:
 * - Hashed columns for fast lookups without decryption
 * - Encrypted columns for secure PII storage
 * - Foreign key with CASCADE delete for GDPR compliance
 */

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     *
     * Creates the user_pii table with proper indexing and constraints.
     */
    public function up(): void
    {
        Schema::create('user_pii', function (Blueprint $table) {
            // Primary key
            $table->id();

            // Foreign key to users table with CASCADE delete
            // When a user is deleted, their PII is automatically removed
            $table->foreignId('user_id')
                ->constrained('users')
                ->onDelete('cascade')
                ->onUpdate('cascade');

            // Hashed columns for lookup (irreversible)
            // SHA-256 produces 64 character hex strings
            // These are indexed for fast lookups without exposing PII
            $table->string('email_hash', 64)
                ->index()
                ->comment('SHA-256 hash of email for lookup');

            $table->string('phone_hash', 64)
                ->nullable()
                ->index()
                ->comment('SHA-256 hash of phone for lookup');

            // Encrypted columns for storage (reversible with encryption key)
            // AES-256 encrypted values stored as TEXT
            // These should NEVER be indexed or used in WHERE clauses
            $table->text('email_encrypted')
                ->comment('AES-256 encrypted email address');

            $table->text('phone_encrypted')
                ->nullable()
                ->comment('AES-256 encrypted phone number');

            $table->text('full_name_encrypted')
                ->comment('AES-256 encrypted full name');

            $table->text('address_encrypted')
                ->nullable()
                ->comment('AES-256 encrypted postal address');

            $table->text('dob_encrypted')
                ->nullable()
                ->comment('AES-256 encrypted date of birth');

            // Audit timestamps
            $table->timestamps();

            // Unique constraint on email_hash to prevent duplicate registrations
            // This allows email uniqueness checks without decrypting emails
            $table->unique('email_hash', 'unique_email_hash');
        });

        // Add comment to table (MariaDB specific)
        DB::statement("ALTER TABLE user_pii COMMENT 'Stores encrypted PII data with hashed lookup columns'");
    }

    /**
     * Reverse the migrations.
     *
     * Drops the user_pii table.
     */
    public function down(): void
    {
        Schema::dropIfExists('user_pii');
    }
};
```

### Laravel Model

```php
<?php
/**
 * UserPii Model
 *
 * Represents the user_pii table for secure PII storage.
 * All PII data is encrypted at rest with hashed lookup columns.
 *
 * Laravel 12.x / PHP 8.4
 */

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class UserPii extends Model
{
    /**
     * The table associated with the model.
     */
    protected $table = 'user_pii';

    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'user_id',
        'email_hash',
        'phone_hash',
        'email_encrypted',
        'phone_encrypted',
        'full_name_encrypted',
        'address_encrypted',
        'dob_encrypted',
    ];

    /**
     * The attributes that should be hidden for serialisation.
     *
     * @var array<int, string>
     */
    protected $hidden = [
        'email_encrypted',
        'phone_encrypted',
        'full_name_encrypted',
        'address_encrypted',
        'dob_encrypted',
    ];

    /**
     * Get the user that owns the PII record.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Scope: Find by email hash.
     *
     * @param \Illuminate\Database\Eloquent\Builder $query
     * @param string $emailHash
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeByEmailHash($query, string $emailHash)
    {
        return $query->where('email_hash', $emailHash);
    }

    /**
     * Scope: Find by phone hash.
     *
     * @param \Illuminate\Database\Eloquent\Builder $query
     * @param string $phoneHash
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeByPhoneHash($query, string $phoneHash)
    {
        return $query->where('phone_hash', $phoneHash);
    }
}
```

---

## Stack 2: Django/Wagtail (PostgreSQL)

### PostgreSQL Schema

```sql
-- PostgreSQL 18.x schema for user_pii table
-- This schema is optimised for PostgreSQL with GDPR compliance features

-- Drop table if exists (development only)
-- DROP TABLE IF EXISTS user_pii CASCADE;

-- Create user_pii table
CREATE TABLE user_pii (
    -- Primary key using BIGSERIAL for auto-incrementing 64-bit integers
    id BIGSERIAL PRIMARY KEY,

    -- Foreign key to auth_user table with CASCADE delete for GDPR compliance
    -- Adjust the referenced table name based on your Django user model
    user_id BIGINT NOT NULL,

    -- Hashed columns for lookup (irreversible)
    -- SHA-256 produces 64 character hex string
    -- These columns are indexed for fast lookups without exposing PII
    email_hash VARCHAR(64) NOT NULL,
    phone_hash VARCHAR(64) NULL,

    -- Encrypted columns for storage (reversible with encryption key)
    -- Using TEXT to accommodate encrypted data which is larger than plaintext
    -- AES-256 encryption produces base64 encoded strings
    email_encrypted TEXT NOT NULL,
    phone_encrypted TEXT NULL,
    full_name_encrypted TEXT NOT NULL,
    address_encrypted TEXT NULL,
    dob_encrypted TEXT NULL,

    -- Audit timestamps
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP

);

-- Add table comment
COMMENT ON TABLE user_pii IS 'Stores encrypted PII data with hashed lookup columns';

-- Add column comments
COMMENT ON COLUMN user_pii.id IS 'Primary key';
COMMENT ON COLUMN user_pii.user_id IS 'Foreign key to auth_user table';
COMMENT ON COLUMN user_pii.email_hash IS 'SHA-256 hash of email for lookup';
COMMENT ON COLUMN user_pii.phone_hash IS 'SHA-256 hash of phone for lookup';
COMMENT ON COLUMN user_pii.email_encrypted IS 'AES-256 encrypted email address';
COMMENT ON COLUMN user_pii.phone_encrypted IS 'AES-256 encrypted phone number';
COMMENT ON COLUMN user_pii.full_name_encrypted IS 'AES-256 encrypted full name';
COMMENT ON COLUMN user_pii.address_encrypted IS 'AES-256 encrypted postal address';
COMMENT ON COLUMN user_pii.dob_encrypted IS 'AES-256 encrypted date of birth';
COMMENT ON COLUMN user_pii.created_at IS 'Record creation timestamp';
COMMENT ON COLUMN user_pii.updated_at IS 'Record last update timestamp';

-- Create unique constraint on email_hash to prevent duplicate registrations
-- This allows email uniqueness checks without decrypting emails
CREATE UNIQUE INDEX unique_email_hash ON user_pii(email_hash);

-- Create index on phone_hash for lookup queries
CREATE INDEX idx_phone_hash ON user_pii(phone_hash);

-- Create index on user_id for efficient joins
CREATE INDEX idx_user_id ON user_pii(user_id);

-- Add foreign key constraint with CASCADE delete
-- When a user is deleted, their PII is automatically removed
-- Adjust the referenced table name based on your Django user model
ALTER TABLE user_pii
    ADD CONSTRAINT fk_user_pii_user_id
    FOREIGN KEY (user_id)
    REFERENCES auth_user(id)
    ON DELETE CASCADE
    ON UPDATE CASCADE;

-- Create trigger to automatically update updated_at timestamp
CREATE OR REPLACE FUNCTION update_user_pii_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_user_pii_updated_at
    BEFORE UPDATE ON user_pii
    FOR EACH ROW
    EXECUTE FUNCTION update_user_pii_updated_at();
```

### Django Migration

```python
"""
migrations/0001_create_user_pii.py

Creates the UserPii model for secure PII storage.
All PII is stored encrypted with hashed lookup columns.

Django 6.x / Python 3.14 / PostgreSQL 18.x

This migration creates a dedicated PII table that stores:
- Hashed columns for fast lookups without decryption
- Encrypted columns for secure PII storage
- Foreign key with CASCADE delete for GDPR compliance
"""

from django.db import migrations, models
import django.db.models.deletion


class Migration(migrations.Migration):
    """Creates the user_pii table for secure PII storage."""

    initial = True

    dependencies = [
        ('accounts', '0001_initial'),
    ]

    operations = [
        migrations.CreateModel(
            name='UserPii',
            fields=[
                # Primary key using BigAutoField
                ('id', models.BigAutoField(
                    auto_created=True,
                    primary_key=True,
                    serialize=False,
                    verbose_name='ID'
                )),

                # Foreign key to User model with CASCADE delete
                # When a user is deleted, their PII is automatically removed
                ('user', models.OneToOneField(
                    on_delete=django.db.models.deletion.CASCADE,
                    related_name='pii',
                    to='accounts.user',
                    verbose_name='User'
                )),

                # Hashed columns for lookup (irreversible)
                # SHA-256 produces 64 character hex strings
                # These are indexed for fast lookups without exposing PII
                ('email_hash', models.CharField(
                    max_length=64,
                    unique=True,
                    db_index=True,
                    verbose_name='Email Hash',
                    help_text='SHA-256 hash of email for lookup'
                )),
                ('phone_hash', models.CharField(
                    max_length=64,
                    null=True,
                    blank=True,
                    db_index=True,
                    verbose_name='Phone Hash',
                    help_text='SHA-256 hash of phone for lookup'
                )),

                # Encrypted columns for storage (reversible with key)
                # AES-256 encrypted values stored as TEXT
                # These should NEVER be indexed or used in WHERE clauses
                ('email_encrypted', models.TextField(
                    verbose_name='Encrypted Email',
                    help_text='AES-256 encrypted email address'
                )),
                ('phone_encrypted', models.TextField(
                    null=True,
                    blank=True,
                    verbose_name='Encrypted Phone',
                    help_text='AES-256 encrypted phone number'
                )),
                ('full_name_encrypted', models.TextField(
                    verbose_name='Encrypted Full Name',
                    help_text='AES-256 encrypted full name'
                )),
                ('address_encrypted', models.TextField(
                    null=True,
                    blank=True,
                    verbose_name='Encrypted Address',
                    help_text='AES-256 encrypted postal address'
                )),
                ('dob_encrypted', models.TextField(
                    null=True,
                    blank=True,
                    verbose_name='Encrypted Date of Birth',
                    help_text='AES-256 encrypted date of birth'
                )),

                # Audit timestamps
                ('created_at', models.DateTimeField(
                    auto_now_add=True,
                    verbose_name='Created At'
                )),
                ('updated_at', models.DateTimeField(
                    auto_now=True,
                    verbose_name='Updated At'
                )),
            ],
            options={
                'db_table': 'user_pii',
                'verbose_name': 'User PII',
                'verbose_name_plural': 'User PII Records',
                'ordering': ['-created_at'],
                'indexes': [
                    models.Index(fields=['email_hash'], name='idx_email_hash'),
                    models.Index(fields=['phone_hash'], name='idx_phone_hash'),
                ],
            },
        ),
    ]
```

### Django Model

```python
"""
models/user_pii.py

UserPii model for secure PII storage with hashed lookup columns.

Django 6.x / Python 3.14 / PostgreSQL 18.x

This model represents encrypted PII data with:
- Hashed columns for fast lookups without decryption
- Encrypted columns for secure PII storage
- One-to-one relationship with User model
"""

from django.db import models
from django.conf import settings


class UserPii(models.Model):
    """
    Stores encrypted PII for users with hashed lookup columns.

    This model uses a dual-column approach for each PII field:
    - {field}_hash: SHA-256 hash for lookups (irreversible)
    - {field}_encrypted: AES-256 encrypted value (reversible with key)

    Related User model via OneToOneField with CASCADE delete.
    """

    # Foreign key to User model with CASCADE delete
    user = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='pii',
        verbose_name='User'
    )

    # Hashed columns for lookup (irreversible)
    # SHA-256 produces 64 character hex strings
    email_hash = models.CharField(
        max_length=64,
        unique=True,
        db_index=True,
        verbose_name='Email Hash',
        help_text='SHA-256 hash of email for lookup'
    )
    phone_hash = models.CharField(
        max_length=64,
        null=True,
        blank=True,
        db_index=True,
        verbose_name='Phone Hash',
        help_text='SHA-256 hash of phone for lookup'
    )

    # Encrypted columns for storage (reversible with key)
    # AES-256 encrypted values stored as TEXT
    email_encrypted = models.TextField(
        verbose_name='Encrypted Email',
        help_text='AES-256 encrypted email address'
    )
    phone_encrypted = models.TextField(
        null=True,
        blank=True,
        verbose_name='Encrypted Phone',
        help_text='AES-256 encrypted phone number'
    )
    full_name_encrypted = models.TextField(
        verbose_name='Encrypted Full Name',
        help_text='AES-256 encrypted full name'
    )
    address_encrypted = models.TextField(
        null=True,
        blank=True,
        verbose_name='Encrypted Address',
        help_text='AES-256 encrypted postal address'
    )
    dob_encrypted = models.TextField(
        null=True,
        blank=True,
        verbose_name='Encrypted Date of Birth',
        help_text='AES-256 encrypted date of birth'
    )

    # Audit timestamps
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name='Created At'
    )
    updated_at = models.DateTimeField(
        auto_now=True,
        verbose_name='Updated At'
    )

    class Meta:
        db_table = 'user_pii'
        verbose_name = 'User PII'
        verbose_name_plural = 'User PII Records'
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['email_hash'], name='idx_email_hash'),
            models.Index(fields=['phone_hash'], name='idx_phone_hash'),
        ]

    def __str__(self) -> str:
        """String representation showing user ID (never expose PII)."""
        return f'PII for User {self.user_id}'

    @classmethod
    def find_by_email_hash(cls, email_hash: str):
        """
        Find a UserPii record by email hash.

        Args:
            email_hash: SHA-256 hash of the email to search for

        Returns:
            UserPii instance or None
        """
        try:
            return cls.objects.select_related('user').get(email_hash=email_hash)
        except cls.DoesNotExist:
            return None

    @classmethod
    def find_by_phone_hash(cls, phone_hash: str):
        """
        Find a UserPii record by phone hash.

        Args:
            phone_hash: SHA-256 hash of the phone to search for

        Returns:
            UserPii instance or None
        """
        try:
            return cls.objects.select_related('user').get(phone_hash=phone_hash)
        except cls.DoesNotExist:
            return None
```

---

## Stack 3: React/Next.js (Prisma/PostgreSQL)

### Prisma Schema

```typescript
/**
 * 1234567890-CreateUserPii.ts
 *
 * Creates the user_pii table for secure PII storage.
 * All PII is stored encrypted with hashed lookup columns.
 */

import { MigrationInterface, QueryRunner, Table, TableIndex, TableForeignKey } from 'typeorm';

export class CreateUserPii1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: 'user_pii',
        columns: [
          {
            name: 'id',
            type: 'bigint',
            isPrimary: true,
            isGenerated: true,
            generationStrategy: 'increment',
          },
          {
            name: 'user_id',
            type: 'bigint',
          },
          // Hashed columns for lookup (irreversible)
          {
            name: 'email_hash',
            type: 'varchar',
            length: '64',
          },
          {
            name: 'phone_hash',
            type: 'varchar',
            length: '64',
            isNullable: true,
          },
          // Encrypted columns for storage (reversible with key)
          {
            name: 'email_encrypted',
            type: 'text',
          },
          {
            name: 'phone_encrypted',
            type: 'text',
            isNullable: true,
          },
          {
            name: 'full_name_encrypted',
            type: 'text',
          },
          {
            name: 'address_encrypted',
            type: 'text',
            isNullable: true,
          },
          {
            name: 'dob_encrypted',
            type: 'text',
            isNullable: true,
          },
          {
            name: 'created_at',
            type: 'timestamp',
            default: 'CURRENT_TIMESTAMP',
          },
          {
            name: 'updated_at',
            type: 'timestamp',
            default: 'CURRENT_TIMESTAMP',
            onUpdate: 'CURRENT_TIMESTAMP',
          },
        ],
      }),
      true,
    );

    // Add unique constraint on email_hash
    await queryRunner.createIndex(
      'user_pii',
      new TableIndex({
        name: 'IDX_USER_PII_EMAIL_HASH_UNIQUE',
        columnNames: ['email_hash'],
        isUnique: true,
      }),
    );

    // Add index on phone_hash
    await queryRunner.createIndex(
      'user_pii',
      new TableIndex({
        name: 'IDX_USER_PII_PHONE_HASH',
        columnNames: ['phone_hash'],
      }),
    );

    // Add foreign key to users table
    await queryRunner.createForeignKey(
      'user_pii',
      new TableForeignKey({
        columnNames: ['user_id'],
        referencedColumnNames: ['id'],
        referencedTableName: 'users',
        onDelete: 'CASCADE',
      }),
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('user_pii');
  }
}
```

### TypeORM Entity

```typescript
/**
 * user-pii.entity.ts
 *
 * UserPii entity for secure PII storage.
 */

import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  OneToOne,
  JoinColumn,
  CreateDateColumn,
  UpdateDateColumn,
} from 'typeorm';
import { User } from './user.entity';

@Entity('user_pii')
export class UserPii {
  @PrimaryGeneratedColumn('increment', { type: 'bigint' })
  id: number;

  @OneToOne(() => User, (user) => user.pii, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'user_id' })
  user: User;

  // Hashed columns for lookup (irreversible)
  @Column({ name: 'email_hash', length: 64, unique: true })
  emailHash: string;

  @Column({ name: 'phone_hash', length: 64, nullable: true })
  phoneHash: string | null;

  // Encrypted columns for storage (reversible with key)
  @Column({ name: 'email_encrypted', type: 'text' })
  emailEncrypted: string;

  @Column({ name: 'phone_encrypted', type: 'text', nullable: true })
  phoneEncrypted: string | null;

  @Column({ name: 'full_name_encrypted', type: 'text' })
  fullNameEncrypted: string;

  @Column({ name: 'address_encrypted', type: 'text', nullable: true })
  addressEncrypted: string | null;

  @Column({ name: 'dob_encrypted', type: 'text', nullable: true })
  dobEncrypted: string | null;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;
}
```

---

## User Model with PII Protection

### Laravel

```php
<?php

namespace App\Models;

use App\Services\PiiStorageService;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    public function pii()
    {
        return $this->hasOne(UserPii::class);
    }

    /**
     * Finds a user by email using the hashed lookup column.
     */
    public static function findByEmail(string $email): ?User
    {
        $hash = app(PiiStorageService::class)->hashForLookup($email);
        $pii = UserPii::where('email_hash', $hash)->first();
        return $pii?->user;
    }
}
```

### Django

```python
from services.pii_storage import PiiStorageService

class User(AbstractUser):
    @classmethod
    def find_by_email(cls, email: str) -> Optional['User']:
        """Finds a user by email using the hashed lookup column."""
        email_hash = PiiStorageService().hash_for_lookup(email)
        try:
            pii = UserPii.objects.select_related('user').get(email_hash=email_hash)
            return pii.user
        except UserPii.DoesNotExist:
            return None
```

### TypeScript

```typescript
static async findByEmail(
  email: string,
  piiService: PiiStorageService,
  userPiiRepository: Repository<UserPii>,
): Promise<User | null> {
  const emailHash = piiService.hashForLookup(email);
  const pii = await userPiiRepository.findOne({
    where: { emailHash },
    relations: ['user'],
  });
  return pii?.user ?? null;
}
```
