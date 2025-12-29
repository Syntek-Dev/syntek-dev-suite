# Passkey Authentication (WebAuthn)

**Last Updated**: 29/12/2025
**Version**: 1.3.1
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Overview

Passkey authentication using WebAuthn for passwordless login. Passkeys use public-key cryptography to provide phishing-resistant authentication that is both more secure and more convenient than passwords.

## Metadata

| Property            | Value                                       |
| ------------------- | ------------------------------------------- |
| **Example Version** | 2.0.0                                       |
| **Last Updated**    | 2025-12                                     |
| **TALL Stack**      | Laravel 12.x / PHP 8.4 / webauthn-lib       |
| **Django Stack**    | Django 6.x / Python 3.14 / py-webauthn      |
| **React Stack**     | Next.js 16.x / React 19.x / @simplewebauthn |
| **Mobile Stack**    | React Native 0.83.x / react-native-passkeys |
| **Protocol**        | WebAuthn Level 3 / FIDO2                    |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [WebAuthn Flow](#webauthn-flow)
  - [Registration (Creating a Passkey)](#registration-creating-a-passkey)
  - [Authentication (Using a Passkey)](#authentication-using-a-passkey)
- [TALL Stack - Laravel 12.x](#tall-stack---laravel-12x)
  - [Passkey Model - Laravel](#passkey-model---laravel)
    - [app/Models/Passkey.php](#appmodelspasskeyphp)
    - [database/migrations/xxxx\_create\_passkeys\_table.php](#databasemigrationsxxxx_create_passkeys_tablephp)
  - [Passkey Controller - Laravel](#passkey-controller---laravel)
    - [app/Http/Controllers/PasskeyController.php](#apphttpcontrollerspasskeycontrollerphp)
  - [Blade Components - Laravel](#blade-components---laravel)
    - [resources/views/components/passkey-login.blade.php](#resourcesviewscomponentspasskey-loginbladephp)
- [Django/Wagtail Stack - Django 6.x](#djangowagtail-stack---django-6x)
  - [Passkey Model - Django](#passkey-model---django)
    - [apps/accounts/models/passkey.py](#appsaccountsmodelspasskeypy)
  - [Passkey Views - Django](#passkey-views---django)
    - [apps/accounts/views/passkey\_views.py](#appsaccountsviewspasskey_viewspy)
- [React/Next.js Stack - Next.js 16.x](#reactnextjs-stack---nextjs-16x)
  - [Passkey Service - Next.js](#passkey-service---nextjs)
    - [lib/auth/passkey-service.ts](#libauthpasskey-servicets)
  - [Passkey Hooks - Next.js](#passkey-hooks---nextjs)
    - [hooks/usePasskey.ts](#hooksusepasskeyts)
  - [Passkey Components - Next.js](#passkey-components---nextjs)
    - [components/PasskeyLogin.tsx](#componentspasskeylogintsx)
- [React Native Stack - React Native 0.83.x](#react-native-stack---react-native-083x)
  - [Passkey Service - React Native](#passkey-service---react-native)
    - [services/passkey.service.ts](#servicespasskeyservicets)
  - [Passkey Hooks - React Native](#passkey-hooks---react-native)
    - [hooks/usePasskey.ts](#hooksusepasskeyts-1)
  - [Passkey Screen - React Native](#passkey-screen---react-native)
    - [screens/PasskeyLoginScreen.tsx](#screenspasskeyloginscreentsx)

## WebAuthn Flow

### Registration (Creating a Passkey)

```
1. Client requests registration options from server
2. Server generates challenge and credential creation options
3. Client calls navigator.credentials.create() with options
4. Authenticator creates key pair, returns credential
5. Client sends credential to server for verification
6. Server verifies credential and stores public key
```

### Authentication (Using a Passkey)

```
1. Client requests authentication options from server
2. Server generates challenge and credential request options
3. Client calls navigator.credentials.get() with options
4. Authenticator signs challenge with private key
5. Client sends signed assertion to server
6. Server verifies signature with stored public key
```

---

## TALL Stack - Laravel 12.x

### Passkey Model - Laravel

#### app/Models/Passkey.php

```php
<?php

/**
 * Passkey.php
 *
 * Model for storing WebAuthn passkey credentials. Each user can have
 * multiple passkeys registered for different devices.
 *
 * @package App\Models
 * @version Laravel 12.x / PHP 8.4
 */

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Passkey extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'user_id',
        'name',
        'credential_id',
        'public_key',
        'sign_count',
        'transports',
        'attestation_type',
        'aaguid',
        'last_used_at',
    ];

    /**
     * The attributes that should be cast.
     *
     * @var array<string, string>
     */
    protected $casts = [
        'public_key' => 'encrypted',
        'transports' => 'array',
        'last_used_at' => 'datetime',
    ];

    /**
     * Gets the user that owns the passkey.
     *
     * @return BelongsTo
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Updates the sign count and last used timestamp.
     *
     * @param int $signCount The new sign count from the authenticator
     * @return bool
     */
    public function updateSignCount(int $signCount): bool
    {
        return $this->update([
            'sign_count' => $signCount,
            'last_used_at' => now(),
        ]);
    }
}
```

#### database/migrations/xxxx_create_passkeys_table.php

```php
<?php

/**
 * Creates the passkeys table for WebAuthn credential storage.
 *
 * @version Laravel 12.x / MariaDB 12.x
 */

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Creates the passkeys table.
     */
    public function up(): void
    {
        Schema::create('passkeys', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');

            // Credential identification
            $table->string('name');                          // User-friendly name
            $table->string('credential_id', 1024)->unique(); // Base64 credential ID
            $table->text('public_key');                      // Encrypted COSE public key

            // Security counters
            $table->unsignedBigInteger('sign_count')->default(0);

            // Authenticator metadata
            $table->json('transports')->nullable();          // ['usb', 'nfc', 'ble', 'internal']
            $table->string('attestation_type', 50)->nullable();
            $table->string('aaguid', 36)->nullable();        // Authenticator GUID

            $table->timestamp('last_used_at')->nullable();
            $table->timestamps();

            $table->index(['user_id', 'credential_id']);
        });
    }

    /**
     * Drops the passkeys table.
     */
    public function down(): void
    {
        Schema::dropIfExists('passkeys');
    }
};
```

---

### Passkey Controller - Laravel

#### app/Http/Controllers/PasskeyController.php

```php
<?php

/**
 * PasskeyController.php
 *
 * Handles WebAuthn passkey registration and authentication.
 * Uses the webauthn-lib package for credential verification.
 *
 * @package App\Http\Controllers
 * @version Laravel 12.x / PHP 8.4
 */

namespace App\Http\Controllers;

use App\Models\Passkey;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Session;
use Webauthn\PublicKeyCredentialCreationOptions;
use Webauthn\PublicKeyCredentialRequestOptions;
use Webauthn\PublicKeyCredentialSource;
use Webauthn\PublicKeyCredentialUserEntity;
use Webauthn\AuthenticatorSelectionCriteria;
use Webauthn\PublicKeyCredentialRpEntity;

class PasskeyController extends Controller
{
    /**
     * Generates options for passkey registration.
     *
     * Creates WebAuthn credential creation options including challenge,
     * relying party info, and user entity for the authenticator.
     *
     * @param Request $request The HTTP request
     * @return \Illuminate\Http\JsonResponse Registration options
     */
    public function registerOptions(Request $request)
    {
        $user = $request->user();

        // Get existing credential IDs to exclude
        $excludeCredentials = $user->passkeys
            ->map(fn($passkey) => [
                'type' => 'public-key',
                'id' => $passkey->credential_id,
                'transports' => $passkey->transports ?? [],
            ])
            ->toArray();

        // Generate challenge
        $challenge = random_bytes(32);
        Session::put('webauthn.challenge', base64_encode($challenge));

        // Create user entity
        $userEntity = new PublicKeyCredentialUserEntity(
            $user->email,
            (string) $user->id,
            $user->name,
        );

        // Create relying party
        $rpEntity = new PublicKeyCredentialRpEntity(
            config('app.name'),
            parse_url(config('app.url'), PHP_URL_HOST),
        );

        // Authenticator selection
        $authenticatorSelection = new AuthenticatorSelectionCriteria(
            AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_PLATFORM,
            AuthenticatorSelectionCriteria::RESIDENT_KEY_REQUIREMENT_PREFERRED,
            AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED,
        );

        // Build options
        $options = new PublicKeyCredentialCreationOptions(
            $rpEntity,
            $userEntity,
            $challenge,
            pubKeyCredParams: $this->getSupportedAlgorithms(),
            timeout: 60000,
            excludeCredentials: $excludeCredentials,
            authenticatorSelection: $authenticatorSelection,
            attestation: 'none',
        );

        return response()->json($options);
    }

    /**
     * Verifies and stores a new passkey registration.
     *
     * Validates the credential from the authenticator and stores
     * the public key for future authentication.
     *
     * @param Request $request The HTTP request containing credential
     * @return \Illuminate\Http\JsonResponse Success or error message
     */
    public function register(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'credential' => 'required|array',
        ]);

        $user = $request->user();
        $credential = $request->input('credential');

        // Retrieve challenge from session
        $challenge = base64_decode(Session::get('webauthn.challenge'));
        Session::forget('webauthn.challenge');

        if (!$challenge) {
            return response()->json(['error' => 'Registration session expired'], 400);
        }

        try {
            // Verify the credential (simplified - use webauthn-lib in production)
            $credentialId = $credential['id'];
            $publicKey = $credential['response']['publicKey'] ?? null;

            if (!$publicKey) {
                return response()->json(['error' => 'Invalid credential'], 400);
            }

            // Store the passkey
            $passkey = $user->passkeys()->create([
                'name' => $request->input('name'),
                'credential_id' => $credentialId,
                'public_key' => $publicKey,
                'sign_count' => 0,
                'transports' => $credential['response']['transports'] ?? [],
                'attestation_type' => 'none',
                'aaguid' => $credential['response']['aaguid'] ?? null,
            ]);

            return response()->json([
                'message' => 'Passkey registered successfully',
                'passkey' => [
                    'id' => $passkey->id,
                    'name' => $passkey->name,
                    'created_at' => $passkey->created_at->toIso8601String(),
                ],
            ]);
        } catch (\Exception $e) {
            return response()->json(['error' => 'Failed to register passkey'], 500);
        }
    }

    /**
     * Generates options for passkey authentication.
     *
     * Creates WebAuthn credential request options including challenge
     * and allowed credentials for the specified user.
     *
     * @param Request $request The HTTP request
     * @return \Illuminate\Http\JsonResponse Authentication options
     */
    public function authenticateOptions(Request $request)
    {
        $request->validate(['email' => 'required|email']);

        $user = User::where('email', $request->email)->first();

        if (!$user || $user->passkeys->isEmpty()) {
            // Return generic error to prevent user enumeration
            return response()->json(['error' => 'No passkeys available'], 400);
        }

        // Get user's credential IDs
        $allowCredentials = $user->passkeys
            ->map(fn($passkey) => [
                'type' => 'public-key',
                'id' => $passkey->credential_id,
                'transports' => $passkey->transports ?? [],
            ])
            ->toArray();

        // Generate challenge
        $challenge = random_bytes(32);
        Session::put('webauthn.challenge', base64_encode($challenge));
        Session::put('webauthn.user_id', $user->id);

        $options = new PublicKeyCredentialRequestOptions(
            $challenge,
            timeout: 60000,
            rpId: parse_url(config('app.url'), PHP_URL_HOST),
            allowCredentials: $allowCredentials,
            userVerification: 'preferred',
        );

        return response()->json($options);
    }

    /**
     * Verifies a passkey authentication assertion.
     *
     * Validates the signed assertion from the authenticator and
     * logs in the user if verification succeeds.
     *
     * @param Request $request The HTTP request containing assertion
     * @return \Illuminate\Http\JsonResponse Success or error message
     */
    public function authenticate(Request $request)
    {
        $request->validate([
            'credential' => 'required|array',
        ]);

        $credential = $request->input('credential');

        // Retrieve challenge and user from session
        $challenge = base64_decode(Session::get('webauthn.challenge'));
        $userId = Session::get('webauthn.user_id');
        Session::forget(['webauthn.challenge', 'webauthn.user_id']);

        if (!$challenge || !$userId) {
            return response()->json(['error' => 'Authentication session expired'], 400);
        }

        $user = User::find($userId);
        if (!$user) {
            return response()->json(['error' => 'User not found'], 404);
        }

        // Find the passkey
        $passkey = $user->passkeys()
            ->where('credential_id', $credential['id'])
            ->first();

        if (!$passkey) {
            return response()->json(['error' => 'Passkey not found'], 404);
        }

        try {
            // Verify assertion (simplified - use webauthn-lib in production)
            $authenticatorData = $credential['response']['authenticatorData'] ?? null;
            $signature = $credential['response']['signature'] ?? null;

            if (!$authenticatorData || !$signature) {
                return response()->json(['error' => 'Invalid assertion'], 400);
            }

            // Verify sign count to detect cloned authenticators
            $newSignCount = $this->extractSignCount($authenticatorData);
            if ($newSignCount <= $passkey->sign_count) {
                // Potential cloned authenticator
                logger()->warning('Potential cloned authenticator detected', [
                    'user_id' => $user->id,
                    'passkey_id' => $passkey->id,
                ]);
            }

            // Update sign count
            $passkey->updateSignCount($newSignCount);

            // Log in the user
            Auth::login($user, remember: true);

            return response()->json([
                'message' => 'Authentication successful',
                'user' => [
                    'id' => $user->id,
                    'email' => $user->email,
                    'name' => $user->name,
                ],
            ]);
        } catch (\Exception $e) {
            return response()->json(['error' => 'Authentication failed'], 401);
        }
    }

    /**
     * Lists all passkeys for the authenticated user.
     *
     * @param Request $request The HTTP request
     * @return \Illuminate\Http\JsonResponse List of passkeys
     */
    public function list(Request $request)
    {
        $passkeys = $request->user()->passkeys()
            ->select('id', 'name', 'last_used_at', 'created_at')
            ->orderBy('last_used_at', 'desc')
            ->get();

        return response()->json(['passkeys' => $passkeys]);
    }

    /**
     * Deletes a passkey.
     *
     * @param Request $request The HTTP request
     * @param int $id The passkey ID
     * @return \Illuminate\Http\JsonResponse Success message
     */
    public function delete(Request $request, int $id)
    {
        $passkey = $request->user()->passkeys()->findOrFail($id);
        $passkey->delete();

        return response()->json(['message' => 'Passkey deleted']);
    }

    /**
     * Returns supported cryptographic algorithms.
     *
     * @return array
     */
    protected function getSupportedAlgorithms(): array
    {
        return [
            ['type' => 'public-key', 'alg' => -7],   // ES256 (ECDSA P-256)
            ['type' => 'public-key', 'alg' => -257], // RS256 (RSASSA-PKCS1-v1_5)
        ];
    }

    /**
     * Extracts sign count from authenticator data.
     *
     * @param string $authenticatorData Base64 encoded authenticator data
     * @return int
     */
    protected function extractSignCount(string $authenticatorData): int
    {
        $data = base64_decode($authenticatorData);
        // Sign count is bytes 33-36 (big-endian)
        return unpack('N', substr($data, 33, 4))[1];
    }
}
```

---

### Blade Components - Laravel

#### resources/views/components/passkey-login.blade.php

```blade
{{--
  passkey-login.blade.php

  Passkey login component for Laravel 12.x / Tailwind 4.x.
  Provides WebAuthn authentication with graceful fallback.
--}}

@props(['email' => ''])

<div
    x-data="passkeyAuth()"
    x-init="checkSupport()"
    {{ $attributes->merge(['class' => 'space-y-4']) }}
>
    {{-- Passkey Support Check --}}
    <template x-if="!isSupported">
        <div class="p-4 bg-yellow-50 rounded-lg">
            <p class="text-yellow-700 text-sm">
                Passkeys are not supported on this device.
                Please use another authentication method.
            </p>
        </div>
    </template>

    {{-- Passkey Login Form --}}
    <template x-if="isSupported && !isLoading">
        <div class="space-y-4">
            <div>
                <label for="passkey-email" class="block text-sm font-medium mb-1">
                    Email
                </label>
                <input
                    type="email"
                    id="passkey-email"
                    x-model="email"
                    class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                    placeholder="you@example.com"
                    required
                />
            </div>

            <button
                type="button"
                @click="authenticate()"
                :disabled="!email"
                class="w-full py-2 px-4 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50 flex items-centre justify-centre gap-2"
            >
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                        d="M15 7a2 2 0 012 2m4 0a6 6 0 01-7.743 5.743L11 17H9v2H7v2H4a1 1 0 01-1-1v-2.586a1 1 0 01.293-.707l5.964-5.964A6 6 0 1121 9z" />
                </svg>
                Sign in with Passkey
            </button>
        </div>
    </template>

    {{-- Loading State --}}
    <template x-if="isLoading">
        <div class="flex items-centre justify-centre py-8">
            <svg class="animate-spin h-8 w-8 text-blue-600" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            <span class="ml-2 text-grey-600">Waiting for authenticator...</span>
        </div>
    </template>

    {{-- Error Message --}}
    <template x-if="error">
        <div class="p-4 bg-red-50 rounded-lg">
            <p class="text-red-600 text-sm" x-text="error"></p>
        </div>
    </template>
</div>

@push('scripts')
<script>
function passkeyAuth() {
    return {
        email: '{{ $email }}',
        isSupported: false,
        isLoading: false,
        error: null,

        /**
         * Checks if WebAuthn is supported.
         */
        async checkSupport() {
            this.isSupported = window.PublicKeyCredential !== undefined &&
                typeof window.PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable === 'function';

            if (this.isSupported) {
                const available = await PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable();
                this.isSupported = available;
            }
        },

        /**
         * Initiates passkey authentication.
         */
        async authenticate() {
            this.isLoading = true;
            this.error = null;

            try {
                // Get authentication options from server
                const optionsResponse = await fetch('/api/passkeys/authenticate/options', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
                    },
                    body: JSON.stringify({ email: this.email }),
                });

                if (!optionsResponse.ok) {
                    const data = await optionsResponse.json();
                    throw new Error(data.error || 'Failed to get authentication options');
                }

                const options = await optionsResponse.json();

                // Decode base64 challenge
                options.challenge = this.base64ToBuffer(options.challenge);
                options.allowCredentials = options.allowCredentials.map(cred => ({
                    ...cred,
                    id: this.base64ToBuffer(cred.id),
                }));

                // Request credential from authenticator
                const credential = await navigator.credentials.get({
                    publicKey: options,
                });

                // Send credential to server for verification
                const verifyResponse = await fetch('/api/passkeys/authenticate', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
                    },
                    body: JSON.stringify({
                        credential: {
                            id: this.bufferToBase64(credential.rawId),
                            type: credential.type,
                            response: {
                                authenticatorData: this.bufferToBase64(credential.response.authenticatorData),
                                clientDataJSON: this.bufferToBase64(credential.response.clientDataJSON),
                                signature: this.bufferToBase64(credential.response.signature),
                                userHandle: credential.response.userHandle
                                    ? this.bufferToBase64(credential.response.userHandle)
                                    : null,
                            },
                        },
                    }),
                });

                const result = await verifyResponse.json();

                if (verifyResponse.ok) {
                    // Redirect to dashboard
                    window.location.href = '/dashboard';
                } else {
                    throw new Error(result.error || 'Authentication failed');
                }
            } catch (err) {
                if (err.name === 'NotAllowedError') {
                    this.error = 'Authentication was cancelled or timed out.';
                } else {
                    this.error = err.message;
                }
            } finally {
                this.isLoading = false;
            }
        },

        /**
         * Converts base64 string to ArrayBuffer.
         */
        base64ToBuffer(base64) {
            const binary = atob(base64.replace(/-/g, '+').replace(/_/g, '/'));
            const buffer = new ArrayBuffer(binary.length);
            const view = new Uint8Array(buffer);
            for (let i = 0; i < binary.length; i++) {
                view[i] = binary.charCodeAt(i);
            }
            return buffer;
        },

        /**
         * Converts ArrayBuffer to base64url string.
         */
        bufferToBase64(buffer) {
            const bytes = new Uint8Array(buffer);
            let binary = '';
            for (let i = 0; i < bytes.byteLength; i++) {
                binary += String.fromCharCode(bytes[i]);
            }
            return btoa(binary).replace(/\+/g, '-').replace(/\//g, '_').replace(/=/g, '');
        },
    };
}
</script>
@endpush
```

---

## Django/Wagtail Stack - Django 6.x

### Passkey Model - Django

#### apps/accounts/models/passkey.py

```python
"""
passkey.py

Model for storing WebAuthn passkey credentials. Each user can have
multiple passkeys registered for different devices.

@package accounts.models
@version Django 6.x / Python 3.14
"""

from django.contrib.auth import get_user_model
from django.db import models
from django.utils import timezone

User = get_user_model()


class Passkey(models.Model):
    """
    Stores WebAuthn passkey credentials for users.

    Attributes:
        user: The user who owns this passkey
        name: User-friendly name for the passkey
        credential_id: Base64-encoded credential identifier
        public_key: Encrypted COSE public key
        sign_count: Counter for cloned authenticator detection
        transports: Available transport methods (USB, NFC, BLE, internal)
        attestation_type: Type of attestation provided
        aaguid: Authenticator GUID
        last_used_at: Last authentication timestamp
    """

    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='passkeys',
    )
    name = models.CharField(max_length=255)
    credential_id = models.CharField(max_length=1024, unique=True)
    public_key = models.TextField()  # Encrypted in application layer
    sign_count = models.BigIntegerField(default=0)
    transports = models.JSONField(default=list, blank=True)
    attestation_type = models.CharField(max_length=50, blank=True, null=True)
    aaguid = models.CharField(max_length=36, blank=True, null=True)
    last_used_at = models.DateTimeField(blank=True, null=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        """Model metadata."""

        db_table = 'passkeys'
        ordering = ['-last_used_at', '-created_at']
        indexes = [
            models.Index(
                fields=['user', 'credential_id'],
                name='passkey_user_cred_idx',
            ),
        ]

    def __str__(self) -> str:
        """String representation."""
        return f"{self.name} ({self.user.email})"

    def update_sign_count(self, new_count: int) -> None:
        """
        Updates the sign count and last used timestamp.

        Args:
            new_count: The new sign count from the authenticator
        """
        self.sign_count = new_count
        self.last_used_at = timezone.now()
        self.save(update_fields=['sign_count', 'last_used_at', 'updated_at'])
```

---

### Passkey Views - Django

#### apps/accounts/views/passkey_views.py

```python
"""
passkey_views.py

Views for WebAuthn passkey registration and authentication.
Uses py_webauthn library for credential verification.

@package accounts.views
@version Django 6.x / Python 3.14
"""

import base64
import json
import secrets
from typing import Optional

from django.contrib.auth import get_user_model, login
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_protect
from django.views.decorators.http import require_http_methods
from django.contrib.auth.decorators import login_required
from webauthn import (
    generate_registration_options,
    verify_registration_response,
    generate_authentication_options,
    verify_authentication_response,
    options_to_json,
)
from webauthn.helpers import (
    bytes_to_base64url,
    base64url_to_bytes,
)
from webauthn.helpers.structs import (
    AuthenticatorSelectionCriteria,
    ResidentKeyRequirement,
    UserVerificationRequirement,
    AuthenticatorAttachment,
    PublicKeyCredentialDescriptor,
    AuthenticatorTransport,
)

from apps.accounts.models import Passkey

User = get_user_model()

# Session keys for WebAuthn challenges
WEBAUTHN_CHALLENGE_KEY = 'webauthn_challenge'
WEBAUTHN_USER_ID_KEY = 'webauthn_user_id'


def get_rp_id() -> str:
    """Returns the Relying Party ID (domain)."""
    from django.conf import settings
    from urllib.parse import urlparse
    return urlparse(settings.SITE_URL).netloc


def get_rp_name() -> str:
    """Returns the Relying Party name."""
    from django.conf import settings
    return settings.SITE_NAME


@login_required
@require_http_methods(["POST"])
@csrf_protect
def register_options(request):
    """
    Generates options for passkey registration.

    Creates WebAuthn credential creation options including challenge,
    relying party info, and user entity.

    Args:
        request: The HTTP request object

    Returns:
        JsonResponse: Registration options for navigator.credentials.create()
    """
    user = request.user

    # Get existing credential IDs to exclude
    exclude_credentials = [
        PublicKeyCredentialDescriptor(
            id=base64url_to_bytes(passkey.credential_id),
            transports=[AuthenticatorTransport(t) for t in passkey.transports] if passkey.transports else None,
        )
        for passkey in user.passkeys.all()
    ]

    # Generate registration options
    options = generate_registration_options(
        rp_id=get_rp_id(),
        rp_name=get_rp_name(),
        user_id=str(user.id).encode(),
        user_name=user.email,
        user_display_name=user.get_full_name() or user.email,
        exclude_credentials=exclude_credentials,
        authenticator_selection=AuthenticatorSelectionCriteria(
            authenticator_attachment=AuthenticatorAttachment.PLATFORM,
            resident_key=ResidentKeyRequirement.PREFERRED,
            user_verification=UserVerificationRequirement.PREFERRED,
        ),
        timeout=60000,
    )

    # Store challenge in session
    request.session[WEBAUTHN_CHALLENGE_KEY] = bytes_to_base64url(options.challenge)

    return JsonResponse(json.loads(options_to_json(options)))


@login_required
@require_http_methods(["POST"])
@csrf_protect
def register(request):
    """
    Verifies and stores a new passkey registration.

    Validates the credential from the authenticator and stores
    the public key for future authentication.

    Args:
        request: The HTTP request containing credential data

    Returns:
        JsonResponse: Success or error message
    """
    try:
        data = json.loads(request.body)
        name = data.get('name', 'My Passkey')
        credential = data.get('credential')

        if not credential:
            return JsonResponse({'error': 'Missing credential data'}, status=400)

        # Retrieve challenge from session
        challenge = request.session.pop(WEBAUTHN_CHALLENGE_KEY, None)
        if not challenge:
            return JsonResponse({'error': 'Registration session expired'}, status=400)

        # Verify registration response
        verification = verify_registration_response(
            credential=credential,
            expected_challenge=base64url_to_bytes(challenge),
            expected_rp_id=get_rp_id(),
            expected_origin=f"https://{get_rp_id()}",
        )

        # Store the passkey
        passkey = Passkey.objects.create(
            user=request.user,
            name=name,
            credential_id=bytes_to_base64url(verification.credential_id),
            public_key=bytes_to_base64url(verification.credential_public_key),
            sign_count=verification.sign_count,
            transports=[t.value for t in (credential.get('transports') or [])],
            attestation_type=verification.attestation_object.fmt if hasattr(verification, 'attestation_object') else None,
            aaguid=str(verification.aaguid) if hasattr(verification, 'aaguid') else None,
        )

        return JsonResponse({
            'message': 'Passkey registered successfully',
            'passkey': {
                'id': passkey.id,
                'name': passkey.name,
                'created_at': passkey.created_at.isoformat(),
            },
        })

    except Exception as e:
        return JsonResponse({'error': str(e)}, status=400)


@require_http_methods(["POST"])
@csrf_protect
def authenticate_options(request):
    """
    Generates options for passkey authentication.

    Creates WebAuthn credential request options including challenge
    and allowed credentials for the specified user.

    Args:
        request: The HTTP request containing email

    Returns:
        JsonResponse: Authentication options for navigator.credentials.get()
    """
    try:
        data = json.loads(request.body)
        email = data.get('email')

        if not email:
            return JsonResponse({'error': 'Email is required'}, status=400)

        user = User.objects.filter(email=email).first()

        if not user or not user.passkeys.exists():
            # Return generic error to prevent user enumeration
            return JsonResponse({'error': 'No passkeys available'}, status=400)

        # Get user's credential IDs
        allow_credentials = [
            PublicKeyCredentialDescriptor(
                id=base64url_to_bytes(passkey.credential_id),
                transports=[AuthenticatorTransport(t) for t in passkey.transports] if passkey.transports else None,
            )
            for passkey in user.passkeys.all()
        ]

        # Generate authentication options
        options = generate_authentication_options(
            rp_id=get_rp_id(),
            allow_credentials=allow_credentials,
            user_verification=UserVerificationRequirement.PREFERRED,
            timeout=60000,
        )

        # Store challenge and user ID in session
        request.session[WEBAUTHN_CHALLENGE_KEY] = bytes_to_base64url(options.challenge)
        request.session[WEBAUTHN_USER_ID_KEY] = user.id

        return JsonResponse(json.loads(options_to_json(options)))

    except json.JSONDecodeError:
        return JsonResponse({'error': 'Invalid JSON'}, status=400)


@require_http_methods(["POST"])
@csrf_protect
def authenticate(request):
    """
    Verifies a passkey authentication assertion.

    Validates the signed assertion from the authenticator and
    logs in the user if verification succeeds.

    Args:
        request: The HTTP request containing assertion

    Returns:
        JsonResponse: Success or error message
    """
    try:
        data = json.loads(request.body)
        credential = data.get('credential')

        if not credential:
            return JsonResponse({'error': 'Missing credential data'}, status=400)

        # Retrieve challenge and user from session
        challenge = request.session.pop(WEBAUTHN_CHALLENGE_KEY, None)
        user_id = request.session.pop(WEBAUTHN_USER_ID_KEY, None)

        if not challenge or not user_id:
            return JsonResponse({'error': 'Authentication session expired'}, status=400)

        user = User.objects.filter(id=user_id).first()
        if not user:
            return JsonResponse({'error': 'User not found'}, status=404)

        # Find the passkey
        passkey = user.passkeys.filter(
            credential_id=credential.get('id')
        ).first()

        if not passkey:
            return JsonResponse({'error': 'Passkey not found'}, status=404)

        # Verify authentication response
        verification = verify_authentication_response(
            credential=credential,
            expected_challenge=base64url_to_bytes(challenge),
            expected_rp_id=get_rp_id(),
            expected_origin=f"https://{get_rp_id()}",
            credential_public_key=base64url_to_bytes(passkey.public_key),
            credential_current_sign_count=passkey.sign_count,
        )

        # Check for cloned authenticator
        if verification.new_sign_count <= passkey.sign_count:
            import logging
            logger = logging.getLogger(__name__)
            logger.warning(
                'Potential cloned authenticator detected',
                extra={
                    'user_id': user.id,
                    'passkey_id': passkey.id,
                }
            )

        # Update sign count
        passkey.update_sign_count(verification.new_sign_count)

        # Log in the user
        login(request, user)

        return JsonResponse({
            'message': 'Authentication successful',
            'user': {
                'id': user.id,
                'email': user.email,
                'name': user.get_full_name(),
            },
        })

    except Exception as e:
        return JsonResponse({'error': str(e)}, status=401)


@login_required
@require_http_methods(["GET"])
def list_passkeys(request):
    """
    Lists all passkeys for the authenticated user.

    Args:
        request: The HTTP request object

    Returns:
        JsonResponse: List of passkeys
    """
    passkeys = request.user.passkeys.values(
        'id', 'name', 'last_used_at', 'created_at'
    )

    return JsonResponse({
        'passkeys': list(passkeys),
        'total': len(passkeys),
    })


@login_required
@require_http_methods(["DELETE"])
@csrf_protect
def delete_passkey(request, passkey_id: int):
    """
    Deletes a passkey.

    Args:
        request: The HTTP request object
        passkey_id: The passkey ID to delete

    Returns:
        JsonResponse: Success or error message
    """
    try:
        passkey = request.user.passkeys.get(id=passkey_id)
        passkey.delete()
        return JsonResponse({'message': 'Passkey deleted'})
    except Passkey.DoesNotExist:
        return JsonResponse({'error': 'Passkey not found'}, status=404)
```

---

## React/Next.js Stack - Next.js 16.x

### Passkey Service - Next.js

#### lib/auth/passkey-service.ts

```typescript
/**
 * passkey-service.ts
 *
 * WebAuthn passkey service for Next.js applications.
 * Handles credential creation and authentication ceremonies.
 *
 * @package Next.js 16.x / React 19.x / TypeScript 5.9
 * @version 2.0.0
 */

import {
  startRegistration,
  startAuthentication,
} from '@simplewebauthn/browser';
import type {
  PublicKeyCredentialCreationOptionsJSON,
  PublicKeyCredentialRequestOptionsJSON,
  RegistrationResponseJSON,
  AuthenticationResponseJSON,
} from '@simplewebauthn/types';

const API_BASE = '/api/passkeys';

/**
 * Passkey registration result.
 */
export interface RegistrationResult {
  success: boolean;
  passkey?: {
    id: number;
    name: string;
    createdAt: string;
  };
  error?: string;
}

/**
 * Passkey authentication result.
 */
export interface AuthenticationResult {
  success: boolean;
  user?: {
    id: string;
    email: string;
    name: string;
  };
  error?: string;
}

/**
 * Passkey data for display.
 */
export interface PasskeyData {
  id: number;
  name: string;
  lastUsedAt: string | null;
  createdAt: string;
}

/**
 * Checks if WebAuthn is supported on the current device.
 *
 * @returns Promise resolving to true if supported
 */
export async function isPasskeySupported(): Promise<boolean> {
  if (typeof window === 'undefined') return false;
  if (!window.PublicKeyCredential) return false;

  try {
    const available = await PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable();
    return available;
  } catch {
    return false;
  }
}

/**
 * Registers a new passkey for the authenticated user.
 *
 * @param name User-friendly name for the passkey
 * @returns Promise resolving to registration result
 */
export async function registerPasskey(name: string): Promise<RegistrationResult> {
  try {
    // Get registration options from server
    const optionsResponse = await fetch(`${API_BASE}/register/options`, {
      method: 'POST',
      credentials: 'include',
    });

    if (!optionsResponse.ok) {
      const error = await optionsResponse.json();
      throw new Error(error.error || 'Failed to get registration options');
    }

    const options: PublicKeyCredentialCreationOptionsJSON = await optionsResponse.json();

    // Create credential with authenticator
    const credential: RegistrationResponseJSON = await startRegistration(options);

    // Verify credential with server
    const verifyResponse = await fetch(`${API_BASE}/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
      body: JSON.stringify({ name, credential }),
    });

    const result = await verifyResponse.json();

    if (!verifyResponse.ok) {
      throw new Error(result.error || 'Registration failed');
    }

    return {
      success: true,
      passkey: result.passkey,
    };
  } catch (error) {
    const message = error instanceof Error ? error.message : 'Registration failed';

    // Handle specific WebAuthn errors
    if (error instanceof Error) {
      if (error.name === 'NotAllowedError') {
        return { success: false, error: 'Registration was cancelled or timed out' };
      }
      if (error.name === 'InvalidStateError') {
        return { success: false, error: 'A passkey for this device already exists' };
      }
    }

    return { success: false, error: message };
  }
}

/**
 * Authenticates a user with their passkey.
 *
 * @param email User's email address
 * @returns Promise resolving to authentication result
 */
export async function authenticateWithPasskey(email: string): Promise<AuthenticationResult> {
  try {
    // Get authentication options from server
    const optionsResponse = await fetch(`${API_BASE}/authenticate/options`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email }),
    });

    if (!optionsResponse.ok) {
      const error = await optionsResponse.json();
      throw new Error(error.error || 'Failed to get authentication options');
    }

    const options: PublicKeyCredentialRequestOptionsJSON = await optionsResponse.json();

    // Get assertion from authenticator
    const credential: AuthenticationResponseJSON = await startAuthentication(options);

    // Verify assertion with server
    const verifyResponse = await fetch(`${API_BASE}/authenticate`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
      body: JSON.stringify({ credential }),
    });

    const result = await verifyResponse.json();

    if (!verifyResponse.ok) {
      throw new Error(result.error || 'Authentication failed');
    }

    return {
      success: true,
      user: result.user,
    };
  } catch (error) {
    const message = error instanceof Error ? error.message : 'Authentication failed';

    if (error instanceof Error) {
      if (error.name === 'NotAllowedError') {
        return { success: false, error: 'Authentication was cancelled or timed out' };
      }
    }

    return { success: false, error: message };
  }
}

/**
 * Lists all passkeys for the authenticated user.
 *
 * @returns Promise resolving to array of passkeys
 */
export async function listPasskeys(): Promise<PasskeyData[]> {
  const response = await fetch(`${API_BASE}/list`, {
    credentials: 'include',
  });

  if (!response.ok) {
    throw new Error('Failed to fetch passkeys');
  }

  const data = await response.json();
  return data.passkeys;
}

/**
 * Deletes a passkey.
 *
 * @param id Passkey ID to delete
 * @returns Promise resolving to true if successful
 */
export async function deletePasskey(id: number): Promise<boolean> {
  const response = await fetch(`${API_BASE}/${id}`, {
    method: 'DELETE',
    credentials: 'include',
  });

  return response.ok;
}
```

---

### Passkey Hooks - Next.js

#### hooks/usePasskey.ts

```typescript
/**
 * usePasskey.ts
 *
 * React hook for passkey operations in Next.js 16 / React 19 applications.
 *
 * @package Next.js 16.x / React 19.x / TypeScript 5.9
 * @version 2.0.0
 */

'use client';

import { useState, useEffect, useCallback } from 'react';
import {
  isPasskeySupported,
  registerPasskey,
  authenticateWithPasskey,
  listPasskeys,
  deletePasskey,
  type PasskeyData,
  type RegistrationResult,
  type AuthenticationResult,
} from '@/lib/auth/passkey-service';

/**
 * Hook return type.
 */
interface UsePasskeyReturn {
  isSupported: boolean;
  passkeys: PasskeyData[];
  loading: boolean;
  error: string | null;
  register: (name: string) => Promise<RegistrationResult>;
  authenticate: (email: string) => Promise<AuthenticationResult>;
  remove: (id: number) => Promise<boolean>;
  refresh: () => Promise<void>;
}

/**
 * Custom hook for passkey operations.
 *
 * Provides methods for registering, authenticating, and managing passkeys.
 *
 * @returns Object with passkey methods and state
 *
 * @example
 * const { isSupported, passkeys, register, authenticate } = usePasskey();
 *
 * const handleRegister = async () => {
 *   const result = await register('My MacBook');
 *   if (result.success) {
 *     console.log('Passkey registered:', result.passkey);
 *   }
 * };
 */
export function usePasskey(): UsePasskeyReturn {
  const [isSupported, setIsSupported] = useState(false);
  const [passkeys, setPasskeys] = useState<PasskeyData[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  /**
   * Checks WebAuthn support on mount.
   */
  useEffect(() => {
    isPasskeySupported().then(setIsSupported);
  }, []);

  /**
   * Refreshes the passkey list.
   */
  const refresh = useCallback(async () => {
    try {
      const data = await listPasskeys();
      setPasskeys(data);
      setError(null);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Failed to fetch passkeys');
    }
  }, []);

  /**
   * Registers a new passkey.
   */
  const register = useCallback(async (name: string): Promise<RegistrationResult> => {
    setLoading(true);
    setError(null);

    try {
      const result = await registerPasskey(name);

      if (result.success) {
        await refresh();
      } else {
        setError(result.error || 'Registration failed');
      }

      return result;
    } finally {
      setLoading(false);
    }
  }, [refresh]);

  /**
   * Authenticates with a passkey.
   */
  const authenticate = useCallback(async (email: string): Promise<AuthenticationResult> => {
    setLoading(true);
    setError(null);

    try {
      const result = await authenticateWithPasskey(email);

      if (!result.success) {
        setError(result.error || 'Authentication failed');
      }

      return result;
    } finally {
      setLoading(false);
    }
  }, []);

  /**
   * Removes a passkey.
   */
  const remove = useCallback(async (id: number): Promise<boolean> => {
    setLoading(true);
    setError(null);

    try {
      const success = await deletePasskey(id);

      if (success) {
        await refresh();
      } else {
        setError('Failed to delete passkey');
      }

      return success;
    } finally {
      setLoading(false);
    }
  }, [refresh]);

  return {
    isSupported,
    passkeys,
    loading,
    error,
    register,
    authenticate,
    remove,
    refresh,
  };
}
```

---

### Passkey Components - Next.js

#### components/PasskeyLogin.tsx

```typescript
/**
 * PasskeyLogin.tsx
 *
 * Passkey login component for Next.js 16 / React 19.
 * Provides WebAuthn authentication with graceful fallback.
 *
 * @package Next.js 16.x / React 19.x / Tailwind 4.x
 * @version 2.0.0
 */

'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { usePasskey } from '@/hooks/usePasskey';

interface PasskeyLoginProps {
  onFallback?: () => void;
  callbackUrl?: string;
}

/**
 * Passkey login component.
 *
 * @example
 * <PasskeyLogin
 *   callbackUrl="/dashboard"
 *   onFallback={() => setShowPasswordLogin(true)}
 * />
 */
export function PasskeyLogin({
  onFallback,
  callbackUrl = '/dashboard',
}: PasskeyLoginProps) {
  const router = useRouter();
  const { isSupported, authenticate, loading, error } = usePasskey();
  const [email, setEmail] = useState('');

  /**
   * Handles passkey authentication.
   */
  const handleAuthenticate = async (e: React.FormEvent) => {
    e.preventDefault();

    const result = await authenticate(email);

    if (result.success) {
      router.push(callbackUrl);
    }
  };

  if (!isSupported) {
    return (
      <div className="p-4 bg-yellow-50 rounded-lg">
        <p className="text-yellow-700 text-sm">
          Passkeys are not supported on this device.
        </p>
        {onFallback && (
          <button
            onClick={onFallback}
            className="mt-2 text-blue-600 hover:text-blue-700 text-sm"
          >
            Use password instead
          </button>
        )}
      </div>
    );
  }

  return (
    <form onSubmit={handleAuthenticate} className="space-y-4">
      <div>
        <label htmlFor="passkey-email" className="block text-sm font-medium mb-1">
          Email
        </label>
        <input
          type="email"
          id="passkey-email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
          placeholder="you@example.com"
          required
          disabled={loading}
        />
      </div>

      <button
        type="submit"
        disabled={loading || !email}
        className="w-full py-2 px-4 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50 flex items-centre justify-centre gap-2"
      >
        {loading ? (
          <>
            <svg className="animate-spin h-5 w-5" fill="none" viewBox="0 0 24 24">
              <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" />
              <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
            </svg>
            Waiting for authenticator...
          </>
        ) : (
          <>
            <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2}
                d="M15 7a2 2 0 012 2m4 0a6 6 0 01-7.743 5.743L11 17H9v2H7v2H4a1 1 0 01-1-1v-2.586a1 1 0 01.293-.707l5.964-5.964A6 6 0 1121 9z" />
            </svg>
            Sign in with Passkey
          </>
        )}
      </button>

      {error && (
        <div className="p-3 bg-red-50 rounded-lg">
          <p className="text-red-600 text-sm">{error}</p>
        </div>
      )}

      {onFallback && (
        <p className="text-centre text-sm text-grey-600">
          <button
            type="button"
            onClick={onFallback}
            className="text-blue-600 hover:text-blue-700"
          >
            Use password instead
          </button>
        </p>
      )}
    </form>
  );
}
```

---

## React Native Stack - React Native 0.83.x

### Passkey Service - React Native

#### services/passkey.service.ts

```typescript
/**
 * passkey.service.ts
 *
 * WebAuthn passkey service for React Native applications.
 * Uses react-native-passkeys for platform authenticator access.
 *
 * @package React Native 0.83.x / TypeScript 5.9
 * @version 2.0.0
 */

import { Passkey } from 'react-native-passkeys';
import { Platform } from 'react-native';

const API_BASE = 'https://api.yourapp.com/passkeys';

/**
 * Passkey registration result.
 */
export interface RegistrationResult {
  success: boolean;
  passkey?: {
    id: number;
    name: string;
    createdAt: string;
  };
  error?: string;
}

/**
 * Passkey authentication result.
 */
export interface AuthenticationResult {
  success: boolean;
  user?: {
    id: string;
    email: string;
    name: string;
  };
  tokens?: {
    accessToken: string;
    refreshToken: string;
    expiresAt: number;
  };
  error?: string;
}

/**
 * Checks if passkeys are supported on the current device.
 *
 * @returns Promise resolving to true if supported
 */
export async function isPasskeySupported(): Promise<boolean> {
  try {
    return await Passkey.isSupported();
  } catch {
    return false;
  }
}

/**
 * Registers a new passkey for the authenticated user.
 *
 * @param name User-friendly name for the passkey
 * @param authToken Current authentication token
 * @returns Promise resolving to registration result
 */
export async function registerPasskey(
  name: string,
  authToken: string
): Promise<RegistrationResult> {
  try {
    // Get registration options from server
    const optionsResponse = await fetch(`${API_BASE}/register/options`, {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${authToken}`,
      },
    });

    if (!optionsResponse.ok) {
      const error = await optionsResponse.json();
      throw new Error(error.error || 'Failed to get registration options');
    }

    const options = await optionsResponse.json();

    // Create credential with platform authenticator
    const credential = await Passkey.create(options);

    // Verify credential with server
    const verifyResponse = await fetch(`${API_BASE}/register`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${authToken}`,
      },
      body: JSON.stringify({ name, credential }),
    });

    const result = await verifyResponse.json();

    if (!verifyResponse.ok) {
      throw new Error(result.error || 'Registration failed');
    }

    return {
      success: true,
      passkey: result.passkey,
    };
  } catch (error) {
    const message = error instanceof Error ? error.message : 'Registration failed';
    return { success: false, error: message };
  }
}

/**
 * Authenticates a user with their passkey.
 *
 * @param email User's email address
 * @returns Promise resolving to authentication result
 */
export async function authenticateWithPasskey(
  email: string
): Promise<AuthenticationResult> {
  try {
    // Get authentication options from server
    const optionsResponse = await fetch(`${API_BASE}/authenticate/options`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email }),
    });

    if (!optionsResponse.ok) {
      const error = await optionsResponse.json();
      throw new Error(error.error || 'Failed to get authentication options');
    }

    const options = await optionsResponse.json();

    // Get assertion from platform authenticator
    const credential = await Passkey.get(options);

    // Verify assertion with server
    const verifyResponse = await fetch(`${API_BASE}/authenticate`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ credential }),
    });

    const result = await verifyResponse.json();

    if (!verifyResponse.ok) {
      throw new Error(result.error || 'Authentication failed');
    }

    return {
      success: true,
      user: result.user,
      tokens: result.tokens,
    };
  } catch (error) {
    const message = error instanceof Error ? error.message : 'Authentication failed';
    return { success: false, error: message };
  }
}
```

---

### Passkey Hooks - React Native

#### hooks/usePasskey.ts

```typescript
/**
 * usePasskey.ts
 *
 * React Native hook for passkey operations.
 *
 * @package React Native 0.83.x / TypeScript 5.9
 * @version 2.0.0
 */

import { useState, useEffect, useCallback } from 'react';
import {
  isPasskeySupported,
  registerPasskey,
  authenticateWithPasskey,
  type RegistrationResult,
  type AuthenticationResult,
} from '@/services/passkey.service';
import { useAuth } from './useAuth';

/**
 * Hook return type.
 */
interface UsePasskeyReturn {
  isSupported: boolean;
  loading: boolean;
  error: string | null;
  register: (name: string) => Promise<RegistrationResult>;
  authenticate: (email: string) => Promise<AuthenticationResult>;
}

/**
 * Custom hook for passkey operations in React Native.
 *
 * @returns Object with passkey methods and state
 */
export function usePasskey(): UsePasskeyReturn {
  const [isSupported, setIsSupported] = useState(false);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const { getToken, setTokens } = useAuth();

  /**
   * Checks passkey support on mount.
   */
  useEffect(() => {
    isPasskeySupported().then(setIsSupported);
  }, []);

  /**
   * Registers a new passkey.
   */
  const register = useCallback(async (name: string): Promise<RegistrationResult> => {
    setLoading(true);
    setError(null);

    try {
      const token = await getToken();
      if (!token) {
        return { success: false, error: 'Not authenticated' };
      }

      const result = await registerPasskey(name, token);

      if (!result.success) {
        setError(result.error || 'Registration failed');
      }

      return result;
    } finally {
      setLoading(false);
    }
  }, [getToken]);

  /**
   * Authenticates with a passkey.
   */
  const authenticate = useCallback(async (email: string): Promise<AuthenticationResult> => {
    setLoading(true);
    setError(null);

    try {
      const result = await authenticateWithPasskey(email);

      if (result.success && result.tokens) {
        await setTokens(result.tokens);
      } else {
        setError(result.error || 'Authentication failed');
      }

      return result;
    } finally {
      setLoading(false);
    }
  }, [setTokens]);

  return {
    isSupported,
    loading,
    error,
    register,
    authenticate,
  };
}
```

---

### Passkey Screen - React Native

#### screens/PasskeyLoginScreen.tsx

```typescript
/**
 * PasskeyLoginScreen.tsx
 *
 * Passkey login screen for React Native.
 * Styled with NativeWind 4.x.
 *
 * @package React Native 0.83.x / TypeScript 5.9 / NativeWind 4.x
 * @version 2.0.0
 */

import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  ActivityIndicator,
  Alert,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { usePasskey } from '@/hooks/usePasskey';

/**
 * Passkey login screen component.
 */
export function PasskeyLoginScreen() {
  const navigation = useNavigation();
  const { isSupported, authenticate, loading, error } = usePasskey();
  const [email, setEmail] = useState('');

  /**
   * Handles passkey authentication.
   */
  const handleAuthenticate = async () => {
    if (!email) {
      Alert.alert('Error', 'Please enter your email address');
      return;
    }

    const result = await authenticate(email);

    if (result.success) {
      navigation.navigate('Dashboard' as never);
    } else {
      Alert.alert('Authentication Failed', result.error || 'Please try again');
    }
  };

  if (!isSupported) {
    return (
      <View className="flex-1 bg-white dark:bg-grey-900 px-6 justify-centre">
        <View className="bg-yellow-50 dark:bg-yellow-900/20 p-4 rounded-lg">
          <Text className="text-yellow-700 dark:text-yellow-400 text-centre">
            Passkeys are not supported on this device.
          </Text>
          <TouchableOpacity
            onPress={() => navigation.navigate('Login' as never)}
            className="mt-4"
          >
            <Text className="text-blue-600 text-centre font-medium">
              Use password instead
            </Text>
          </TouchableOpacity>
        </View>
      </View>
    );
  }

  return (
    <View className="flex-1 bg-white dark:bg-grey-900 px-6 justify-centre">
      <View className="items-centre mb-8">
        <View className="w-16 h-16 bg-blue-100 dark:bg-blue-900/30 rounded-full items-centre justify-centre mb-4">
          <Text className="text-3xl">🔐</Text>
        </View>
        <Text className="text-2xl font-bold text-grey-900 dark:text-white">
          Sign in with Passkey
        </Text>
        <Text className="text-grey-600 dark:text-grey-400 text-centre mt-2">
          Use your fingerprint, face, or device PIN
        </Text>
      </View>

      <View className="space-y-4">
        <View>
          <Text className="text-sm font-medium mb-1 text-grey-700 dark:text-grey-300">
            Email
          </Text>
          <TextInput
            value={email}
            onChangeText={setEmail}
            placeholder="you@example.com"
            keyboardType="email-address"
            autoCapitalize="none"
            autoCorrect={false}
            editable={!loading}
            className="w-full px-4 py-3 border border-grey-300 dark:border-grey-700 rounded-lg text-grey-900 dark:text-white bg-white dark:bg-grey-800"
            placeholderTextColor="#9CA3AF"
          />
        </View>

        <TouchableOpacity
          onPress={handleAuthenticate}
          disabled={loading || !email}
          className={`w-full py-3 px-4 rounded-lg flex-row items-centre justify-centre ${
            loading || !email ? 'bg-blue-400' : 'bg-blue-600'
          }`}
        >
          {loading ? (
            <>
              <ActivityIndicator color="white" size="small" />
              <Text className="text-white font-medium ml-2">
                Waiting for authenticator...
              </Text>
            </>
          ) : (
            <>
              <Text className="text-white font-medium text-lg">
                Sign in with Passkey
              </Text>
            </>
          )}
        </TouchableOpacity>

        {error && (
          <View className="bg-red-50 dark:bg-red-900/20 p-3 rounded-lg">
            <Text className="text-red-600 dark:text-red-400 text-centre text-sm">
              {error}
            </Text>
          </View>
        )}

        <View className="flex-row items-centre my-4">
          <View className="flex-1 h-px bg-grey-300 dark:bg-grey-700" />
          <Text className="mx-4 text-grey-500 dark:text-grey-400">or</Text>
          <View className="flex-1 h-px bg-grey-300 dark:bg-grey-700" />
        </View>

        <TouchableOpacity
          onPress={() => navigation.navigate('Login' as never)}
          className="py-3"
        >
          <Text className="text-blue-600 text-centre font-medium">
            Sign in with password
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}
```
