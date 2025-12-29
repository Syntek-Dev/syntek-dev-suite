# PII Storage Services

**Last Updated**: 29/12/2025
**Version**: 1.3.1
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Metadata

| Property         | Value         |
| ---------------- | ------------- |
| **Version**      | 2.0.0         |
| **Last Updated** | December 2025 |
| **Status**       | Stable        |

## Framework Versions Tested

| Framework    | Version | Tested Date |
| ------------ | ------- | ----------- |
| Laravel      | 12.x    | 20/12/2025  |
| Django       | 6.x     | 20/12/2025  |
| Next.js      | 16.x    | 20/12/2025  |
| React Native | 0.83.x  | 20/12/2025  |
| PHP          | 8.4     | 20/12/2025  |
| Python       | 3.14    | 20/12/2025  |
| Node.js      | 24.x    | 20/12/2025  |
| TypeScript   | 5.9     | 20/12/2025  |
| MariaDB      | 12.x    | 20/12/2025  |
| PostgreSQL   | 18.x    | 20/12/2025  |
| Prisma       | 6.x     | 20/12/2025  |

---

## Table of Contents

- [Metadata](#metadata)
- [Framework Versions Tested](#framework-versions-tested)
- [Table of Contents](#table-of-contents)
- [Overview](#overview)
  - [Security Methods](#security-methods)
- [Laravel (TALL Stack)](#laravel-tall-stack)
  - [PII Storage Service Implementation](#pii-storage-service-implementation)
  - [User Controller Example](#user-controller-example)
- [Django/Wagtail](#djangowagtail)
  - [PII Storage Service Implementation](#pii-storage-service-implementation-1)
  - [Strawberry GraphQL Mutation](#strawberry-graphql-mutation)
- [React/Next.js](#reactnextjs)
  - [PII Storage Service Implementation](#pii-storage-service-implementation-2)
  - [API Route Example](#api-route-example)
- [React Native](#react-native)
  - [GraphQL Client Configuration](#graphql-client-configuration)
  - [Profile Update Hook](#profile-update-hook)
  - [Profile Retrieval Hook](#profile-retrieval-hook)
  - [Usage Example](#usage-example)



## Overview

PII storage services handle secure storage of personally identifiable information by:
1. **HMAC Hashing** data for lookup (irreversible, used for searching)
2. **AES-256 Encrypting** data for storage (reversible, used for display)

**Pattern:** Store both a hash (for lookup) and encrypted value (for retrieval) of PII fields.

### Security Methods
- **Hashing:** HMAC-SHA256 for lookup indices
- **Encryption:** AES-256-GCM for data storage
- **Key Management:** Environment-based key rotation support

---

## Laravel (TALL Stack)

### PII Storage Service Implementation

```php
<?php
/**
 * PiiStorageService.php
 *
 * Service for encrypting and hashing PII data.
 * Uses HMAC-SHA256 for hashing and AES-256-GCM for encryption.
 */

namespace App\Services;

use Illuminate\Support\Facades\Crypt;

class PiiStorageService
{
    /**
     * Hash a value using HMAC-SHA256 for lookup purposes.
     * This is irreversible and used for database indices.
     *
     * @param string $value The value to hash
     * @return string The HMAC-SHA256 hash
     */
    public function hashForLookup(string $value): string
    {
        $key = config('app.pii_hash_key');
        return hash_hmac('sha256', $value, $key);
    }

    /**
     * Encrypt a value using AES-256-GCM.
     * This is reversible and used for storage.
     *
     * @param string $value The value to encrypt
     * @return string The encrypted value
     */
    public function encrypt(string $value): string
    {
        // Laravel's Crypt facade uses AES-256-CBC by default
        // For AES-256-GCM, use OpenSSL directly
        $key = base64_decode(config('app.key'));
        $iv = random_bytes(16);
        $tag = '';

        $encrypted = openssl_encrypt(
            $value,
            'aes-256-gcm',
            $key,
            OPENSSL_RAW_DATA,
            $iv,
            $tag,
            '',
            16
        );

        // Combine IV, tag, and encrypted data
        return base64_encode($iv . $tag . $encrypted);
    }

    /**
     * Decrypt a value that was encrypted using AES-256-GCM.
     *
     * @param string $encrypted The encrypted value
     * @return string The decrypted value
     */
    public function decrypt(string $encrypted): string
    {
        $key = base64_decode(config('app.key'));
        $data = base64_decode($encrypted);

        // Extract IV (16 bytes), tag (16 bytes), and ciphertext
        $iv = substr($data, 0, 16);
        $tag = substr($data, 16, 16);
        $ciphertext = substr($data, 32);

        $decrypted = openssl_decrypt(
            $ciphertext,
            'aes-256-gcm',
            $key,
            OPENSSL_RAW_DATA,
            $iv,
            $tag
        );

        if ($decrypted === false) {
            throw new \RuntimeException('Decryption failed');
        }

        return $decrypted;
    }
}
```

### User Controller Example

```php
<?php
/**
 * UserController.php
 *
 * Handles user account operations via API.
 * All PII is processed through PiiStorageService for proper encryption.
 */

namespace App\Http\Controllers\Api;

use App\Services\PiiStorageService;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Constructor with dependency injection.
     *
     * @param PiiStorageService $piiService The PII storage service
     */
    public function __construct(
        private PiiStorageService $piiService
    ) {}

    /**
     * Updates user profile with PII protection.
     * Encrypts all PII before storage using AES-256-GCM.
     * Creates HMAC-SHA256 hashes for lookup purposes.
     *
     * @param Request $request The incoming request
     * @return \Illuminate\Http\JsonResponse
     */
    public function updateProfile(Request $request)
    {
        $validated = $request->validate([
            'email' => 'sometimes|email',
            'full_name' => 'sometimes|string|max:255',
            'phone' => 'sometimes|string|max:20',
        ]);

        $user = $request->user();

        // Update PII through secure service
        if (isset($validated['email'])) {
            $user->pii()->updateOrCreate(
                ['user_id' => $user->id],
                [
                    'email_hash' => $this->piiService->hashForLookup($validated['email']),
                    'email_encrypted' => $this->piiService->encrypt($validated['email']),
                ]
            );
        }

        if (isset($validated['full_name'])) {
            $user->pii()->updateOrCreate(
                ['user_id' => $user->id],
                [
                    'full_name_encrypted' => $this->piiService->encrypt($validated['full_name']),
                ]
            );
        }

        if (isset($validated['phone'])) {
            $user->pii()->updateOrCreate(
                ['user_id' => $user->id],
                [
                    'phone_hash' => $this->piiService->hashForLookup($validated['phone']),
                    'phone_encrypted' => $this->piiService->encrypt($validated['phone']),
                ]
            );
        }

        return response()->json(['message' => 'Profile updated']);
    }

    /**
     * Retrieves user profile with decrypted PII.
     *
     * @param Request $request The incoming request
     * @return \Illuminate\Http\JsonResponse
     */
    public function getProfile(Request $request)
    {
        $user = $request->user();
        $pii = $user->pii;

        if (!$pii) {
            return response()->json(['message' => 'No profile data found'], 404);
        }

        return response()->json([
            'email' => $pii->email_encrypted ? $this->piiService->decrypt($pii->email_encrypted) : null,
            'full_name' => $pii->full_name_encrypted ? $this->piiService->decrypt($pii->full_name_encrypted) : null,
            'phone' => $pii->phone_encrypted ? $this->piiService->decrypt($pii->phone_encrypted) : null,
        ]);
    }
}
```

---

## Django/Wagtail

### PII Storage Service Implementation

```python
"""
services/pii_storage.py

Service for encrypting and hashing PII data.
Uses HMAC-SHA256 for hashing and AES-256-GCM for encryption.
"""

import hmac
import hashlib
import base64
import os
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
from django.conf import settings


class PiiStorageService:
    """
    Service for secure PII storage using HMAC-SHA256 hashing and AES-256-GCM encryption.

    This service provides methods to:
    - Hash PII data for lookup purposes (irreversible)
    - Encrypt PII data for storage (reversible)
    - Decrypt previously encrypted PII data
    """

    def __init__(self):
        """
        Initialise the PII storage service.

        Retrieves encryption keys from Django settings.
        """
        self.hash_key = settings.PII_HASH_KEY.encode()
        self.encryption_key = base64.b64decode(settings.PII_ENCRYPTION_KEY)

    def hash_for_lookup(self, value: str) -> str:
        """
        Hash a value using HMAC-SHA256 for lookup purposes.

        This is irreversible and used for database indices.

        Args:
            value: The value to hash

        Returns:
            The HMAC-SHA256 hash as a hexadecimal string
        """
        return hmac.new(
            self.hash_key,
            value.encode(),
            hashlib.sha256
        ).hexdigest()

    def encrypt(self, value: str) -> str:
        """
        Encrypt a value using AES-256-GCM.

        This is reversible and used for storage.

        Args:
            value: The value to encrypt

        Returns:
            The encrypted value as a base64-encoded string
        """
        aesgcm = AESGCM(self.encryption_key)
        nonce = os.urandom(12)  # 96-bit nonce for GCM

        ciphertext = aesgcm.encrypt(
            nonce,
            value.encode(),
            None  # No associated data
        )

        # Combine nonce and ciphertext
        combined = nonce + ciphertext
        return base64.b64encode(combined).decode()

    def decrypt(self, encrypted: str) -> str:
        """
        Decrypt a value that was encrypted using AES-256-GCM.

        Args:
            encrypted: The encrypted value as a base64-encoded string

        Returns:
            The decrypted value

        Raises:
            ValueError: If decryption fails
        """
        aesgcm = AESGCM(self.encryption_key)
        combined = base64.b64decode(encrypted)

        # Extract nonce (first 12 bytes) and ciphertext
        nonce = combined[:12]
        ciphertext = combined[12:]

        try:
            plaintext = aesgcm.decrypt(nonce, ciphertext, None)
            return plaintext.decode()
        except Exception as e:
            raise ValueError(f"Decryption failed: {str(e)}")
```

### Strawberry GraphQL Mutation

```python
"""
schema/mutations/user_mutations.py

GraphQL mutations for user account operations.
All PII is processed through PiiStorageService for proper encryption.
"""

import strawberry
from strawberry.types import Info
from typing import Optional

from services.pii_storage import PiiStorageService
from apps.accounts.models import User, UserPii


@strawberry.type
class UpdateProfileResult:
    """Result type for profile update mutation."""
    success: bool
    message: str


@strawberry.type
class UserProfile:
    """User profile data with decrypted PII."""
    email: Optional[str]
    full_name: Optional[str]
    phone: Optional[str]


@strawberry.type
class UserMutations:
    """GraphQL mutations for user account operations."""

    @strawberry.mutation
    def update_profile(
        self,
        info: Info,
        email: Optional[str] = None,
        full_name: Optional[str] = None,
        phone: Optional[str] = None,
    ) -> UpdateProfileResult:
        """
        Updates user profile with PII protection.

        Encrypts all PII before storage using AES-256-GCM.
        Creates HMAC-SHA256 hashes for lookup purposes.

        Args:
            info: GraphQL resolver info containing request context
            email: Optional new email address
            full_name: Optional new full name
            phone: Optional new phone number

        Returns:
            UpdateProfileResult: Success status and message
        """
        request = info.context.get('request')
        user = request.user

        pii_service = PiiStorageService()

        # Update PII through secure service
        if email:
            UserPii.objects.update_or_create(
                user=user,
                defaults={
                    'email_hash': pii_service.hash_for_lookup(email),
                    'email_encrypted': pii_service.encrypt(email),
                }
            )

        if full_name:
            UserPii.objects.filter(user=user).update(
                full_name_encrypted=pii_service.encrypt(full_name)
            )

        if phone:
            UserPii.objects.filter(user=user).update(
                phone_hash=pii_service.hash_for_lookup(phone),
                phone_encrypted=pii_service.encrypt(phone),
            )

        return UpdateProfileResult(success=True, message='Profile updated')


@strawberry.type
class UserQueries:
    """GraphQL queries for user account operations."""

    @strawberry.field
    def profile(self, info: Info) -> Optional[UserProfile]:
        """
        Retrieves user profile with decrypted PII.

        Args:
            info: GraphQL resolver info containing request context

        Returns:
            UserProfile with decrypted PII data, or None if no profile exists
        """
        request = info.context.get('request')
        user = request.user

        try:
            pii = UserPii.objects.get(user=user)
        except UserPii.DoesNotExist:
            return None

        pii_service = PiiStorageService()

        return UserProfile(
            email=pii_service.decrypt(pii.email_encrypted) if pii.email_encrypted else None,
            full_name=pii_service.decrypt(pii.full_name_encrypted) if pii.full_name_encrypted else None,
            phone=pii_service.decrypt(pii.phone_encrypted) if pii.phone_encrypted else None,
        )
```

---

## React/Next.js

### PII Storage Service Implementation

```typescript
/**
 * pii-storage.service.ts
 *
 * Service for encrypting and hashing PII data.
 * Uses HMAC-SHA256 for hashing and AES-256-GCM for encryption.
 */

import crypto from 'crypto';

export class PiiStorageService {
  private hashKey: string;
  private encryptionKey: Buffer;

  /**
   * Initialise the PII storage service.
   * Retrieves encryption keys from environment variables.
   */
  constructor() {
    this.hashKey = process.env.PII_HASH_KEY!;
    this.encryptionKey = Buffer.from(process.env.PII_ENCRYPTION_KEY!, 'base64');

    if (!this.hashKey || !this.encryptionKey) {
      throw new Error('PII encryption keys not configured');
    }
  }

  /**
   * Hash a value using HMAC-SHA256 for lookup purposes.
   * This is irreversible and used for database indices.
   *
   * @param value - The value to hash
   * @returns The HMAC-SHA256 hash as a hexadecimal string
   */
  hashForLookup(value: string): string {
    return crypto
      .createHmac('sha256', this.hashKey)
      .update(value)
      .digest('hex');
  }

  /**
   * Encrypt a value using AES-256-GCM.
   * This is reversible and used for storage.
   *
   * @param value - The value to encrypt
   * @returns The encrypted value as a base64-encoded string
   */
  encrypt(value: string): string {
    // Generate a random 12-byte nonce (96 bits)
    const nonce = crypto.randomBytes(12);

    // Create cipher with AES-256-GCM
    const cipher = crypto.createCipheriv('aes-256-gcm', this.encryptionKey, nonce);

    // Encrypt the value
    let encrypted = cipher.update(value, 'utf8', 'base64');
    encrypted += cipher.final('base64');

    // Get authentication tag
    const authTag = cipher.getAuthTag();

    // Combine nonce, auth tag, and encrypted data
    const combined = Buffer.concat([
      nonce,
      authTag,
      Buffer.from(encrypted, 'base64'),
    ]);

    return combined.toString('base64');
  }

  /**
   * Decrypt a value that was encrypted using AES-256-GCM.
   *
   * @param encrypted - The encrypted value as a base64-encoded string
   * @returns The decrypted value
   * @throws Error if decryption fails
   */
  decrypt(encrypted: string): string {
    const combined = Buffer.from(encrypted, 'base64');

    // Extract nonce (first 12 bytes), auth tag (next 16 bytes), and ciphertext
    const nonce = combined.subarray(0, 12);
    const authTag = combined.subarray(12, 28);
    const ciphertext = combined.subarray(28);

    // Create decipher
    const decipher = crypto.createDecipheriv('aes-256-gcm', this.encryptionKey, nonce);
    decipher.setAuthTag(authTag);

    try {
      let decrypted = decipher.update(ciphertext, undefined, 'utf8');
      decrypted += decipher.final('utf8');
      return decrypted;
    } catch (error) {
      throw new Error(`Decryption failed: ${error}`);
    }
  }
}
```

### API Route Example

```typescript
/**
 * app/api/user/profile/route.ts
 *
 * API route for user profile operations.
 * All PII is processed through PiiStorageService for proper encryption.
 */

import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';
import { PiiStorageService } from '@/services/pii-storage.service';
import { prisma } from '@/lib/prisma';
import { z } from 'zod';

/**
 * Validation schema for profile update request.
 */
const updateProfileSchema = z.object({
  email: z.string().email().optional(),
  fullName: z.string().max(255).optional(),
  phone: z.string().max(20).optional(),
});

/**
 * Updates user profile with PII protection.
 * Encrypts all PII before storage using AES-256-GCM.
 * Creates HMAC-SHA256 hashes for lookup purposes.
 *
 * @param request - The incoming Next.js request
 * @returns JSON response with success status
 */
export async function PATCH(request: NextRequest) {
  try {
    // Authenticate user
    const session = await getServerSession(authOptions);
    if (!session?.user?.id) {
      return NextResponse.json({ error: 'Unauthorised' }, { status: 401 });
    }

    // Parse and validate request body
    const body = await request.json();
    const validated = updateProfileSchema.parse(body);

    const piiService = new PiiStorageService();

    // Update PII through secure service
    const updateData: any = {};

    if (validated.email) {
      updateData.emailHash = piiService.hashForLookup(validated.email);
      updateData.emailEncrypted = piiService.encrypt(validated.email);
    }

    if (validated.fullName) {
      updateData.fullNameEncrypted = piiService.encrypt(validated.fullName);
    }

    if (validated.phone) {
      updateData.phoneHash = piiService.hashForLookup(validated.phone);
      updateData.phoneEncrypted = piiService.encrypt(validated.phone);
    }

    // Update user PII in database using Prisma
    await prisma.userPii.upsert({
      where: { userId: session.user.id },
      update: updateData,
      create: {
        userId: session.user.id,
        ...updateData,
      },
    });

    return NextResponse.json({
      success: true,
      message: 'Profile updated',
    });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Invalid request data', details: error.errors },
        { status: 400 }
      );
    }

    console.error('Profile update error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

/**
 * Retrieves user profile with decrypted PII.
 *
 * @param request - The incoming Next.js request
 * @returns JSON response with decrypted profile data
 */
export async function GET(request: NextRequest) {
  try {
    // Authenticate user
    const session = await getServerSession(authOptions);
    if (!session?.user?.id) {
      return NextResponse.json({ error: 'Unauthorised' }, { status: 401 });
    }

    // Fetch user PII from database
    const pii = await prisma.userPii.findUnique({
      where: { userId: session.user.id },
    });

    if (!pii) {
      return NextResponse.json({ error: 'Profile not found' }, { status: 404 });
    }

    const piiService = new PiiStorageService();

    // Decrypt PII data
    return NextResponse.json({
      email: pii.emailEncrypted ? piiService.decrypt(pii.emailEncrypted) : null,
      fullName: pii.fullNameEncrypted ? piiService.decrypt(pii.fullNameEncrypted) : null,
      phone: pii.phoneEncrypted ? piiService.decrypt(pii.phoneEncrypted) : null,
    });
  } catch (error) {
    console.error('Profile retrieval error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

---

## React Native

React Native applications should **not implement PII encryption/decryption directly**. Instead, they should communicate with the backend GraphQL API to handle all PII operations securely.

### GraphQL Client Configuration

```typescript
/**
 * lib/apollo-client.ts
 *
 * Apollo Client configuration for GraphQL API communication.
 * All PII operations are handled server-side.
 */

import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';
import AsyncStorage from '@react-native-async-storage/async-storage';

/**
 * Create HTTP link to GraphQL API endpoint.
 */
const httpLink = createHttpLink({
  uri: process.env.EXPO_PUBLIC_GRAPHQL_ENDPOINT,
});

/**
 * Create authentication link to attach JWT tokens to requests.
 */
const authLink = setContext(async (_, { headers }) => {
  // Retrieve authentication token from secure storage
  const token = await AsyncStorage.getItem('authToken');

  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  };
});

/**
 * Initialise Apollo Client with authentication and caching.
 * All PII operations are performed server-side via GraphQL mutations/queries.
 */
export const apolloClient = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache(),
});
```

### Profile Update Hook

```typescript
/**
 * hooks/useUpdateProfile.ts
 *
 * Custom hook for updating user profile via GraphQL API.
 * PII encryption is handled server-side.
 */

import { gql, useMutation } from '@apollo/client';

/**
 * GraphQL mutation for updating user profile.
 * Server handles all PII encryption using AES-256-GCM and HMAC-SHA256 hashing.
 */
const UPDATE_PROFILE_MUTATION = gql`
  mutation UpdateProfile($email: String, $fullName: String, $phone: String) {
    updateProfile(email: $email, fullName: $fullName, phone: $phone) {
      success
      message
    }
  }
`;

/**
 * Custom hook to update user profile.
 * All PII is securely encrypted server-side before storage.
 *
 * @returns Mutation function and loading/error states
 */
export function useUpdateProfile() {
  const [updateProfile, { loading, error, data }] = useMutation(UPDATE_PROFILE_MUTATION);

  /**
   * Update user profile with PII data.
   * Server encrypts all PII using AES-256-GCM before storage.
   *
   * @param variables - Profile update data (email, fullName, phone)
   */
  const handleUpdateProfile = async (variables: {
    email?: string;
    fullName?: string;
    phone?: string;
  }) => {
    try {
      await updateProfile({ variables });
    } catch (err) {
      console.error('Profile update error:', err);
    }
  };

  return {
    updateProfile: handleUpdateProfile,
    loading,
    error,
    success: data?.updateProfile?.success,
  };
}
```

### Profile Retrieval Hook

```typescript
/**
 * hooks/useProfile.ts
 *
 * Custom hook for retrieving user profile via GraphQL API.
 * PII decryption is handled server-side.
 */

import { gql, useQuery } from '@apollo/client';

/**
 * GraphQL query for retrieving user profile.
 * Server handles all PII decryption.
 */
const GET_PROFILE_QUERY = gql`
  query GetProfile {
    profile {
      email
      fullName
      phone
    }
  }
`;

/**
 * Custom hook to retrieve user profile.
 * All PII is securely decrypted server-side before being returned.
 *
 * @returns Profile data and loading/error states
 */
export function useProfile() {
  const { data, loading, error, refetch } = useQuery(GET_PROFILE_QUERY);

  return {
    profile: data?.profile,
    loading,
    error,
    refetch,
  };
}
```

### Usage Example

```typescript
/**
 * screens/ProfileScreen.tsx
 *
 * User profile screen demonstrating PII updates via GraphQL API.
 * All encryption/decryption happens server-side.
 */

import React, { useState } from 'react';
import { View, TextInput, Button, Text } from 'react-native';
import { useProfile } from '@/hooks/useProfile';
import { useUpdateProfile } from '@/hooks/useUpdateProfile';

/**
 * Profile screen component.
 * Demonstrates secure PII handling via GraphQL API.
 */
export function ProfileScreen() {
  const { profile, loading: profileLoading, refetch } = useProfile();
  const { updateProfile, loading: updateLoading, success } = useUpdateProfile();

  const [email, setEmail] = useState('');
  const [fullName, setFullName] = useState('');
  const [phone, setPhone] = useState('');

  /**
   * Handle profile update submission.
   * Server encrypts all PII using AES-256-GCM before storage.
   */
  const handleSubmit = async () => {
    await updateProfile({ email, fullName, phone });

    if (success) {
      // Refresh profile data
      refetch();
    }
  };

  if (profileLoading) {
    return <Text>Loading profile...</Text>;
  }

  return (
    <View>
      <TextInput
        placeholder="Email"
        value={email || profile?.email || ''}
        onChangeText={setEmail}
        keyboardType="email-address"
      />
      <TextInput
        placeholder="Full Name"
        value={fullName || profile?.fullName || ''}
        onChangeText={setFullName}
      />
      <TextInput
        placeholder="Phone"
        value={phone || profile?.phone || ''}
        onChangeText={setPhone}
        keyboardType="phone-pad"
      />
      <Button
        title="Update Profile"
        onPress={handleSubmit}
        disabled={updateLoading}
      />
      {success && <Text>Profile updated successfully!</Text>}
    </View>
  );
}
```
