# Django Migrations

## Metadata

| Property         | Value         |
| ---------------- | ------------- |
| **Version**      | 2.0.0         |
| **Last Updated** | December 2025 |
| **Status**       | Stable        |

## Framework Versions Tested

| Framework  | Version | Tested Date   |
| ---------- | ------- | ------------- |
| Django     | 6.x     | December 2025 |
| Python     | 3.14    | December 2025 |
| PostgreSQL | 18.x    | December 2025 |

---

## Table of Contents

- [Metadata](#metadata)
- [Framework Versions Tested](#framework-versions-tested)
- [Table of Contents](#table-of-contents)
- [Basic Model and Migration](#basic-model-and-migration)
- [Model with Foreign Keys](#model-with-foreign-keys)
- [PII Model (GDPR Compliant)](#pii-model-gdpr-compliant)
- [Role and Permission Models](#role-and-permission-models)
- [Running Migrations](#running-migrations)
- [Data Migration Example](#data-migration-example)


## Basic Model and Migration

```python
"""
models.py

Defines the User model with standard authentication fields.
Uses UUID for public-facing identifiers and soft deletes
for GDPR-compliant data retention.
"""

import uuid
from django.contrib.auth.models import AbstractBaseUser, PermissionsMixin
from django.db import models
from django.utils import timezone


class User(AbstractBaseUser, PermissionsMixin):
    """
    Custom user model with UUID for public references.

    Attributes:
        uuid: Public-facing unique identifier
        username: Unique username for login
        email: Unique email address
        is_active: Whether the account is active
        is_staff: Whether user can access admin site
        date_joined: When the user registered
        deleted_at: Soft delete timestamp (None = not deleted)
    """

    uuid = models.UUIDField(default=uuid.uuid4, editable=False, unique=True)
    username = models.CharField(max_length=150, unique=True)
    email = models.EmailField(unique=True)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)
    date_joined = models.DateTimeField(default=timezone.now)
    deleted_at = models.DateTimeField(null=True, blank=True)

    USERNAME_FIELD = "email"
    REQUIRED_FIELDS = ["username"]

    class Meta:
        db_table = "users"
        indexes = [
            models.Index(fields=["email"]),
            models.Index(fields=["username"]),
            models.Index(fields=["uuid"]),
        ]

    def soft_delete(self):
        """Marks the user as deleted without removing from database."""
        self.deleted_at = timezone.now()
        self.is_active = False
        self.save(update_fields=["deleted_at", "is_active"])
```

## Model with Foreign Keys

```python
"""
orders/models.py

Defines the Order model with foreign key relationships
to users. Uses UUID for order references.
"""

import uuid
from django.db import models
from django.conf import settings


class OrderStatus(models.TextChoices):
    """Enumeration of order statuses."""
    PENDING = "pending", "Pending"
    PROCESSING = "processing", "Processing"
    SHIPPED = "shipped", "Shipped"
    DELIVERED = "delivered", "Delivered"
    CANCELLED = "cancelled", "Cancelled"


class Order(models.Model):
    """
    Represents a customer order.

    Attributes:
        uuid: Public-facing order reference
        user: The customer who placed the order
        status: Current order status
        total: Order total in pounds (GBP)
        metadata: Additional order data as JSON
        created_at: When the order was placed
        updated_at: When the order was last modified
        deleted_at: Soft delete timestamp
    """

    uuid = models.UUIDField(default=uuid.uuid4, editable=False, unique=True)
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name="orders"
    )
    status = models.CharField(
        max_length=20,
        choices=OrderStatus.choices,
        default=OrderStatus.PENDING
    )
    total = models.DecimalField(max_digits=10, decimal_places=2)
    metadata = models.JSONField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    deleted_at = models.DateTimeField(null=True, blank=True)

    class Meta:
        db_table = "orders"
        ordering = ["-created_at"]
        indexes = [
            models.Index(fields=["user", "status"]),
            models.Index(fields=["created_at"]),
            models.Index(fields=["uuid"]),
        ]

    def __str__(self):
        return f"Order {self.uuid}"
```

## PII Model (GDPR Compliant)

```python
"""
users/models.py

Separate PII model for GDPR compliance.
Stores hashed values for lookups and encrypted values for retrieval.
"""

import hashlib
import hmac
from django.conf import settings
from django.db import models
from cryptography.fernet import Fernet


class UserPii(models.Model):
    """
    Stores user PII separately for GDPR compliance.

    Uses HMAC-SHA256 for lookup hashes (irreversible)
    and Fernet encryption for stored values (reversible).
    """

    user = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name="pii"
    )

    # Hashed columns for lookup (irreversible)
    email_hash = models.CharField(max_length=64, db_index=True)
    phone_hash = models.CharField(max_length=64, null=True, blank=True, db_index=True)

    # Encrypted columns for storage (reversible)
    email_encrypted = models.TextField()
    phone_encrypted = models.TextField(null=True, blank=True)
    full_name_encrypted = models.TextField()
    address_encrypted = models.TextField(null=True, blank=True)
    dob_encrypted = models.TextField(null=True, blank=True)

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        db_table = "user_pii"

    @staticmethod
    def hash_for_lookup(value: str) -> str:
        """
        Creates an HMAC-SHA256 hash for secure lookups.

        Args:
            value: The value to hash (e.g., email address)

        Returns:
            64-character hexadecimal hash string
        """
        key = settings.PII_HASH_KEY.encode()
        return hmac.new(key, value.lower().encode(), hashlib.sha256).hexdigest()

    @staticmethod
    def encrypt(value: str) -> str:
        """
        Encrypts a value using Fernet symmetric encryption.

        Args:
            value: The plaintext value to encrypt

        Returns:
            Base64-encoded encrypted string
        """
        cipher = Fernet(settings.PII_ENCRYPTION_KEY.encode())
        return cipher.encrypt(value.encode()).decode()

    @staticmethod
    def decrypt(encrypted_value: str) -> str:
        """
        Decrypts a Fernet-encrypted value.

        Args:
            encrypted_value: The encrypted string to decrypt

        Returns:
            The original plaintext value
        """
        cipher = Fernet(settings.PII_ENCRYPTION_KEY.encode())
        return cipher.decrypt(encrypted_value.encode()).decode()
```

## Role and Permission Models

```python
"""
accounts/models.py

Role-based access control models for Django.
"""

from django.conf import settings
from django.db import models


class Role(models.Model):
    """
    Defines a role that can be assigned to users.

    Attributes:
        name: Unique role identifier (e.g., 'admin', 'manager')
        display_name: Human-readable role name
        description: Detailed role description
        is_system: Whether this is a system role (cannot be deleted)
    """

    name = models.CharField(max_length=50, unique=True)
    display_name = models.CharField(max_length=100, null=True, blank=True)
    description = models.TextField(null=True, blank=True)
    is_system = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        db_table = "roles"

    def __str__(self):
        return self.display_name or self.name


class Permission(models.Model):
    """
    Defines a granular permission.

    Attributes:
        name: Unique permission identifier (e.g., 'pii.access')
        display_name: Human-readable permission name
        group_name: Category for grouping permissions in UI
    """

    name = models.CharField(max_length=100, unique=True)
    display_name = models.CharField(max_length=100, null=True, blank=True)
    description = models.TextField(null=True, blank=True)
    group_name = models.CharField(max_length=50, null=True, blank=True, db_index=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        db_table = "permissions"

    def __str__(self):
        return self.display_name or self.name


class RolePermission(models.Model):
    """Links roles to permissions (many-to-many)."""

    role = models.ForeignKey(Role, on_delete=models.CASCADE, related_name="role_permissions")
    permission = models.ForeignKey(Permission, on_delete=models.CASCADE, related_name="permission_roles")

    class Meta:
        db_table = "role_permissions"
        unique_together = ("role", "permission")


class UserRole(models.Model):
    """Links users to roles (many-to-many) with metadata."""

    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name="user_roles"
    )
    role = models.ForeignKey(Role, on_delete=models.CASCADE, related_name="role_users")
    assigned_at = models.DateTimeField(auto_now_add=True)
    assigned_by = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name="assigned_roles"
    )

    class Meta:
        db_table = "user_roles"
        unique_together = ("user", "role")
```

## Running Migrations

```bash
# Create migrations for an app
python manage.py makemigrations accounts

# Create migrations for all apps
python manage.py makemigrations

# Show planned SQL without running
python manage.py sqlmigrate accounts 0001

# Run all pending migrations
python manage.py migrate

# Run migrations for specific app
python manage.py migrate accounts

# Rollback to specific migration
python manage.py migrate accounts 0002

# Show migration status
python manage.py showmigrations

# Create empty migration for data migration
python manage.py makemigrations accounts --empty --name populate_roles
```

## Data Migration Example

```python
"""
Data migration to populate initial roles and permissions.
File: accounts/migrations/0002_populate_roles.py
"""

from django.db import migrations


def create_roles_and_permissions(apps, schema_editor):
    """Creates initial roles and permissions."""
    Role = apps.get_model("accounts", "Role")
    Permission = apps.get_model("accounts", "Permission")
    RolePermission = apps.get_model("accounts", "RolePermission")

    # Create permissions
    permissions = [
        Permission(name="pii.access", display_name="Access PII", group_name="PII"),
        Permission(name="pii.export", display_name="Export PII", group_name="PII"),
        Permission(name="pii.delete", display_name="Delete PII", group_name="PII"),
        Permission(name="users.view", display_name="View Users", group_name="Users"),
        Permission(name="users.edit", display_name="Edit Users", group_name="Users"),
    ]
    Permission.objects.bulk_create(permissions)

    # Create roles
    admin_role = Role.objects.create(
        name="admin",
        display_name="Administrator",
        is_system=True
    )

    # Assign all permissions to admin
    pii_access = Permission.objects.get(name="pii.access")
    pii_export = Permission.objects.get(name="pii.export")
    RolePermission.objects.create(role=admin_role, permission=pii_access)
    RolePermission.objects.create(role=admin_role, permission=pii_export)


def reverse_roles_and_permissions(apps, schema_editor):
    """Removes initial roles and permissions."""
    Role = apps.get_model("accounts", "Role")
    Permission = apps.get_model("accounts", "Permission")
    Role.objects.filter(is_system=True).delete()
    Permission.objects.all().delete()


class Migration(migrations.Migration):
    dependencies = [
        ("accounts", "0001_initial"),
    ]

    operations = [
        migrations.RunPython(
            create_roles_and_permissions,
            reverse_roles_and_permissions
        ),
    ]
```
