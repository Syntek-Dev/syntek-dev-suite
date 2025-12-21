# Multi-Factor Authentication (MFA)

## Overview

Multi-factor authentication implementations including TOTP (Time-based One-Time Password) and SMS/Email OTP. Includes backup code generation for account recovery across all four technology stacks.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
| **TALL Stack** | Laravel 12.x / PHP 8.4 / MariaDB 12.x |
| **Django Stack** | Django 6.x / Python 3.14 / PostgreSQL 18.x / Strawberry GraphQL |
| **React Stack** | Next.js 16.x / React 19.x / TypeScript 5.9 / Prisma 6.x |
| **Mobile Stack** | React Native 0.83.x / TypeScript 5.9 / NativeWind 4.x / Apollo Client |
| **Dependencies** | pragmarx/google2fa (Laravel), pyotp (Django), otpauth (Node.js) |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [TALL Stack - Laravel 12.x](#tall-stack---laravel-12x)
- [TOTP Implementation - Laravel](#totp-implementation---laravel)
- [SMS/Email OTP Service - Laravel](#smsemail-otp-service---laravel)
- [Django/Wagtail Stack - Django 6.x](#djangowagtail-stack---django-6x)
- [TOTP Implementation - Django](#totp-implementation---django)
- [GraphQL MFA Mutations - Django](#graphql-mfa-mutations---django)
- [React/Next.js Stack - Next.js 16.x](#reactnextjs-stack---nextjs-16x)
- [TOTP Implementation - Next.js](#totp-implementation---nextjs)
- [API Routes - Next.js](#api-routes---nextjs)
- [React Native Stack - React Native 0.83.x](#react-native-stack---react-native-083x)
- [TOTP Hook - React Native](#totp-hook---react-native)
- [MFA Setup Screen - React Native](#mfa-setup-screen---react-native)


## TALL Stack - Laravel 12.x

## TOTP Implementation - Laravel

### app/Http/Controllers/MfaController.php

```php
<?php

/**
 * MfaController.php
 *
 * Handles Multi-Factor Authentication setup, verification, and management.
 * Uses TOTP (Time-based One-Time Password) with Google Authenticator compatibility.
 *
 * @package App\Http\Controllers
 * @version Laravel 12.x / PHP 8.4
 */

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;
use PragmaRX\Google2FA\Google2FA;

class MfaController extends Controller
{
    /**
     * Initiates MFA setup by generating a secret key.
     *
     * Generates a TOTP secret, stores it encrypted on the user account,
     * and returns the secret along with a QR code URL for authenticator app setup.
     * MFA is not enabled until the user verifies a code from their authenticator app.
     *
     * @param Request $request The HTTP request
     * @return \Illuminate\Http\JsonResponse JSON containing secret and QR code URL
     */
    public function enable(Request $request)
    {
        $google2fa = new Google2FA();
        $user = $request->user();

        // Generate secret
        $secret = $google2fa->generateSecretKey();

        // Store encrypted secret (don't enable yet)
        $user->mfa_secret = encrypt($secret);
        $user->save();

        // Generate QR code URL
        $qrCodeUrl = $google2fa->getQRCodeUrl(
            config('app.name'),
            $user->email,
            $secret
        );

        return response()->json([
            'secret' => $secret,
            'qr_code_url' => $qrCodeUrl,
        ]);
    }

    /**
     * Verifies the MFA setup by validating a TOTP code.
     *
     * Validates the provided 6-digit code against the stored secret.
     * On success, enables MFA on the account and generates backup codes.
     *
     * @param Request $request The HTTP request containing the verification code
     * @return \Illuminate\Http\JsonResponse Success message with backup codes or error
     */
    public function verify(Request $request)
    {
        $request->validate(['code' => 'required|string|size:6']);

        $google2fa = new Google2FA();
        $user = $request->user();
        $secret = decrypt($user->mfa_secret);

        if ($google2fa->verifyKey($secret, $request->code)) {
            $user->mfa_enabled = true;
            $user->mfa_verified_at = now();
            $user->save();

            // Generate backup codes
            $backupCodes = $this->generateBackupCodes($user);

            return response()->json([
                'message' => 'MFA enabled successfully',
                'backup_codes' => $backupCodes,
            ]);
        }

        return response()->json(['error' => 'Invalid code'], 422);
    }

    /**
     * Generates and stores backup codes for MFA recovery.
     *
     * Creates 10 unique backup codes in the format XXXXXXXX-XXXXXXXX.
     * Codes are hashed before storage - plain text codes are returned
     * once only for the user to save securely.
     *
     * @param User $user The user to generate backup codes for
     * @return array Array of plain text backup codes (returned once only)
     */
    protected function generateBackupCodes(User $user): array
    {
        $codes = [];
        for ($i = 0; $i < 10; $i++) {
            $codes[] = Str::random(8) . '-' . Str::random(8);
        }

        // Store hashed backup codes
        $user->mfa_backup_codes = array_map(fn($code) => Hash::make($code), $codes);
        $user->save();

        return $codes; // Return plain codes ONCE for user to save
    }
}
```

---

## SMS/Email OTP Service - Laravel

### app/Services/OtpService.php

```php
<?php

/**
 * OtpService.php
 *
 * Handles generation and verification of one-time passwords (OTP) for
 * SMS and email-based authentication. Includes rate limiting and
 * attempt tracking to prevent brute force attacks.
 *
 * @package App\Services
 * @version Laravel 12.x / PHP 8.4
 */

namespace App\Services;

use App\Models\User;
use App\Exceptions\TooManyAttemptsException;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Hash;

class OtpService
{
    /**
     * Generates a new OTP for the specified user and channel.
     *
     * Creates a 6-digit numeric code, hashes it for storage, and sets
     * a 10-minute expiry. Resets the attempt counter for the user.
     *
     * @param User $user The user to generate OTP for
     * @param string $channel The delivery channel ('email' or 'sms')
     * @return string The plain text OTP code to send to the user
     */
    public function generate(User $user, string $channel = 'email'): string
    {
        $otp = str_pad(random_int(0, 999999), 6, '0', STR_PAD_LEFT);

        Cache::put(
            "otp:{$user->id}:{$channel}",
            Hash::make($otp),
            now()->addMinutes(10)
        );

        // Track attempts
        Cache::put(
            "otp_attempts:{$user->id}",
            0,
            now()->addMinutes(30)
        );

        return $otp;
    }

    /**
     * Verifies an OTP code for the specified user and channel.
     *
     * Checks the provided code against the stored hash. Tracks failed
     * attempts and throws an exception after 5 consecutive failures.
     * On success, clears the stored OTP and attempt counter.
     *
     * @param User $user The user attempting verification
     * @param string $otp The OTP code to verify
     * @param string $channel The delivery channel ('email' or 'sms')
     * @return bool True if OTP is valid, false otherwise
     * @throws TooManyAttemptsException When attempt limit exceeded
     */
    public function verify(User $user, string $otp, string $channel = 'email'): bool
    {
        $attempts = Cache::get("otp_attempts:{$user->id}", 0);

        if ($attempts >= 5) {
            throw new TooManyAttemptsException('Too many verification attempts.');
        }

        $storedHash = Cache::get("otp:{$user->id}:{$channel}");

        if (!$storedHash || !Hash::check($otp, $storedHash)) {
            Cache::increment("otp_attempts:{$user->id}");
            return false;
        }

        Cache::forget("otp:{$user->id}:{$channel}");
        Cache::forget("otp_attempts:{$user->id}");

        return true;
    }
}
```

---

## Django/Wagtail Stack - Django 6.x

### TOTP Implementation - Django

#### authentication/services/mfa_service.py

```python
"""
mfa_service.py

Handles Multi-Factor Authentication setup, verification, and management
using TOTP (Time-based One-Time Password) with Google Authenticator compatibility.

@package authentication.services
@version Django 6.x / Python 3.14
"""

import pyotp
import qrcode
import io
import base64
import secrets
from typing import Tuple, List
from django.contrib.auth import get_user_model
from django.core.cache import cache
from django.utils import timezone
from hashlib import sha256

User = get_user_model()


class MfaService:
    """
    Service class for managing Multi-Factor Authentication operations.

    Provides methods for generating TOTP secrets, creating QR codes,
    verifying codes, and managing backup codes for account recovery.
    """

    @staticmethod
    def generate_secret() -> str:
        """
        Generates a new TOTP secret key.

        Creates a base32-encoded secret key compatible with Google Authenticator
        and other TOTP applications. The secret is 32 bytes of random data.

        Returns:
            str: Base32-encoded secret key
        """
        return pyotp.random_base32()

    @staticmethod
    def generate_qr_code(user: User, secret: str) -> str:
        """
        Generates a QR code for authenticator app setup.

        Creates a QR code image containing the TOTP URI for easy scanning
        by authenticator applications. Returns the QR code as a base64-encoded
        PNG image suitable for embedding in HTML.

        Args:
            user: The user account for which to generate the QR code
            secret: The TOTP secret key

        Returns:
            str: Base64-encoded PNG image of the QR code
        """
        totp = pyotp.TOTP(secret)
        provisioning_uri = totp.provisioning_uri(
            name=user.email,
            issuer_name="Your App Name"
        )

        # Generate QR code
        qr = qrcode.QRCode(version=1, box_size=10, border=5)
        qr.add_data(provisioning_uri)
        qr.make(fit=True)

        # Create image
        img = qr.make_image(fill_colour="black", back_colour="white")

        # Convert to base64
        buffer = io.BytesIO()
        img.save(buffer, format='PNG')
        img_str = base64.b64encode(buffer.getvalue()).decode()

        return f"data:image/png;base64,{img_str}"

    @staticmethod
    def verify_totp(secret: str, code: str) -> bool:
        """
        Verifies a TOTP code against the secret.

        Validates the provided 6-digit code using the TOTP algorithm.
        Allows for a window of ±1 time period to account for clock drift.

        Args:
            secret: The TOTP secret key
            code: The 6-digit code to verify

        Returns:
            bool: True if the code is valid, False otherwise
        """
        totp = pyotp.TOTP(secret)
        return totp.verify(code, valid_window=1)

    @staticmethod
    def generate_backup_codes(user: User, count: int = 10) -> List[str]:
        """
        Generates and stores backup codes for MFA recovery.

        Creates unique backup codes in the format XXXXXXXX-XXXXXXXX.
        Codes are hashed using SHA-256 before storage. Plain text codes
        are returned once only for the user to save securely.

        Args:
            user: The user to generate backup codes for
            count: Number of backup codes to generate (default: 10)

        Returns:
            list: Array of plain text backup codes (returned once only)
        """
        codes = []
        hashed_codes = []

        for _ in range(count):
            # Generate 16-character backup code
            code_part1 = secrets.token_hex(4).upper()
            code_part2 = secrets.token_hex(4).upper()
            code = f"{code_part1}-{code_part2}"

            codes.append(code)
            # Hash the code for storage
            hashed_codes.append(sha256(code.encode()).hexdigest())

        # Store hashed codes
        user.mfa_backup_codes = hashed_codes
        user.save(update_fields=['mfa_backup_codes'])

        return codes

    @staticmethod
    def verify_backup_code(user: User, code: str) -> bool:
        """
        Verifies a backup code and removes it if valid.

        Checks the provided backup code against stored hashed codes.
        On successful verification, removes the code from storage to
        prevent reuse.

        Args:
            user: The user attempting verification
            code: The backup code to verify

        Returns:
            bool: True if the code is valid, False otherwise
        """
        if not user.mfa_backup_codes:
            return False

        hashed_input = sha256(code.encode()).hexdigest()

        if hashed_input in user.mfa_backup_codes:
            # Remove used backup code
            user.mfa_backup_codes.remove(hashed_input)
            user.save(update_fields=['mfa_backup_codes'])
            return True

        return False


class OtpService:
    """
    Service class for managing SMS/Email OTP operations.

    Provides methods for generating and verifying one-time passwords
    with rate limiting and attempt tracking.
    """

    @staticmethod
    def generate_otp(user: User, channel: str = 'email') -> str:
        """
        Generates a new OTP for the specified user and channel.

        Creates a 6-digit numeric code and stores it in cache with a
        10-minute expiry. Resets the attempt counter for the user.

        Args:
            user: The user to generate OTP for
            channel: The delivery channel ('email' or 'sms')

        Returns:
            str: The plain text OTP code to send to the user
        """
        # Generate 6-digit OTP
        otp = f"{secrets.randbelow(1000000):06d}"

        # Store hashed OTP in cache for 10 minutes
        cache_key = f"otp:{user.id}:{channel}"
        hashed_otp = sha256(otp.encode()).hexdigest()
        cache.set(cache_key, hashed_otp, timeout=600)  # 10 minutes

        # Reset attempt counter
        attempt_key = f"otp_attempts:{user.id}"
        cache.set(attempt_key, 0, timeout=1800)  # 30 minutes

        return otp

    @staticmethod
    def verify_otp(user: User, otp: str, channel: str = 'email') -> bool:
        """
        Verifies an OTP code for the specified user and channel.

        Checks the provided code against the stored hash. Tracks failed
        attempts and rejects verification after 5 consecutive failures.
        On success, clears the stored OTP and attempt counter.

        Args:
            user: The user attempting verification
            otp: The OTP code to verify
            channel: The delivery channel ('email' or 'sms')

        Returns:
            bool: True if OTP is valid, False otherwise

        Raises:
            ValueError: When attempt limit exceeded
        """
        attempt_key = f"otp_attempts:{user.id}"
        attempts = cache.get(attempt_key, 0)

        if attempts >= 5:
            raise ValueError('Too many verification attempts. Please request a new code.')

        cache_key = f"otp:{user.id}:{channel}"
        stored_hash = cache.get(cache_key)

        if not stored_hash:
            return False

        hashed_input = sha256(otp.encode()).hexdigest()

        if hashed_input != stored_hash:
            # Increment attempt counter
            cache.set(attempt_key, attempts + 1, timeout=1800)
            return False

        # Valid OTP - clear cache
        cache.delete(cache_key)
        cache.delete(attempt_key)

        return True
```

### GraphQL MFA Mutations - Django

#### authentication/graphql/mutations/mfa_mutations.py

```python
"""
mfa_mutations.py

GraphQL mutations for Multi-Factor Authentication operations using Strawberry GraphQL.

@package authentication.graphql.mutations
@version Django 6.x / Python 3.14 / Strawberry GraphQL
"""

import strawberry
from typing import List, Optional
from django.contrib.auth import get_user_model
from authentication.services.mfa_service import MfaService, OtpService
from authentication.graphql.types import MfaSetupType, MfaVerifyType, OtpResponseType

User = get_user_model()


@strawberry.type
class MfaMutation:
    """
    GraphQL mutations for Multi-Factor Authentication operations.

    Provides mutations for enabling MFA, verifying codes, generating
    backup codes, and managing OTP authentication.
    """

    @strawberry.mutation
    def enable_mfa(self, info) -> MfaSetupType:
        """
        Initiates MFA setup by generating a secret key and QR code.

        Generates a TOTP secret, stores it encrypted on the user account,
        and returns the secret along with a QR code for authenticator app setup.
        MFA is not enabled until the user verifies a code from their app.

        Args:
            info: GraphQL resolve info containing request and user context

        Returns:
            MfaSetupType: Object containing secret and QR code data URL
        """
        user = info.context.request.user

        if not user.is_authenticated:
            raise ValueError('Authentication required')

        # Generate secret
        secret = MfaService.generate_secret()

        # Store encrypted secret (don't enable MFA yet)
        user.mfa_secret = secret  # Encrypt in model save method
        user.mfa_enabled = False
        user.save()

        # Generate QR code
        qr_code_url = MfaService.generate_qr_code(user, secret)

        return MfaSetupType(
            secret=secret,
            qr_code_url=qr_code_url
        )

    @strawberry.mutation
    def verify_mfa(self, code: str, info) -> MfaVerifyType:
        """
        Verifies the MFA setup by validating a TOTP code.

        Validates the provided 6-digit code against the stored secret.
        On success, enables MFA on the account and generates backup codes.

        Args:
            code: The 6-digit TOTP code from authenticator app
            info: GraphQL resolve info containing request and user context

        Returns:
            MfaVerifyType: Object containing success status and backup codes
        """
        user = info.context.request.user

        if not user.is_authenticated:
            raise ValueError('Authentication required')

        if not user.mfa_secret:
            raise ValueError('MFA setup not initiated')

        # Verify TOTP code
        if not MfaService.verify_totp(user.mfa_secret, code):
            raise ValueError('Invalid verification code')

        # Enable MFA
        user.mfa_enabled = True
        user.mfa_verified_at = timezone.now()
        user.save()

        # Generate backup codes
        backup_codes = MfaService.generate_backup_codes(user)

        return MfaVerifyType(
            success=True,
            backup_codes=backup_codes,
            message='MFA enabled successfully'
        )

    @strawberry.mutation
    def disable_mfa(self, code: str, info) -> strawberry.scalars.JSON:
        """
        Disables MFA for the user account.

        Requires verification of a TOTP code or backup code before
        disabling MFA to prevent unauthorised disabling.

        Args:
            code: TOTP code or backup code for verification
            info: GraphQL resolve info containing request and user context

        Returns:
            JSON: Success status and message
        """
        user = info.context.request.user

        if not user.is_authenticated:
            raise ValueError('Authentication required')

        if not user.mfa_enabled:
            raise ValueError('MFA is not enabled')

        # Verify code (TOTP or backup)
        is_valid = (
            MfaService.verify_totp(user.mfa_secret, code) or
            MfaService.verify_backup_code(user, code)
        )

        if not is_valid:
            raise ValueError('Invalid verification code')

        # Disable MFA
        user.mfa_enabled = False
        user.mfa_secret = None
        user.mfa_backup_codes = []
        user.save()

        return {
            'success': True,
            'message': 'MFA disabled successfully'
        }

    @strawberry.mutation
    def generate_otp(self, channel: str = 'email', info) -> OtpResponseType:
        """
        Generates and sends an OTP to the user.

        Creates a 6-digit OTP and sends it via the specified channel
        (email or SMS). The OTP is valid for 10 minutes.

        Args:
            channel: Delivery channel ('email' or 'sms')
            info: GraphQL resolve info containing request and user context

        Returns:
            OtpResponseType: Object containing success status and message
        """
        user = info.context.request.user

        if not user.is_authenticated:
            raise ValueError('Authentication required')

        if channel not in ['email', 'sms']:
            raise ValueError('Invalid channel. Must be "email" or "sms"')

        # Generate OTP
        otp = OtpService.generate_otp(user, channel)

        # TODO: Send OTP via email/SMS service
        # send_otp_email(user.email, otp) or send_otp_sms(user.phone, otp)

        return OtpResponseType(
            success=True,
            message=f'OTP sent to your {channel}'
        )

    @strawberry.mutation
    def verify_otp(self, code: str, channel: str = 'email', info) -> strawberry.scalars.JSON:
        """
        Verifies an OTP code.

        Validates the provided OTP code. Tracks failed attempts and
        throws an error after 5 consecutive failures.

        Args:
            code: The 6-digit OTP code
            channel: Delivery channel ('email' or 'sms')
            info: GraphQL resolve info containing request and user context

        Returns:
            JSON: Success status and message
        """
        user = info.context.request.user

        if not user.is_authenticated:
            raise ValueError('Authentication required')

        try:
            is_valid = OtpService.verify_otp(user, code, channel)
        except ValueError as e:
            raise ValueError(str(e))

        if not is_valid:
            raise ValueError('Invalid or expired OTP code')

        return {
            'success': True,
            'message': 'OTP verified successfully'
        }
```

---

## React/Next.js Stack - Next.js 16.x

### TOTP Implementation - Next.js

#### lib/auth/mfa-service.ts

```typescript
/**
 * mfa-service.ts
 *
 * Multi-Factor Authentication service for Next.js applications.
 * Handles TOTP generation, verification, and backup code management
 * using Node.js crypto and otpauth libraries.
 *
 * @package lib/auth
 * @version Next.js 16.x / React 19.x / TypeScript 5.9
 */

import { createHmac, randomBytes } from 'crypto';
import { TOTP } from 'otpauth';
import QRCode from 'qrcode';
import { prisma } from '@/lib/prisma';

/**
 * Interface for MFA setup response
 */
export interface MfaSetupResponse {
  secret: string;
  qrCodeUrl: string;
  backupCodes?: string[];
}

/**
 * Interface for verification response
 */
export interface VerificationResponse {
  success: boolean;
  backupCodes?: string[];
  message?: string;
}

/**
 * Service class for Multi-Factor Authentication operations.
 *
 * Provides methods for generating TOTP secrets, creating QR codes,
 * verifying codes, and managing backup codes for account recovery.
 */
export class MfaService {
  /**
   * Generates a new TOTP secret for MFA setup.
   *
   * Creates a random 32-byte secret encoded as base32, compatible
   * with Google Authenticator and other TOTP applications.
   *
   * @returns Base32-encoded secret string
   */
  static generateSecret(): string {
    const buffer = randomBytes(32);
    return this.base32Encode(buffer);
  }

  /**
   * Encodes a buffer to base32 format.
   *
   * @param buffer - The buffer to encode
   * @returns Base32-encoded string
   */
  private static base32Encode(buffer: Buffer): string {
    const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ234567';
    let bits = 0;
    let value = 0;
    let output = '';

    for (let i = 0; i < buffer.length; i++) {
      value = (value << 8) | buffer[i];
      bits += 8;

      while (bits >= 5) {
        output += alphabet[(value >>> (bits - 5)) & 31];
        bits -= 5;
      }
    }

    if (bits > 0) {
      output += alphabet[(value << (5 - bits)) & 31];
    }

    return output;
  }

  /**
   * Generates a QR code for authenticator app setup.
   *
   * Creates a data URL containing a QR code image for the TOTP URI.
   * The QR code can be scanned by authenticator apps for easy setup.
   *
   * @param email - User's email address
   * @param secret - The TOTP secret
   * @param issuer - Application name (default: 'Your App')
   * @returns Promise resolving to data URL of QR code image
   */
  static async generateQrCode(
    email: string,
    secret: string,
    issuer: string = 'Your App'
  ): Promise<string> {
    const totp = new TOTP({
      issuer,
      label: email,
      algorithm: 'SHA1',
      digits: 6,
      period: 30,
      secret,
    });

    const uri = totp.toString();
    return QRCode.toDataURL(uri);
  }

  /**
   * Verifies a TOTP code against the secret.
   *
   * Validates the provided 6-digit code using the TOTP algorithm.
   * Allows for a window of ±1 time period to account for clock drift.
   *
   * @param secret - The TOTP secret key
   * @param code - The 6-digit code to verify
   * @returns True if the code is valid, false otherwise
   */
  static verifyTotp(secret: string, code: string): boolean {
    const totp = new TOTP({
      algorithm: 'SHA1',
      digits: 6,
      period: 30,
      secret,
    });

    // Verify with a window of ±1 period (90 seconds total)
    const delta = totp.validate({ token: code, window: 1 });
    return delta !== null;
  }

  /**
   * Generates backup codes for MFA recovery.
   *
   * Creates unique backup codes in the format XXXXXXXX-XXXXXXXX.
   * Codes are hashed using SHA-256 before storage in the database.
   *
   * @param userId - The user ID to generate backup codes for
   * @param count - Number of backup codes to generate (default: 10)
   * @returns Promise resolving to array of plain text backup codes
   */
  static async generateBackupCodes(
    userId: string,
    count: number = 10
  ): Promise<string[]> {
    const codes: string[] = [];
    const hashedCodes: string[] = [];

    for (let i = 0; i < count; i++) {
      // Generate 16-character backup code
      const part1 = randomBytes(4).toString('hex').toUpperCase();
      const part2 = randomBytes(4).toString('hex').toUpperCase();
      const code = `${part1}-${part2}`;

      codes.push(code);

      // Hash the code for storage
      const hash = createHmac('sha256', process.env.MFA_SECRET_KEY!)
        .update(code)
        .digest('hex');

      hashedCodes.push(hash);
    }

    // Store hashed codes in database
    await prisma.user.update({
      where: { id: userId },
      data: { mfaBackupCodes: hashedCodes },
    });

    return codes;
  }

  /**
   * Verifies a backup code and removes it if valid.
   *
   * Checks the provided backup code against stored hashed codes.
   * On successful verification, removes the code from storage to
   * prevent reuse.
   *
   * @param userId - The user ID attempting verification
   * @param code - The backup code to verify
   * @returns Promise resolving to true if valid, false otherwise
   */
  static async verifyBackupCode(
    userId: string,
    code: string
  ): Promise<boolean> {
    const user = await prisma.user.findUnique({
      where: { id: userId },
      select: { mfaBackupCodes: true },
    });

    if (!user?.mfaBackupCodes || user.mfaBackupCodes.length === 0) {
      return false;
    }

    const hash = createHmac('sha256', process.env.MFA_SECRET_KEY!)
      .update(code)
      .digest('hex');

    const codeIndex = user.mfaBackupCodes.indexOf(hash);

    if (codeIndex === -1) {
      return false;
    }

    // Remove used backup code
    const updatedCodes = user.mfaBackupCodes.filter((_, i) => i !== codeIndex);

    await prisma.user.update({
      where: { id: userId },
      data: { mfaBackupCodes: updatedCodes },
    });

    return true;
  }
}

/**
 * Service class for managing SMS/Email OTP operations.
 *
 * Provides methods for generating and verifying one-time passwords
 * with Redis-based caching and rate limiting.
 */
export class OtpService {
  /**
   * Generates a new OTP for the specified user and channel.
   *
   * Creates a 6-digit numeric code and stores it in Redis with a
   * 10-minute expiry. Resets the attempt counter for the user.
   *
   * @param userId - The user ID to generate OTP for
   * @param channel - The delivery channel ('email' or 'sms')
   * @returns Promise resolving to the plain text OTP code
   */
  static async generateOtp(
    userId: string,
    channel: 'email' | 'sms' = 'email'
  ): Promise<string> {
    // Generate 6-digit OTP
    const otp = Math.floor(100000 + Math.random() * 900000).toString();

    // Hash OTP for storage
    const hash = createHmac('sha256', process.env.OTP_SECRET_KEY!)
      .update(otp)
      .digest('hex');

    // Store in Redis/cache (pseudo-code - use your cache implementation)
    // await redis.setex(`otp:${userId}:${channel}`, 600, hash);
    // await redis.setex(`otp_attempts:${userId}`, 1800, '0');

    return otp;
  }

  /**
   * Verifies an OTP code for the specified user and channel.
   *
   * Checks the provided code against the cached hash. Tracks failed
   * attempts and throws an error after 5 consecutive failures.
   * On success, clears the cached OTP and attempt counter.
   *
   * @param userId - The user ID attempting verification
   * @param otp - The OTP code to verify
   * @param channel - The delivery channel ('email' or 'sms')
   * @returns Promise resolving to true if valid, false otherwise
   * @throws Error when attempt limit exceeded
   */
  static async verifyOtp(
    userId: string,
    otp: string,
    channel: 'email' | 'sms' = 'email'
  ): Promise<boolean> {
    // Get attempt count (pseudo-code)
    // const attempts = parseInt(await redis.get(`otp_attempts:${userId}`) || '0');

    const attempts = 0; // Replace with actual Redis call

    if (attempts >= 5) {
      throw new Error('Too many verification attempts. Please request a new code.');
    }

    // Get stored hash (pseudo-code)
    // const storedHash = await redis.get(`otp:${userId}:${channel}`);

    const storedHash = null; // Replace with actual Redis call

    if (!storedHash) {
      return false;
    }

    const hash = createHmac('sha256', process.env.OTP_SECRET_KEY!)
      .update(otp)
      .digest('hex');

    if (hash !== storedHash) {
      // Increment attempt counter (pseudo-code)
      // await redis.incr(`otp_attempts:${userId}`);
      return false;
    }

    // Valid OTP - clear cache (pseudo-code)
    // await redis.del(`otp:${userId}:${channel}`);
    // await redis.del(`otp_attempts:${userId}`);

    return true;
  }
}
```

### API Routes - Next.js

#### app/api/auth/mfa/enable/route.ts

```typescript
/**
 * app/api/auth/mfa/enable/route.ts
 *
 * API route for enabling Multi-Factor Authentication.
 * Generates TOTP secret and QR code for authenticator app setup.
 *
 * @package app/api/auth/mfa
 * @version Next.js 16.x / React 19.x / TypeScript 5.9
 */

import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth/next';
import { authOptions } from '@/lib/auth/auth-options';
import { MfaService } from '@/lib/auth/mfa-service';
import { prisma } from '@/lib/prisma';

/**
 * Handles POST requests to enable MFA for the authenticated user.
 *
 * Generates a new TOTP secret, stores it encrypted in the database,
 * and returns the secret along with a QR code for app setup.
 * MFA is not enabled until the user verifies a code.
 *
 * @param request - The Next.js request object
 * @returns JSON response containing secret and QR code URL
 */
export async function POST(request: NextRequest) {
  try {
    // Get authenticated user
    const session = await getServerSession(authOptions);

    if (!session?.user?.email) {
      return NextResponse.json(
        { error: 'Unauthorised' },
        { status: 401 }
      );
    }

    const user = await prisma.user.findUnique({
      where: { email: session.user.email },
    });

    if (!user) {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      );
    }

    // Generate TOTP secret
    const secret = MfaService.generateSecret();

    // Generate QR code
    const qrCodeUrl = await MfaService.generateQrCode(
      user.email,
      secret,
      process.env.NEXT_PUBLIC_APP_NAME || 'Your App'
    );

    // Store encrypted secret (don't enable MFA yet)
    await prisma.user.update({
      where: { id: user.id },
      data: {
        mfaSecret: secret, // Encrypt in Prisma middleware
        mfaEnabled: false,
      },
    });

    return NextResponse.json({
      secret,
      qrCodeUrl,
    });
  } catch (error) {
    console.error('MFA enable error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

#### app/api/auth/mfa/verify/route.ts

```typescript
/**
 * app/api/auth/mfa/verify/route.ts
 *
 * API route for verifying MFA setup with a TOTP code.
 * Enables MFA on the account and generates backup codes.
 *
 * @package app/api/auth/mfa
 * @version Next.js 16.x / React 19.x / TypeScript 5.9
 */

import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth/next';
import { authOptions } from '@/lib/auth/auth-options';
import { MfaService } from '@/lib/auth/mfa-service';
import { prisma } from '@/lib/prisma';

/**
 * Handles POST requests to verify MFA setup.
 *
 * Validates the provided TOTP code against the stored secret.
 * On success, enables MFA and generates backup codes for recovery.
 *
 * @param request - The Next.js request object containing the code
 * @returns JSON response with success status and backup codes
 */
export async function POST(request: NextRequest) {
  try {
    // Get authenticated user
    const session = await getServerSession(authOptions);

    if (!session?.user?.email) {
      return NextResponse.json(
        { error: 'Unauthorised' },
        { status: 401 }
      );
    }

    const { code } = await request.json();

    if (!code || code.length !== 6) {
      return NextResponse.json(
        { error: 'Invalid code format' },
        { status: 400 }
      );
    }

    const user = await prisma.user.findUnique({
      where: { email: session.user.email },
    });

    if (!user?.mfaSecret) {
      return NextResponse.json(
        { error: 'MFA setup not initiated' },
        { status: 400 }
      );
    }

    // Verify TOTP code
    const isValid = MfaService.verifyTotp(user.mfaSecret, code);

    if (!isValid) {
      return NextResponse.json(
        { error: 'Invalid verification code' },
        { status: 422 }
      );
    }

    // Enable MFA
    await prisma.user.update({
      where: { id: user.id },
      data: {
        mfaEnabled: true,
        mfaVerifiedAt: new Date(),
      },
    });

    // Generate backup codes
    const backupCodes = await MfaService.generateBackupCodes(user.id);

    return NextResponse.json({
      success: true,
      message: 'MFA enabled successfully',
      backupCodes,
    });
  } catch (error) {
    console.error('MFA verification error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

---

## React Native Stack - React Native 0.83.x

### TOTP Hook - React Native

#### hooks/useMfa.ts

```typescript
/**
 * useMfa.ts
 *
 * React Native hook for Multi-Factor Authentication operations.
 * Provides methods for TOTP verification and backup code management
 * using Apollo Client for GraphQL communication.
 *
 * @package hooks
 * @version React Native 0.83.x / TypeScript 5.9 / Apollo Client
 */

import { useState } from 'react';
import { useMutation } from '@apollo/client';
import { gql } from '@apollo/client';
import * as Crypto from 'expo-crypto';
import { Alert } from 'react-native';

/**
 * GraphQL mutation for enabling MFA
 */
const ENABLE_MFA = gql`
  mutation EnableMfa {
    enableMfa {
      secret
      qrCodeUrl
    }
  }
`;

/**
 * GraphQL mutation for verifying MFA setup
 */
const VERIFY_MFA = gql`
  mutation VerifyMfa($code: String!) {
    verifyMfa(code: $code) {
      success
      backupCodes
      message
    }
  }
`;

/**
 * GraphQL mutation for disabling MFA
 */
const DISABLE_MFA = gql`
  mutation DisableMfa($code: String!) {
    disableMfa(code: $code) {
      success
      message
    }
  }
`;

/**
 * Interface for MFA setup data
 */
interface MfaSetup {
  secret: string;
  qrCodeUrl: string;
}

/**
 * Interface for MFA verification result
 */
interface MfaVerification {
  success: boolean;
  backupCodes?: string[];
  message?: string;
}

/**
 * Interface for the hook's return value
 */
interface UseMfaReturn {
  setupData: MfaSetup | null;
  backupCodes: string[] | null;
  loading: boolean;
  error: string | null;
  enableMfa: () => Promise<void>;
  verifyMfa: (code: string) => Promise<boolean>;
  disableMfa: (code: string) => Promise<boolean>;
  generateLocalTotp: (secret: string) => Promise<string>;
}

/**
 * Custom hook for Multi-Factor Authentication operations.
 *
 * Provides methods for setting up, verifying, and disabling MFA
 * using GraphQL mutations. Includes client-side TOTP generation
 * for testing and verification.
 *
 * @returns Object containing MFA methods and state
 */
export function useMfa(): UseMfaReturn {
  const [setupData, setSetupData] = useState<MfaSetup | null>(null);
  const [backupCodes, setBackupCodes] = useState<string[] | null>(null);
  const [error, setError] = useState<string | null>(null);

  const [enableMfaMutation, { loading: enableLoading }] = useMutation(ENABLE_MFA);
  const [verifyMfaMutation, { loading: verifyLoading }] = useMutation(VERIFY_MFA);
  const [disableMfaMutation, { loading: disableLoading }] = useMutation(DISABLE_MFA);

  const loading = enableLoading || verifyLoading || disableLoading;

  /**
   * Initiates MFA setup by requesting a new TOTP secret.
   *
   * Calls the GraphQL mutation to generate a secret and QR code.
   * The setup data is stored in state for display to the user.
   *
   * @throws Error if the mutation fails
   */
  const enableMfa = async (): Promise<void> => {
    try {
      setError(null);
      const { data } = await enableMfaMutation();

      if (data?.enableMfa) {
        setSetupData(data.enableMfa);
      }
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Failed to enable MFA';
      setError(errorMessage);
      Alert.alert('Error', errorMessage);
      throw err;
    }
  };

  /**
   * Verifies the MFA setup with a TOTP code.
   *
   * Validates the provided code and enables MFA if valid.
   * Backup codes are stored in state for display to the user.
   *
   * @param code - The 6-digit TOTP code from authenticator app
   * @returns Promise resolving to true if verification succeeded
   * @throws Error if the mutation fails
   */
  const verifyMfa = async (code: string): Promise<boolean> => {
    try {
      setError(null);
      const { data } = await verifyMfaMutation({
        variables: { code },
      });

      if (data?.verifyMfa?.success) {
        setBackupCodes(data.verifyMfa.backupCodes || []);
        setSetupData(null);
        return true;
      }

      return false;
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Invalid verification code';
      setError(errorMessage);
      Alert.alert('Error', errorMessage);
      return false;
    }
  };

  /**
   * Disables MFA for the user account.
   *
   * Requires verification with a TOTP code or backup code.
   * Clears all MFA-related state on success.
   *
   * @param code - TOTP code or backup code for verification
   * @returns Promise resolving to true if disabling succeeded
   * @throws Error if the mutation fails
   */
  const disableMfa = async (code: string): Promise<boolean> => {
    try {
      setError(null);
      const { data } = await disableMfaMutation({
        variables: { code },
      });

      if (data?.disableMfa?.success) {
        setSetupData(null);
        setBackupCodes(null);
        return true;
      }

      return false;
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Failed to disable MFA';
      setError(errorMessage);
      Alert.alert('Error', errorMessage);
      return false;
    }
  };

  /**
   * Generates a TOTP code locally for testing purposes.
   *
   * Uses expo-crypto to generate a time-based code from the secret.
   * This can be used to verify the secret is working correctly.
   *
   * Note: For production use, users should use a dedicated authenticator app.
   *
   * @param secret - The base32-encoded TOTP secret
   * @returns Promise resolving to the 6-digit TOTP code
   */
  const generateLocalTotp = async (secret: string): Promise<string> => {
    try {
      // Decode base32 secret
      const secretBytes = base32Decode(secret);

      // Get current time step (30-second intervals)
      const timeStep = Math.floor(Date.now() / 1000 / 30);
      const timeBuffer = Buffer.alloc(8);
      timeBuffer.writeBigInt64BE(BigInt(timeStep));

      // Generate HMAC-SHA1
      const hmac = await Crypto.digestStringAsync(
        Crypto.CryptoDigestAlgorithm.SHA1,
        secretBytes.toString('hex') + timeBuffer.toString('hex'),
        { encoding: Crypto.CryptoEncoding.HEX }
      );

      // Extract dynamic binary code
      const hmacBytes = Buffer.from(hmac, 'hex');
      const offset = hmacBytes[hmacBytes.length - 1] & 0x0f;
      const code =
        ((hmacBytes[offset] & 0x7f) << 24) |
        ((hmacBytes[offset + 1] & 0xff) << 16) |
        ((hmacBytes[offset + 2] & 0xff) << 8) |
        (hmacBytes[offset + 3] & 0xff);

      // Generate 6-digit code
      const otp = (code % 1000000).toString().padStart(6, '0');
      return otp;
    } catch (err) {
      console.error('TOTP generation error:', err);
      throw new Error('Failed to generate TOTP code');
    }
  };

  return {
    setupData,
    backupCodes,
    loading,
    error,
    enableMfa,
    verifyMfa,
    disableMfa,
    generateLocalTotp,
  };
}

/**
 * Decodes a base32-encoded string to a Buffer.
 *
 * @param base32 - The base32-encoded string
 * @returns Buffer containing the decoded bytes
 */
function base32Decode(base32: string): Buffer {
  const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ234567';
  const cleanedInput = base32.toUpperCase().replace(/=+$/, '');

  let bits = 0;
  let value = 0;
  const output: number[] = [];

  for (let i = 0; i < cleanedInput.length; i++) {
    const idx = alphabet.indexOf(cleanedInput[i]);
    if (idx === -1) throw new Error('Invalid base32 character');

    value = (value << 5) | idx;
    bits += 5;

    if (bits >= 8) {
      output.push((value >>> (bits - 8)) & 0xff);
      bits -= 8;
    }
  }

  return Buffer.from(output);
}
```

### MFA Setup Screen - React Native

#### screens/MfaSetupScreen.tsx

```typescript
/**
 * MfaSetupScreen.tsx
 *
 * React Native screen for Multi-Factor Authentication setup.
 * Displays QR code for authenticator app scanning and handles
 * code verification with backup code generation.
 *
 * @package screens
 * @version React Native 0.83.x / TypeScript 5.9 / NativeWind 4.x
 */

import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  ScrollView,
  Image,
  Alert,
  Clipboard,
} from 'react-native';
import { useMfa } from '@/hooks/useMfa';

/**
 * MFA Setup Screen Component
 *
 * Guides users through the MFA setup process:
 * 1. Displays QR code for authenticator app
 * 2. Shows manual entry secret key
 * 3. Verifies TOTP code
 * 4. Displays backup codes for safe storage
 *
 * @returns JSX.Element The MFA setup screen
 */
export default function MfaSetupScreen() {
  const {
    setupData,
    backupCodes,
    loading,
    error,
    enableMfa,
    verifyMfa,
  } = useMfa();

  const [verificationCode, setVerificationCode] = useState('');
  const [step, setStep] = useState<'setup' | 'verify' | 'complete'>('setup');

  /**
   * Initialises MFA setup on component mount
   */
  useEffect(() => {
    if (step === 'setup' && !setupData) {
      handleEnableMfa();
    }
  }, [step, setupData]);

  /**
   * Initiates the MFA setup process
   */
  const handleEnableMfa = async () => {
    try {
      await enableMfa();
      setStep('verify');
    } catch (err) {
      console.error('MFA setup error:', err);
    }
  };

  /**
   * Handles TOTP code verification
   */
  const handleVerifyCode = async () => {
    if (verificationCode.length !== 6) {
      Alert.alert('Invalid Code', 'Please enter a 6-digit code');
      return;
    }

    const success = await verifyMfa(verificationCode);

    if (success) {
      setStep('complete');
      setVerificationCode('');
    } else {
      Alert.alert('Verification Failed', 'The code you entered is invalid');
    }
  };

  /**
   * Copies text to clipboard
   */
  const copyToClipboard = (text: string) => {
    Clipboard.setString(text);
    Alert.alert('Copied', 'Copied to clipboard');
  };

  /**
   * Renders the QR code setup step
   */
  const renderSetupStep = () => (
    <View className="flex-1 p-6">
      <Text className="text-2xl font-bold mb-4 text-gray-900 dark:text-white">
        Set Up Two-Factor Authentication
      </Text>

      <Text className="text-base mb-6 text-gray-700 dark:text-gray-300">
        Scan this QR code with your authenticator app (Google Authenticator, Authy, etc.)
      </Text>

      {setupData?.qrCodeUrl && (
        <View className="items-centre mb-6">
          <Image
            source={{ uri: setupData.qrCodeUrl }}
            className="w-64 h-64"
            resizeMode="contain"
          />
        </View>
      )}

      {setupData?.secret && (
        <View className="mb-6">
          <Text className="text-sm font-semibold mb-2 text-gray-700 dark:text-gray-300">
            Or enter this code manually:
          </Text>
          <TouchableOpacity
            onPress={() => copyToClipboard(setupData.secret)}
            className="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg"
          >
            <Text className="font-mono text-centre text-gray-900 dark:text-white">
              {setupData.secret}
            </Text>
          </TouchableOpacity>
          <Text className="text-xs text-centre mt-2 text-gray-500">
            Tap to copy
          </Text>
        </View>
      )}

      <TouchableOpacity
        onPress={() => setStep('verify')}
        disabled={loading}
        className="bg-blue-600 p-4 rounded-lg"
      >
        <Text className="text-white text-centre font-semibold">
          {loading ? 'Loading...' : 'Continue'}
        </Text>
      </TouchableOpacity>
    </View>
  );

  /**
   * Renders the code verification step
   */
  const renderVerifyStep = () => (
    <View className="flex-1 p-6">
      <Text className="text-2xl font-bold mb-4 text-gray-900 dark:text-white">
        Verify Your Code
      </Text>

      <Text className="text-base mb-6 text-gray-700 dark:text-gray-300">
        Enter the 6-digit code from your authenticator app
      </Text>

      <TextInput
        value={verificationCode}
        onChangeText={setVerificationCode}
        placeholder="000000"
        keyboardType="number-pad"
        maxLength={6}
        className="bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-700 rounded-lg p-4 text-centre text-2xl font-mono mb-6 text-gray-900 dark:text-white"
      />

      <TouchableOpacity
        onPress={handleVerifyCode}
        disabled={loading || verificationCode.length !== 6}
        className={`p-4 rounded-lg ${
          loading || verificationCode.length !== 6
            ? 'bg-gray-400'
            : 'bg-blue-600'
        }`}
      >
        <Text className="text-white text-centre font-semibold">
          {loading ? 'Verifying...' : 'Verify Code'}
        </Text>
      </TouchableOpacity>

      <TouchableOpacity
        onPress={() => setStep('setup')}
        className="mt-4 p-4"
      >
        <Text className="text-blue-600 text-centre">
          Back to QR Code
        </Text>
      </TouchableOpacity>
    </View>
  );

  /**
   * Renders the backup codes step
   */
  const renderCompleteStep = () => (
    <ScrollView className="flex-1 p-6">
      <Text className="text-2xl font-bold mb-4 text-gray-900 dark:text-white">
        Backup Codes
      </Text>

      <Text className="text-base mb-6 text-gray-700 dark:text-gray-300">
        Save these backup codes in a safe place. You can use them to access your
        account if you lose your authenticator device.
      </Text>

      <View className="bg-gray-100 dark:bg-gray-800 p-4 rounded-lg mb-6">
        {backupCodes?.map((code, index) => (
          <TouchableOpacity
            key={index}
            onPress={() => copyToClipboard(code)}
            className="py-2"
          >
            <Text className="font-mono text-centre text-gray-900 dark:text-white">
              {code}
            </Text>
          </TouchableOpacity>
        ))}
      </View>

      <Text className="text-sm text-centre mb-6 text-gray-500">
        Tap any code to copy it
      </Text>

      <TouchableOpacity
        onPress={() => {
          // Navigate back or to settings
          Alert.alert('Success', 'Two-factor authentication is now enabled');
        }}
        className="bg-green-600 p-4 rounded-lg"
      >
        <Text className="text-white text-centre font-semibold">
          Done
        </Text>
      </TouchableOpacity>
    </ScrollView>
  );

  return (
    <View className="flex-1 bg-white dark:bg-gray-900">
      {step === 'setup' && renderSetupStep()}
      {step === 'verify' && renderVerifyStep()}
      {step === 'complete' && renderCompleteStep()}
    </View>
  );
}
```
