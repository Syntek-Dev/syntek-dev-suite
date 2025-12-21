# Laravel Migrations

## Metadata

| Property | Value |
|----------|-------|
| **Version** | 2.0.0 |
| **Last Updated** | 12/2025 |
| **Status** | Stable |

## Framework Versions Tested

| Framework | Version | Tested Date |
|-----------|---------|-------------|
| Laravel | 12.x | 12/2025 |
| PHP | 8.4 | 12/2025 |
| MariaDB | 12.x | 12/2025 |

---

## Table of Contents

- [Basic Migration](#basic-migration)
- [Migration with Foreign Keys](#migration-with-foreign-keys)
- [Migration with PII Table (GDPR Compliant)](#migration-with-pii-table-gdpr-compliant)
- [Migration with Pivot Table](#migration-with-pivot-table)
- [Running Migrations](#running-migrations)


## Basic Migration

```php
<?php
/**
 * CreateUsersTable Migration
 *
 * Creates the users table with standard authentication fields.
 * Includes UUID for public-facing identifiers and soft deletes
 * for GDPR-compliant data retention.
 */

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     *
     * Creates the users table with:
     * - Auto-increment ID for internal use
     * - UUID for public-facing references
     * - Standard authentication fields
     * - Soft deletes for data retention
     */
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->string('username')->unique();
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
            $table->softDeletes();
        });
    }

    /**
     * Reverse the migrations.
     *
     * Drops the users table if it exists.
     *
     * @return void
     */
    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

## Migration with Foreign Keys

```php
<?php
/**
 * CreateOrdersTable Migration
 *
 * Creates the orders table with foreign key relationships
 * to users and products. Uses UUID for order references.
 */

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     *
     * Creates the orders table with:
     * - Foreign key relationship to users table
     * - Order status tracking with enum validation
     * - Financial data with precise decimal storage
     * - Flexible metadata storage using JSON
     * - Composite indexes for optimised query performance
     * - Soft deletes for maintaining historical records
     *
     * @return void
     */
    public function up(): void
    {
        Schema::create('orders', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->enum('status', ['pending', 'processing', 'shipped', 'delivered', 'cancelled'])
                  ->default('pending');
            $table->decimal('total', 10, 2);
            $table->json('metadata')->nullable();
            $table->timestamps();
            $table->softDeletes();

            // Indexes for common queries
            $table->index(['user_id', 'status']);
            $table->index('created_at');
        });
    }

    /**
     * Reverse the migrations.
     *
     * Drops the orders table if it exists.
     *
     * @return void
     */
    public function down(): void
    {
        Schema::dropIfExists('orders');
    }
};
```

## Migration with PII Table (GDPR Compliant)

```php
<?php
/**
 * CreateUserPiiTable Migration
 *
 * Creates a separate PII table for GDPR compliance.
 * Stores hashed values for lookups and encrypted values for retrieval.
 */

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     *
     * Creates the user_pii table for GDPR-compliant PII storage with:
     * - One-to-one relationship with users table
     * - Hashed columns (SHA-256) for secure lookup without decryption
     * - Encrypted columns for reversible data storage
     * - Separate storage facilitates data export and deletion requests
     * - Indexed hash columns for efficient query performance
     * - Cascade delete ensures data consistency on user deletion
     *
     * @return void
     */
    public function up(): void
    {
        Schema::create('user_pii', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->unique()->constrained()->cascadeOnDelete();

            // Hashed columns for lookup (irreversible)
            $table->string('email_hash', 64)->index();
            $table->string('phone_hash', 64)->nullable()->index();

            // Encrypted columns for storage (reversible)
            $table->text('email_encrypted');
            $table->text('phone_encrypted')->nullable();
            $table->text('full_name_encrypted');
            $table->text('address_encrypted')->nullable();
            $table->text('dob_encrypted')->nullable();

            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * Drops the user_pii table if it exists.
     *
     * @return void
     */
    public function down(): void
    {
        Schema::dropIfExists('user_pii');
    }
};
```

## Migration with Pivot Table

```php
<?php
/**
 * CreateRolePermissionTables Migration
 *
 * Creates roles, permissions, and pivot tables for RBAC.
 */

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('roles', function (Blueprint $table) {
            $table->id();
            $table->string('name', 50)->unique();
            $table->string('display_name', 100)->nullable();
            $table->text('description')->nullable();
            $table->boolean('is_system')->default(false);
            $table->timestamps();
        });

        Schema::create('permissions', function (Blueprint $table) {
            $table->id();
            $table->string('name', 100)->unique();
            $table->string('display_name', 100)->nullable();
            $table->text('description')->nullable();
            $table->string('group_name', 50)->nullable()->index();
            $table->timestamps();
        });

        Schema::create('role_permissions', function (Blueprint $table) {
            $table->foreignId('role_id')->constrained()->cascadeOnDelete();
            $table->foreignId('permission_id')->constrained()->cascadeOnDelete();
            $table->primary(['role_id', 'permission_id']);
        });

        Schema::create('user_roles', function (Blueprint $table) {
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->foreignId('role_id')->constrained()->cascadeOnDelete();
            $table->timestamp('assigned_at')->useCurrent();
            $table->foreignId('assigned_by')->nullable()->constrained('users')->nullOnDelete();
            $table->primary(['user_id', 'role_id']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('user_roles');
        Schema::dropIfExists('role_permissions');
        Schema::dropIfExists('permissions');
        Schema::dropIfExists('roles');
    }
};
```

## Running Migrations

```bash
# Run all pending migrations
php artisan migrate

# Run migrations for specific environment
php artisan migrate --env=testing

# Rollback last batch
php artisan migrate:rollback

# Rollback all and re-run
php artisan migrate:fresh

# Rollback and re-run with seeders
php artisan migrate:fresh --seed

# Show migration status
php artisan migrate:status

# Generate migration file
php artisan make:migration create_products_table
php artisan make:migration add_status_to_orders_table
```
