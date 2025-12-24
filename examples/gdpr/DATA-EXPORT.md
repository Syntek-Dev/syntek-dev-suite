# GDPR Data Export (DSAR)

## Overview

Implementation patterns for GDPR data subject rights including Right to Access (Article 15), Right to Erasure (Article 17), and consent management.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
| **Laravel** | 12.x |
| **PHP** | 8.4 |
| **MariaDB** | 12.x |
| **Django** | 6.x |
| **Python** | 3.14 |
| **PostgreSQL** | 18.x |
| **Next.js** | 16.x |
| **Node.js** | 24.x |
| **TypeScript** | 5.9 |
| **Prisma** | 6.x |
| **React Native** | 0.83.x |
| **Stacks** | TALL, Django/Wagtail, React/Next.js, React Native |

---

## Table of Contents

- [GDPR Data Export (DSAR)](#gdpr-data-export-dsar)
  - [Overview](#overview)
  - [Metadata](#metadata)
  - [Table of Contents](#table-of-contents)
  - [Data Subject Rights](#data-subject-rights)
  - [TALL Stack (Laravel 12.x)](#tall-stack-laravel-12x)
    - [GDPR Controller](#gdpr-controller)
      - [app/Http/Controllers/GdprController.php](#apphttpcontrollersgdprcontrollerphp)
    - [Data Export Service](#data-export-service)
      - [app/Services/GdprDataExportService.php](#appservicesgdprdataexportservicephp)
    - [Consent Management](#consent-management)
      - [app/Models/UserConsent.php](#appmodelsuserconsentphp)
  - [Django/Wagtail (Django 6.x)](#djangowagtail-django-6x)
    - [GDPR Views](#gdpr-views)
      - [apps/gdpr/views.py](#appsgdprviewspy)
    - [Data Export Service](#data-export-service-1)
      - [apps/gdpr/services/gdpr\_export.py](#appsgdprservicesgdpr_exportpy)
  - [Consent Management - Laravel](#consent-management---laravel)
    - [app/Models/UserConsent.php](#appmodelsuserconsentphp-1)
  - [Audit Trail Requirements](#audit-trail-requirements)
    - [Audit Log Migration](#audit-log-migration)



## Data Subject Rights

| Right | Article | Implementation |
|-------|---------|----------------|
| Access | 15 | Data export endpoint |
| Rectification | 16 | Profile update endpoints |
| Erasure | 17 | Account deletion with anonymisation |
| Portability | 20 | Machine-readable export (JSON/CSV) |
| Object | 21 | Marketing opt-out preferences |

---

## TALL Stack (Laravel 12.x)

### GDPR Controller

#### app/Http/Controllers/GdprController.php

```php
<?php

/**
 * GdprController.php
 *
 * Handles GDPR data subject requests including data export,
 * account deletion, and consent management.
 *
 * @package App\Http\Controllers
 * @version 2.0.0
 */

namespace App\Http\Controllers;

use App\Services\GdprDataExportService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;

class GdprController extends Controller
{
    /**
     * Initialises the GDPR controller with required services.
     *
     * @param GdprDataExportService $exportService Service for handling data exports
     */
    public function __construct(
        protected GdprDataExportService $exportService
    ) {}

    /**
     * Exports all personal data for the authenticated user.
     *
     * Implements GDPR Article 15 Right to Access. Returns all user data
     * in machine-readable JSON format with metadata about the export.
     * Logs the export activity for compliance audit trail.
     *
     * @param Request $request The HTTP request containing authenticated user
     * @return JsonResponse Complete user data export in JSON format
     */
    public function exportData(Request $request): JsonResponse
    {
        $user = $request->user();

        // Generate complete data export
        $data = $this->exportService->exportUserData($user);

        // Log the export for audit trail compliance
        DB::table('gdpr_audit_log')->insert([
            'user_id' => $user->id,
            'action' => 'data_export',
            'ip_hash' => hash_hmac('sha256', $request->ip(), config('app.key')),
            'created_at' => now(),
        ]);

        return response()->json([
            'export_date' => now()->toIso8601String(),
            'export_type' => 'gdpr_article_15',
            'format_version' => '2.0.0',
            'data' => $data,
        ]);
    }

    /**
     * Initiates account deletion request.
     *
     * Implements GDPR Article 17 Right to Erasure. Anonymises data that
     * must be retained for legal/financial reasons whilst deleting personal data.
     * Requires password confirmation to prevent accidental deletion.
     *
     * @param Request $request The HTTP request with password and confirmation
     * @return JsonResponse Deletion confirmation message
     */
    public function deleteAccount(Request $request): JsonResponse
    {
        $request->validate([
            'password' => 'required|string',
            'confirmation' => 'required|in:DELETE',
        ]);

        $user = $request->user();

        // Verify password for security
        if (!Hash::check($request->password, $user->password)) {
            return response()->json(['error' => 'Invalid password'], 403);
        }

        // Store original user ID hash before anonymisation
        $originalUserHash = hash('sha256', $user->id . config('app.key'));

        // Anonymise data that must be retained
        $this->exportService->anonymiseUser($user);

        // Log the deletion for audit trail
        DB::table('gdpr_audit_log')->insert([
            'user_id' => null, // User is now anonymised
            'action' => 'account_deletion',
            'metadata' => json_encode(['original_user_hash' => $originalUserHash]),
            'ip_hash' => hash_hmac('sha256', $request->ip(), config('app.key')),
            'created_at' => now(),
        ]);

        return response()->json([
            'message' => 'Account deletion initiated. Personal data will be removed within 30 days.',
            'deletion_reference' => $originalUserHash,
        ]);
    }

    /**
     * Updates user consent preferences.
     *
     * Implements GDPR Article 7 consent requirements. Records consent
     * changes with full audit trail including IP address and timestamp.
     *
     * @param Request $request The HTTP request with consent preferences
     * @return JsonResponse Updated consent preferences
     */
    public function updateConsent(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'consents' => 'required|array',
            'consents.*.type' => 'required|string|in:marketing_email,marketing_sms,analytics,third_party_sharing,profiling',
            'consents.*.granted' => 'required|boolean',
        ]);

        $user = $request->user();

        foreach ($validated['consents'] as $consent) {
            $user->consents()->updateOrCreate(
                ['type' => $consent['type']],
                [
                    'granted' => $consent['granted'],
                    'ip_hash' => hash_hmac('sha256', $request->ip(), config('app.key')),
                    'user_agent' => $request->userAgent(),
                ]
            );

            // Log consent change
            DB::table('gdpr_audit_log')->insert([
                'user_id' => $user->id,
                'action' => 'consent_change',
                'metadata' => json_encode([
                    'type' => $consent['type'],
                    'granted' => $consent['granted'],
                ]),
                'ip_hash' => hash_hmac('sha256', $request->ip(), config('app.key')),
                'created_at' => now(),
            ]);
        }

        return response()->json([
            'message' => 'Consent preferences updated successfully',
            'consents' => $user->consents()->get(),
        ]);
    }
}
```

### Data Export Service

#### app/Services/GdprDataExportService.php

```php
<?php

/**
 * GdprDataExportService.php
 *
 * Handles GDPR data export and anonymisation operations.
 * Exports all personal data and anonymises data that must be retained.
 *
 * @package App\Services
 * @version 2.0.0
 */

namespace App\Services;

use App\Models\User;
use App\Services\PiiStorageService;

class GdprDataExportService
{
    /**
     * Initialises the GDPR data export service.
     *
     * @param PiiStorageService $piiService Service for PII encryption/decryption
     */
    public function __construct(
        protected PiiStorageService $piiService
    ) {}

    /**
     * Exports all personal data for a user.
     *
     * Decrypts all PII for the export as the user has right to access
     * their own data under GDPR Article 15. Includes account data,
     * activity history, orders, communications, and consents.
     *
     * @param User $user The user to export data for
     * @return array Complete user data export with all personal information
     */
    public function exportUserData(User $user): array
    {
        return [
            'personal_information' => $this->exportPersonalInformation($user),
            'account_data' => $this->exportAccountData($user),
            'activity_history' => $this->exportActivityHistory($user),
            'orders' => $this->exportOrders($user),
            'communications' => $this->exportCommunications($user),
            'consents' => $this->exportConsents($user),
        ];
    }

    /**
     * Exports personal information including encrypted PII.
     *
     * @param User $user The user to export data for
     * @return array Personal information including email, name, phone, address
     */
    protected function exportPersonalInformation(User $user): array
    {
        return [
            'email' => $this->piiService->decrypt($user->email_encrypted),
            'full_name' => $this->piiService->decrypt($user->full_name_encrypted),
            'phone' => $user->phone_encrypted
                ? $this->piiService->decrypt($user->phone_encrypted)
                : null,
            'address' => $user->address_encrypted
                ? $this->piiService->decrypt($user->address_encrypted)
                : null,
            'date_of_birth' => $user->dob_encrypted
                ? $this->piiService->decrypt($user->dob_encrypted)
                : null,
        ];
    }

    /**
     * Exports account metadata and status information.
     *
     * @param User $user The user to export data for
     * @return array Account creation, login, and status information
     */
    protected function exportAccountData(User $user): array
    {
        return [
            'created_at' => $user->created_at->toIso8601String(),
            'last_login' => $user->last_login_at?->toIso8601String(),
            'account_status' => $user->status,
            'email_verified_at' => $user->email_verified_at?->toIso8601String(),
        ];
    }

    /**
     * Exports user activity history.
     *
     * @param User $user The user to export data for
     * @return array Activity log entries
     */
    protected function exportActivityHistory(User $user): array
    {
        return $user->activities()
            ->select('action', 'created_at', 'metadata')
            ->orderBy('created_at', 'desc')
            ->get()
            ->toArray();
    }

    /**
     * Exports order history.
     *
     * @param User $user The user to export data for
     * @return array Order information
     */
    protected function exportOrders(User $user): array
    {
        return $user->orders()
            ->select('id', 'total', 'status', 'created_at', 'updated_at')
            ->with('items:id,order_id,product_name,quantity,price')
            ->get()
            ->toArray();
    }

    /**
     * Exports communications history.
     *
     * @param User $user The user to export data for
     * @return array Messages and communications
     */
    protected function exportCommunications(User $user): array
    {
        return $user->messages()
            ->select('subject', 'body', 'created_at', 'direction')
            ->get()
            ->toArray();
    }

    /**
     * Exports consent preferences.
     *
     * @param User $user The user to export data for
     * @return array Consent records with granted status
     */
    protected function exportConsents(User $user): array
    {
        return $user->consents()
            ->select('type', 'granted', 'updated_at')
            ->get()
            ->toArray();
    }

    /**
     * Anonymises a user account for data retention.
     *
     * Replaces personal data with anonymised values whilst retaining
     * transaction history for legal/financial requirements. Implements
     * GDPR Article 17 Right to Erasure.
     *
     * @param User $user The user to anonymise
     * @return void
     */
    public function anonymiseUser(User $user): void
    {
        $anonymisedId = 'ANON-' . hash('sha256', $user->id . config('app.key'));

        // Update user with anonymised data
        $user->update([
            'email_hash' => hash('sha256', $anonymisedId),
            'email_encrypted' => null,
            'full_name_encrypted' => null,
            'phone_encrypted' => null,
            'address_encrypted' => null,
            'dob_encrypted' => null,
            'password' => null,
            'remember_token' => null,
            'status' => 'anonymised',
            'anonymised_at' => now(),
        ]);

        // Revoke all authentication tokens
        $user->tokens()->delete();

        // Cancel active subscriptions
        $user->subscriptions()->update(['status' => 'cancelled']);

        // Anonymise related data
        $this->anonymiseRelatedData($user);
    }

    /**
     * Anonymises data related to the user account.
     *
     * @param User $user The anonymised user
     * @return void
     */
    protected function anonymiseRelatedData(User $user): void
    {
        // Anonymise messages
        $user->messages()->update([
            'subject' => '[REDACTED]',
            'body' => '[REDACTED]',
        ]);

        // Anonymise activity logs
        $user->activities()->update([
            'metadata' => json_encode(['anonymised' => true]),
        ]);
    }
}
```

### Consent Management

#### app/Models/UserConsent.php

```php
<?php

/**
 * UserConsent.php
 *
 * Tracks user consent for various data processing activities.
 * Maintains full audit trail of consent changes.
 *
 * @package App\Models
 * @version 2.0.0
 */

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class UserConsent extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array<string>
     */
    protected $fillable = [
        'user_id',
        'type',
        'granted',
        'ip_hash',
        'user_agent',
    ];

    /**
     * The attributes that should be cast.
     *
     * @var array<string, string>
     */
    protected $casts = [
        'granted' => 'boolean',
    ];

    /**
     * Consent types that can be tracked.
     *
     * @var array<string, string>
     */
    public const TYPES = [
        'marketing_email' => 'Marketing emails',
        'marketing_sms' => 'Marketing SMS',
        'analytics' => 'Analytics cookies',
        'third_party_sharing' => 'Third-party data sharing',
        'profiling' => 'User profiling',
    ];

    /**
     * Retrieves the user who owns this consent record.
     *
     * @return BelongsTo<User, UserConsent>
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Checks if consent is granted for a specific type.
     *
     * @param string $type The consent type to check
     * @return bool Whether consent is granted
     */
    public function isGranted(string $type): bool
    {
        return $this->where('type', $type)
            ->where('granted', true)
            ->exists();
    }

    /**
     * Retrieves the human-readable label for a consent type.
     *
     * @return string|null The consent type label
     */
    public function getTypeLabel(): ?string
    {
        return self::TYPES[$this->type] ?? null;
    }
}
```

---

## Django/Wagtail (Django 6.x)

### GDPR Views

#### apps/gdpr/views.py

```python
"""
views.py

Handles GDPR data subject requests including data export,
account deletion, and consent management.

@package apps.gdpr
@version 2.0.0
"""

import hashlib
import json
from datetime import datetime

from django.conf import settings
from django.contrib.auth.decorators import login_required
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods
from django.utils import timezone

from apps.gdpr.services.gdpr_export import GdprDataExportService
from apps.gdpr.models import GdprAuditLog


@login_required
@require_http_methods(["GET"])
def export_data(request):
    """
    Exports all personal data for the authenticated user.

    Implements GDPR Article 15 Right to Access. Returns all user data
    in machine-readable JSON format with metadata about the export.
    Logs the export activity for compliance audit trail.

    Args:
        request: The HTTP request containing authenticated user

    Returns:
        JsonResponse: Complete user data export in JSON format
    """
    user = request.user
    export_service = GdprDataExportService()

    # Generate complete data export
    data = export_service.export_user_data(user)

    # Log the export for audit trail compliance
    GdprAuditLog.objects.create(
        user=user,
        action='data_export',
        ip_hash=hashlib.sha256(
            f"{request.META.get('REMOTE_ADDR')}{settings.SECRET_KEY}".encode()
        ).hexdigest(),
    )

    return JsonResponse({
        'export_date': timezone.now().isoformat(),
        'export_type': 'gdpr_article_15',
        'format_version': '2.0.0',
        'data': data,
    })


@login_required
@require_http_methods(["POST"])
def delete_account(request):
    """
    Initiates account deletion request.

    Implements GDPR Article 17 Right to Erasure. Anonymises data that
    must be retained for legal/financial reasons whilst deleting personal data.
    Requires password confirmation to prevent accidental deletion.

    Args:
        request: The HTTP request with password and confirmation

    Returns:
        JsonResponse: Deletion confirmation message
    """
    user = request.user

    # Parse request body
    try:
        data = json.loads(request.body)
    except json.JSONDecodeError:
        return JsonResponse({'error': 'Invalid JSON'}, status=400)

    # Validate required fields
    password = data.get('password')
    confirmation = data.get('confirmation')

    if not password or not confirmation:
        return JsonResponse({'error': 'Missing required fields'}, status=400)

    if confirmation != 'DELETE':
        return JsonResponse({'error': 'Invalid confirmation'}, status=400)

    # Verify password for security
    if not user.check_password(password):
        return JsonResponse({'error': 'Invalid password'}, status=403)

    # Store original user ID hash before anonymisation
    original_user_hash = hashlib.sha256(
        f"{user.id}{settings.SECRET_KEY}".encode()
    ).hexdigest()

    # Anonymise data that must be retained
    export_service = GdprDataExportService()
    export_service.anonymise_user(user)

    # Log the deletion for audit trail
    GdprAuditLog.objects.create(
        user=None,  # User is now anonymised
        action='account_deletion',
        metadata={'original_user_hash': original_user_hash},
        ip_hash=hashlib.sha256(
            f"{request.META.get('REMOTE_ADDR')}{settings.SECRET_KEY}".encode()
        ).hexdigest(),
    )

    return JsonResponse({
        'message': 'Account deletion initiated. Personal data will be removed within 30 days.',
        'deletion_reference': original_user_hash,
    })


@login_required
@require_http_methods(["POST"])
def update_consent(request):
    """
    Updates user consent preferences.

    Implements GDPR Article 7 consent requirements. Records consent
    changes with full audit trail including IP address and timestamp.

    Args:
        request: The HTTP request with consent preferences

    Returns:
        JsonResponse: Updated consent preferences
    """
    user = request.user

    # Parse request body
    try:
        data = json.loads(request.body)
    except json.JSONDecodeError:
        return JsonResponse({'error': 'Invalid JSON'}, status=400)

    consents = data.get('consents', [])

    if not isinstance(consents, list):
        return JsonResponse({'error': 'Consents must be an array'}, status=400)

    # Update each consent
    for consent_data in consents:
        consent_type = consent_data.get('type')
        granted = consent_data.get('granted')

        if not consent_type or granted is None:
            continue

        # Validate consent type
        from apps.gdpr.models import UserConsent
        if consent_type not in UserConsent.CONSENT_TYPES:
            continue

        # Update or create consent
        consent, created = UserConsent.objects.update_or_create(
            user=user,
            type=consent_type,
            defaults={
                'granted': granted,
                'ip_hash': hashlib.sha256(
                    f"{request.META.get('REMOTE_ADDR')}{settings.SECRET_KEY}".encode()
                ).hexdigest(),
                'user_agent': request.META.get('HTTP_USER_AGENT', ''),
            }
        )

        # Log consent change
        GdprAuditLog.objects.create(
            user=user,
            action='consent_change',
            metadata={
                'type': consent_type,
                'granted': granted,
            },
            ip_hash=hashlib.sha256(
                f"{request.META.get('REMOTE_ADDR')}{settings.SECRET_KEY}".encode()
            ).hexdigest(),
        )

    return JsonResponse({
        'message': 'Consent preferences updated successfully',
        'consents': list(user.consents.values('type', 'granted', 'updated_at')),
    })
```

### Data Export Service

#### apps/gdpr/services/gdpr_export.py

```python
"""
gdpr_export.py

Handles GDPR data export and anonymisation operations.
Exports all personal data and anonymises data that must be retained.
"""

import hashlib
from datetime import datetime
from typing import Any

from django.conf import settings

from apps.users.models import User
from apps.core.services.pii_storage import PiiStorageService


class GdprDataExportService:
    """
    Service for GDPR data subject access requests.
    Handles data export and user anonymisation.
    """

    def __init__(self):
        self.pii_service = PiiStorageService()

    def export_user_data(self, user: User) -> dict[str, Any]:
        """
        Exports all personal data for a user.

        Decrypts all PII for the export as the user has right to access
        their own data. Includes account data, activity history, orders,
        communications, and consents.

        Args:
            user: The user to export data for

        Returns:
            Complete user data export dictionary
        """
        return {
            'personal_information': {
                'email': self.pii_service.decrypt(user.email_encrypted),
                'full_name': self.pii_service.decrypt(user.full_name_encrypted),
                'phone': self.pii_service.decrypt(user.phone_encrypted)
                    if user.phone_encrypted else None,
                'address': self.pii_service.decrypt(user.address_encrypted)
                    if user.address_encrypted else None,
                'date_of_birth': self.pii_service.decrypt(user.dob_encrypted)
                    if user.dob_encrypted else None,
            },
            'account_data': {
                'created_at': user.created_at.isoformat(),
                'last_login': user.last_login.isoformat() if user.last_login else None,
                'account_status': user.status,
            },
            'activity_history': list(
                user.activities.values('action', 'created_at')
            ),
            'orders': list(
                user.orders.values('id', 'total', 'status', 'created_at')
            ),
            'communications': list(
                user.messages.values('subject', 'body', 'created_at')
            ),
            'consents': list(
                user.consents.values('type', 'granted', 'updated_at')
            ),
        }

    def anonymise_user(self, user: User) -> None:
        """
        Anonymises a user account for data retention.

        Replaces personal data with anonymised values while retaining
        transaction history for legal/financial requirements.

        Args:
            user: The user to anonymise
        """
        anonymised_id = f"ANON-{hashlib.sha256(f'{user.id}{settings.SECRET_KEY}'.encode()).hexdigest()}"

        user.email_hash = hashlib.sha256(anonymised_id.encode()).hexdigest()
        user.email_encrypted = None
        user.full_name_encrypted = None
        user.phone_encrypted = None
        user.address_encrypted = None
        user.dob_encrypted = None
        user.password = None
        user.status = 'anonymised'
        user.anonymised_at = datetime.now()
        user.save()

        # Revoke all tokens
        user.auth_tokens.all().delete()

        # Cancel subscriptions
        user.subscriptions.update(status='cancelled')
```

---

## Consent Management - Laravel

### app/Models/UserConsent.php

```php
<?php

/**
 * UserConsent.php
 *
 * Tracks user consent for various data processing activities.
 * Maintains full audit trail of consent changes.
 */

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class UserConsent extends Model
{
    protected $fillable = [
        'user_id',
        'type',
        'granted',
        'ip_hash',
        'user_agent',
    ];

    protected $casts = [
        'granted' => 'boolean',
    ];

    /**
     * Consent types that can be tracked.
     */
    public const TYPES = [
        'marketing_email' => 'Marketing emails',
        'marketing_sms' => 'Marketing SMS',
        'analytics' => 'Analytics cookies',
        'third_party_sharing' => 'Third-party data sharing',
        'profiling' => 'User profiling',
    ];

    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

---

## Audit Trail Requirements

All GDPR-related activities must be logged:

```
[2025-01-15 10:30:45] [GDPR] User 123 exported personal data
[2025-01-15 10:31:00] [GDPR] User 456 withdrew marketing consent
[2025-01-15 10:32:00] [GDPR] User 789 requested account deletion
[2025-01-15 10:33:00] [GDPR] Admin exported user list (authorised)
```

### Audit Log Migration

```php
Schema::create('gdpr_audit_log', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->nullable()->constrained()->nullOnDelete();
    $table->string('action', 50)->index(); // data_export, consent_change, deletion
    $table->json('metadata')->nullable();
    $table->string('ip_hash', 64);
    $table->timestamp('created_at')->useCurrent();
});
```
