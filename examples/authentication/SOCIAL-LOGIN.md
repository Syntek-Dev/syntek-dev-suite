# Social Login Authentication

**Last Updated**: 29/12/2025
**Version**: 1.3.1
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Overview

Social login authentication implementations for five major providers: Google, Facebook, GitHub, Twitter/X, and Instagram. Each stack uses the recommended authentication library for secure OAuth2/OpenID Connect integration.

## Metadata

| Property            | Value                                          |
| ------------------- | ---------------------------------------------- |
| **Example Version** | 2.0.0                                          |
| **Last Updated**    | 2025-12                                        |
| **TALL Stack**      | Laravel 12.x / PHP 8.4 / Socialite             |
| **Django Stack**    | Django 6.x / Python 3.14 / Allauth             |
| **React Stack**     | Next.js 16.x / React 19.x / NextAuth v5        |
| **Mobile Stack**    | React Native 0.83.x / Expo AuthSession         |
| **Providers**       | Google, Facebook, GitHub, Twitter/X, Instagram |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [Supported Providers](#supported-providers)
- [TALL Stack - Laravel 12.x](#tall-stack---laravel-12x)
  - [Laravel Socialite Configuration](#laravel-socialite-configuration)
    - [config/services.php](#configservicesphp)
    - [.env.example](#envexample)
  - [Social Login Controller - Laravel](#social-login-controller---laravel)
    - [app/Http/Controllers/SocialLoginController.php](#apphttpcontrollerssociallogincontrollerphp)
    - [app/Models/SocialAccount.php](#appmodelssocialaccountphp)
  - [Blade Components - Laravel](#blade-components---laravel)
    - [resources/views/components/social-login-buttons.blade.php](#resourcesviewscomponentssocial-login-buttonsbladephp)
- [Django/Wagtail Stack - Django 6.x](#djangowagtail-stack---django-6x)
  - [Django Allauth Configuration](#django-allauth-configuration)
    - [config/settings/base.py (partial)](#configsettingsbasepy-partial)
  - [Social Login Views - Django](#social-login-views---django)
    - [apps/accounts/views/social\_views.py](#appsaccountsviewssocial_viewspy)
  - [GraphQL Integration - Django](#graphql-integration---django)
    - [apps/accounts/graphql/mutations/social\_mutations.py](#appsaccountsgraphqlmutationssocial_mutationspy)
- [React/Next.js Stack - Next.js 16.x](#reactnextjs-stack---nextjs-16x)
  - [NextAuth Configuration](#nextauth-configuration)
    - [lib/auth/auth.config.ts](#libauthauthconfigts)
    - [auth.ts](#authts)
  - [Social Login Buttons - Next.js](#social-login-buttons---nextjs)
    - [components/SocialLoginButtons.tsx](#componentssocialloginbuttonstsx)
- [React Native Stack - React Native 0.83.x](#react-native-stack---react-native-083x)
  - [Social Auth Configuration - React Native](#social-auth-configuration---react-native)
    - [config/social-auth.config.ts](#configsocial-authconfigts)
  - [Social Login Hooks - React Native](#social-login-hooks---react-native)
    - [hooks/useSocialAuth.ts](#hooksusesocialauthts)
  - [Social Login Screen - React Native](#social-login-screen---react-native)
    - [screens/SocialLoginScreen.tsx](#screenssocialloginscreentsx)

## Supported Providers

| Provider  | OAuth Version              | Scopes Required                        |
| --------- | -------------------------- | -------------------------------------- |
| Google    | OAuth 2.0 / OpenID Connect | `openid email profile`                 |
| Facebook  | OAuth 2.0                  | `email public_profile`                 |
| GitHub    | OAuth 2.0                  | `read:user user:email`                 |
| Twitter/X | OAuth 2.0                  | `users.read tweet.read offline.access` |
| Instagram | OAuth 2.0                  | `user_profile user_media`              |

---

## TALL Stack - Laravel 12.x

### Laravel Socialite Configuration

#### config/services.php

```php
<?php

/**
 * Social authentication provider configuration.
 *
 * Configures OAuth credentials for each supported social login provider.
 * All credentials should be stored in environment variables.
 *
 * @package Laravel 12.x / Socialite
 * @version 2.0.0
 */

return [
    // Google OAuth 2.0 / OpenID Connect
    'google' => [
        'client_id' => env('GOOGLE_CLIENT_ID'),
        'client_secret' => env('GOOGLE_CLIENT_SECRET'),
        'redirect' => env('GOOGLE_REDIRECT_URI', '/auth/google/callback'),
    ],

    // Facebook OAuth 2.0
    'facebook' => [
        'client_id' => env('FACEBOOK_CLIENT_ID'),
        'client_secret' => env('FACEBOOK_CLIENT_SECRET'),
        'redirect' => env('FACEBOOK_REDIRECT_URI', '/auth/facebook/callback'),
    ],

    // GitHub OAuth 2.0
    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => env('GITHUB_REDIRECT_URI', '/auth/github/callback'),
    ],

    // Twitter/X OAuth 2.0
    'twitter' => [
        'client_id' => env('TWITTER_CLIENT_ID'),
        'client_secret' => env('TWITTER_CLIENT_SECRET'),
        'redirect' => env('TWITTER_REDIRECT_URI', '/auth/twitter/callback'),
    ],

    // Instagram OAuth 2.0
    'instagram' => [
        'client_id' => env('INSTAGRAM_CLIENT_ID'),
        'client_secret' => env('INSTAGRAM_CLIENT_SECRET'),
        'redirect' => env('INSTAGRAM_REDIRECT_URI', '/auth/instagram/callback'),
    ],
];
```

#### .env.example

```env
# Google OAuth
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GOOGLE_REDIRECT_URI=/auth/google/callback

# Facebook OAuth
FACEBOOK_CLIENT_ID=your-facebook-app-id
FACEBOOK_CLIENT_SECRET=your-facebook-app-secret
FACEBOOK_REDIRECT_URI=/auth/facebook/callback

# GitHub OAuth
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret
GITHUB_REDIRECT_URI=/auth/github/callback

# Twitter/X OAuth 2.0
TWITTER_CLIENT_ID=your-twitter-client-id
TWITTER_CLIENT_SECRET=your-twitter-client-secret
TWITTER_REDIRECT_URI=/auth/twitter/callback

# Instagram OAuth
INSTAGRAM_CLIENT_ID=your-instagram-client-id
INSTAGRAM_CLIENT_SECRET=your-instagram-client-secret
INSTAGRAM_REDIRECT_URI=/auth/instagram/callback
```

---

### Social Login Controller - Laravel

#### app/Http/Controllers/SocialLoginController.php

```php
<?php

/**
 * SocialLoginController.php
 *
 * Handles OAuth2 social login flow for Google, Facebook, GitHub,
 * Twitter/X, and Instagram. Creates or links user accounts based
 * on email address from OAuth provider.
 *
 * @package App\Http\Controllers
 * @version Laravel 12.x / PHP 8.4
 */

namespace App\Http\Controllers;

use App\Models\User;
use App\Models\SocialAccount;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;
use Laravel\Socialite\Facades\Socialite;
use Laravel\Socialite\Two\InvalidStateException;

class SocialLoginController extends Controller
{
    /**
     * Supported social providers and their scopes.
     */
    protected array $providers = [
        'google' => ['openid', 'email', 'profile'],
        'facebook' => ['email', 'public_profile'],
        'github' => ['read:user', 'user:email'],
        'twitter' => ['users.read', 'tweet.read', 'offline.access'],
        'instagram' => ['user_profile', 'user_media'],
    ];

    /**
     * Redirects to the OAuth provider for authentication.
     *
     * Initiates the OAuth flow by redirecting the user to the provider's
     * authentication page with the configured scopes.
     *
     * @param string $provider The OAuth provider name
     * @return \Illuminate\Http\RedirectResponse Redirect to OAuth provider
     */
    public function redirect(string $provider)
    {
        $this->validateProvider($provider);

        return Socialite::driver($provider)
            ->scopes($this->providers[$provider] ?? [])
            ->redirect();
    }

    /**
     * Handles the OAuth callback from the provider.
     *
     * Receives the OAuth callback, retrieves user data, and either
     * logs in an existing user or creates a new account.
     *
     * @param string $provider The OAuth provider name
     * @return \Illuminate\Http\RedirectResponse Redirect to dashboard or login
     */
    public function callback(string $provider)
    {
        $this->validateProvider($provider);

        try {
            $socialUser = Socialite::driver($provider)->user();
        } catch (InvalidStateException $e) {
            return redirect()->route('login')
                ->with('error', 'Authentication failed. Please try again.');
        }

        // Find or create user
        $user = $this->findOrCreateUser($socialUser, $provider);

        // Log the user in
        Auth::login($user, remember: true);

        // Log the social login event
        activity()
            ->performedOn($user)
            ->log("Logged in via {$provider}");

        return redirect()->route('dashboard')
            ->with('success', 'Welcome back!');
    }

    /**
     * Finds an existing user or creates a new one.
     *
     * First checks for an existing social account link, then tries to
     * match by email. Creates a new user if no match is found.
     *
     * @param mixed $socialUser The user data from OAuth provider
     * @param string $provider The OAuth provider name
     * @return User The user model
     */
    protected function findOrCreateUser($socialUser, string $provider): User
    {
        // Check for existing social account link
        $socialAccount = SocialAccount::where('provider', $provider)
            ->where('provider_id', $socialUser->getId())
            ->first();

        if ($socialAccount) {
            // Update access token
            $socialAccount->update([
                'token' => $socialUser->token,
                'refresh_token' => $socialUser->refreshToken,
            ]);

            return $socialAccount->user;
        }

        // Check for existing user with same email
        $user = User::where('email', $socialUser->getEmail())->first();

        if (!$user) {
            // Create new user
            $user = User::create([
                'name' => $socialUser->getName() ?? $socialUser->getNickname(),
                'email' => $socialUser->getEmail(),
                'password' => Hash::make(Str::random(32)), // Random password for social login
                'email_verified_at' => now(), // Social login emails are pre-verified
                'avatar_url' => $socialUser->getAvatar(),
            ]);
        }

        // Link social account
        $user->socialAccounts()->create([
            'provider' => $provider,
            'provider_id' => $socialUser->getId(),
            'token' => $socialUser->token,
            'refresh_token' => $socialUser->refreshToken,
            'avatar' => $socialUser->getAvatar(),
        ]);

        return $user;
    }

    /**
     * Validates the provider is supported.
     *
     * @param string $provider The OAuth provider name
     * @throws \Illuminate\Validation\ValidationException
     */
    protected function validateProvider(string $provider): void
    {
        if (!array_key_exists($provider, $this->providers)) {
            abort(404, 'Provider not supported');
        }
    }

    /**
     * Unlinks a social account from the user.
     *
     * Removes the social account link but does not delete the user.
     * Requires the user to have a password set before unlinking.
     *
     * @param Request $request The HTTP request
     * @param string $provider The OAuth provider to unlink
     * @return \Illuminate\Http\JsonResponse
     */
    public function unlink(Request $request, string $provider)
    {
        $user = $request->user();

        // Ensure user has password set before unlinking
        if (!$user->password) {
            return response()->json([
                'error' => 'Please set a password before unlinking social accounts.',
            ], 400);
        }

        $socialAccount = $user->socialAccounts()
            ->where('provider', $provider)
            ->first();

        if (!$socialAccount) {
            return response()->json([
                'error' => 'Social account not linked.',
            ], 404);
        }

        $socialAccount->delete();

        return response()->json([
            'message' => "Successfully unlinked {$provider} account.",
        ]);
    }
}
```

#### app/Models/SocialAccount.php

```php
<?php

/**
 * SocialAccount.php
 *
 * Model for storing OAuth social account links. Supports multiple
 * social accounts per user for account linking functionality.
 *
 * @package App\Models
 * @version Laravel 12.x / PHP 8.4
 */

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class SocialAccount extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'user_id',
        'provider',
        'provider_id',
        'token',
        'refresh_token',
        'avatar',
    ];

    /**
     * The attributes that should be hidden for serialisation.
     *
     * @var array<int, string>
     */
    protected $hidden = [
        'token',
        'refresh_token',
    ];

    /**
     * The attributes that should be cast.
     *
     * @var array<string, string>
     */
    protected $casts = [
        'token' => 'encrypted',
        'refresh_token' => 'encrypted',
    ];

    /**
     * Gets the user that owns the social account.
     *
     * @return BelongsTo
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

---

### Blade Components - Laravel

#### resources/views/components/social-login-buttons.blade.php

```blade
{{--
  social-login-buttons.blade.php

  Reusable social login button component for Laravel 12.x / Tailwind 4.x.
  Displays all configured social login options with appropriate icons.
--}}

@props(['providers' => ['google', 'facebook', 'github', 'twitter', 'instagram']])

<div {{ $attributes->merge(['class' => 'space-y-3']) }}>
    <p class="text-centre text-grey-500 text-sm">Or continue with</p>

    <div class="grid grid-cols-5 gap-3">
        @foreach($providers as $provider)
            <a
                href="{{ route('social.redirect', $provider) }}"
                class="flex items-centre justify-centre px-4 py-2 border border-grey-300 rounded-lg hover:bg-grey-50 transition-colours"
                title="Continue with {{ ucfirst($provider) }}"
            >
                @switch($provider)
                    @case('google')
                        {{-- Google icon --}}
                        <svg class="w-5 h-5" viewBox="0 0 24 24" fill="none">
                            <path d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z" fill="#4285F4"/>
                            <path d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z" fill="#34A853"/>
                            <path d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z" fill="#FBBC05"/>
                            <path d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z" fill="#EA4335"/>
                        </svg>
                        @break
                    @case('facebook')
                        {{-- Facebook icon --}}
                        <svg class="w-5 h-5" fill="#1877F2" viewBox="0 0 24 24">
                            <path d="M24 12.073c0-6.627-5.373-12-12-12s-12 5.373-12 12c0 5.99 4.388 10.954 10.125 11.854v-8.385H7.078v-3.47h3.047V9.43c0-3.007 1.792-4.669 4.533-4.669 1.312 0 2.686.235 2.686.235v2.953H15.83c-1.491 0-1.956.925-1.956 1.874v2.25h3.328l-.532 3.47h-2.796v8.385C19.612 23.027 24 18.062 24 12.073z"/>
                        </svg>
                        @break
                    @case('github')
                        {{-- GitHub icon --}}
                        <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
                            <path fill-rule="evenodd" clip-rule="evenodd" d="M12 2C6.477 2 2 6.484 2 12.017c0 4.425 2.865 8.18 6.839 9.504.5.092.682-.217.682-.483 0-.237-.008-.868-.013-1.703-2.782.605-3.369-1.343-3.369-1.343-.454-1.158-1.11-1.466-1.11-1.466-.908-.62.069-.608.069-.608 1.003.07 1.531 1.032 1.531 1.032.892 1.53 2.341 1.088 2.91.832.092-.647.35-1.088.636-1.338-2.22-.253-4.555-1.113-4.555-4.951 0-1.093.39-1.988 1.029-2.688-.103-.253-.446-1.272.098-2.65 0 0 .84-.27 2.75 1.026A9.564 9.564 0 0112 6.844c.85.004 1.705.115 2.504.337 1.909-1.296 2.747-1.027 2.747-1.027.546 1.379.202 2.398.1 2.651.64.7 1.028 1.595 1.028 2.688 0 3.848-2.339 4.695-4.566 4.943.359.309.678.92.678 1.855 0 1.338-.012 2.419-.012 2.747 0 .268.18.58.688.482A10.019 10.019 0 0022 12.017C22 6.484 17.522 2 12 2z"/>
                        </svg>
                        @break
                    @case('twitter')
                        {{-- Twitter/X icon --}}
                        <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
                            <path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/>
                        </svg>
                        @break
                    @case('instagram')
                        {{-- Instagram icon --}}
                        <svg class="w-5 h-5" viewBox="0 0 24 24" fill="url(#instagram-gradient)">
                            <defs>
                                <linearGradient id="instagram-gradient" x1="0%" y1="100%" x2="100%" y2="0%">
                                    <stop offset="0%" style="stop-colour:#FEDA75"/>
                                    <stop offset="25%" style="stop-colour:#FA7E1E"/>
                                    <stop offset="50%" style="stop-colour:#D62976"/>
                                    <stop offset="75%" style="stop-colour:#962FBF"/>
                                    <stop offset="100%" style="stop-colour:#4F5BD5"/>
                                </linearGradient>
                            </defs>
                            <path d="M12 2.163c3.204 0 3.584.012 4.85.07 3.252.148 4.771 1.691 4.919 4.919.058 1.265.069 1.645.069 4.849 0 3.205-.012 3.584-.069 4.849-.149 3.225-1.664 4.771-4.919 4.919-1.266.058-1.644.07-4.85.07-3.204 0-3.584-.012-4.849-.07-3.26-.149-4.771-1.699-4.919-4.92-.058-1.265-.07-1.644-.07-4.849 0-3.204.013-3.583.07-4.849.149-3.227 1.664-4.771 4.919-4.919 1.266-.057 1.645-.069 4.849-.069zm0-2.163c-3.259 0-3.667.014-4.947.072-4.358.2-6.78 2.618-6.98 6.98-.059 1.281-.073 1.689-.073 4.948 0 3.259.014 3.668.072 4.948.2 4.358 2.618 6.78 6.98 6.98 1.281.058 1.689.072 4.948.072 3.259 0 3.668-.014 4.948-.072 4.354-.2 6.782-2.618 6.979-6.98.059-1.28.073-1.689.073-4.948 0-3.259-.014-3.667-.072-4.947-.196-4.354-2.617-6.78-6.979-6.98-1.281-.059-1.69-.073-4.949-.073zm0 5.838c-3.403 0-6.162 2.759-6.162 6.162s2.759 6.163 6.162 6.163 6.162-2.759 6.162-6.163c0-3.403-2.759-6.162-6.162-6.162zm0 10.162c-2.209 0-4-1.79-4-4 0-2.209 1.791-4 4-4s4 1.791 4 4c0 2.21-1.791 4-4 4zm6.406-11.845c-.796 0-1.441.645-1.441 1.44s.645 1.44 1.441 1.44c.795 0 1.439-.645 1.439-1.44s-.644-1.44-1.439-1.44z"/>
                        </svg>
                        @break
                @endswitch
            </a>
        @endforeach
    </div>
</div>
```

---

## Django/Wagtail Stack - Django 6.x

### Django Allauth Configuration

#### config/settings/base.py (partial)

```python
"""
Social authentication configuration for Django 6.x using django-allauth.

Configures OAuth credentials for Google, Facebook, GitHub, Twitter/X,
and Instagram social login providers.

Package: Django 6.x / django-allauth
Version: 2.0.0
"""

INSTALLED_APPS = [
    # ... other apps
    'django.contrib.sites',
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.google',
    'allauth.socialaccount.providers.facebook',
    'allauth.socialaccount.providers.github',
    'allauth.socialaccount.providers.twitter_oauth2',
    'allauth.socialaccount.providers.instagram',
]

SITE_ID = 1

# Allauth configuration
ACCOUNT_EMAIL_REQUIRED = True
ACCOUNT_USERNAME_REQUIRED = False
ACCOUNT_AUTHENTICATION_METHOD = 'email'
ACCOUNT_EMAIL_VERIFICATION = 'optional'  # Social emails are pre-verified
SOCIALACCOUNT_AUTO_SIGNUP = True
SOCIALACCOUNT_EMAIL_VERIFICATION = 'none'  # Trust OAuth provider emails

# Social provider settings
SOCIALACCOUNT_PROVIDERS = {
    'google': {
        'APP': {
            'client_id': env('GOOGLE_CLIENT_ID'),
            'secret': env('GOOGLE_CLIENT_SECRET'),
        },
        'SCOPE': ['openid', 'email', 'profile'],
        'AUTH_PARAMS': {'access_type': 'online'},
    },
    'facebook': {
        'APP': {
            'client_id': env('FACEBOOK_CLIENT_ID'),
            'secret': env('FACEBOOK_CLIENT_SECRET'),
        },
        'SCOPE': ['email', 'public_profile'],
        'METHOD': 'oauth2',
        'VERIFIED_EMAIL': True,
    },
    'github': {
        'APP': {
            'client_id': env('GITHUB_CLIENT_ID'),
            'secret': env('GITHUB_CLIENT_SECRET'),
        },
        'SCOPE': ['read:user', 'user:email'],
    },
    'twitter_oauth2': {
        'APP': {
            'client_id': env('TWITTER_CLIENT_ID'),
            'secret': env('TWITTER_CLIENT_SECRET'),
        },
        'SCOPE': ['users.read', 'tweet.read', 'offline.access'],
    },
    'instagram': {
        'APP': {
            'client_id': env('INSTAGRAM_CLIENT_ID'),
            'secret': env('INSTAGRAM_CLIENT_SECRET'),
        },
        'SCOPE': ['user_profile', 'user_media'],
    },
}
```

---

### Social Login Views - Django

#### apps/accounts/views/social_views.py

```python
"""
Social login views for Django 6.x with Allauth.

Handles OAuth flow, account linking, and social account management
using django-allauth for provider integration.

Package: Django 6.x / django-allauth
Version: 2.0.0
"""

from django.http import JsonResponse
from django.contrib.auth.decorators import login_required
from django.views.decorators.http import require_http_methods
from django.views.decorators.csrf import csrf_protect
from allauth.socialaccount.models import SocialAccount
import json


@login_required
@require_http_methods(["GET"])
def list_social_accounts(request):
    """
    Lists all linked social accounts for the authenticated user.

    Returns a list of connected social providers with their
    provider names and unique identifiers.

    Args:
        request: The HTTP request object

    Returns:
        JsonResponse: List of linked social accounts
    """
    social_accounts = SocialAccount.objects.filter(user=request.user)

    accounts = [
        {
            'provider': account.provider,
            'uid': account.uid,
            'date_joined': account.date_joined.isoformat(),
            'extra_data': {
                'name': account.extra_data.get('name', ''),
                'email': account.extra_data.get('email', ''),
                'picture': account.extra_data.get('picture', ''),
            },
        }
        for account in social_accounts
    ]

    return JsonResponse({
        'accounts': accounts,
        'total': len(accounts),
    })


@login_required
@require_http_methods(["DELETE"])
@csrf_protect
def unlink_social_account(request, provider):
    """
    Unlinks a social account from the user.

    Removes the social account link but does not delete the user.
    Requires the user to have a password set or another social
    account linked before unlinking.

    Args:
        request: The HTTP request object
        provider: The OAuth provider to unlink

    Returns:
        JsonResponse: Success or error message
    """
    user = request.user

    # Check if user has password set or other social accounts
    has_password = user.has_usable_password()
    social_count = SocialAccount.objects.filter(user=user).count()

    if not has_password and social_count <= 1:
        return JsonResponse({
            'error': 'Please set a password before unlinking your only social account.',
        }, status=400)

    # Find and delete the social account
    try:
        social_account = SocialAccount.objects.get(
            user=user,
            provider=provider,
        )
        social_account.delete()

        return JsonResponse({
            'message': f'Successfully unlinked {provider} account.',
        })

    except SocialAccount.DoesNotExist:
        return JsonResponse({
            'error': f'No {provider} account linked.',
        }, status=404)
```

---

### GraphQL Integration - Django

#### apps/accounts/graphql/mutations/social_mutations.py

```python
"""
GraphQL mutations for social authentication using Strawberry GraphQL.

Provides mutations for initiating social login, handling callbacks,
and managing linked social accounts.

Package: Django 6.x / Strawberry GraphQL
Version: 2.0.0
"""

import strawberry
from typing import List, Optional
from strawberry.types import Info
from allauth.socialaccount.models import SocialAccount
from allauth.socialaccount.providers.oauth2.client import OAuth2Client


@strawberry.type
class SocialAccountType:
    """GraphQL type for social account data."""

    provider: str
    uid: str
    date_joined: str
    name: Optional[str] = None
    email: Optional[str] = None
    picture: Optional[str] = None


@strawberry.type
class SocialLoginResult:
    """Result type for social login operations."""

    success: bool
    message: str
    redirect_url: Optional[str] = None


@strawberry.type
class SocialQuery:
    """GraphQL queries for social accounts."""

    @strawberry.field
    def my_social_accounts(self, info: Info) -> List[SocialAccountType]:
        """
        Returns all linked social accounts for the authenticated user.

        Args:
            info: GraphQL resolve info

        Returns:
            List of linked social accounts
        """
        user = info.context.request.user

        if not user.is_authenticated:
            return []

        accounts = SocialAccount.objects.filter(user=user)

        return [
            SocialAccountType(
                provider=account.provider,
                uid=account.uid,
                date_joined=account.date_joined.isoformat(),
                name=account.extra_data.get('name'),
                email=account.extra_data.get('email'),
                picture=account.extra_data.get('picture'),
            )
            for account in accounts
        ]


@strawberry.type
class SocialMutation:
    """GraphQL mutations for social account management."""

    @strawberry.mutation
    def unlink_social_account(
        self,
        info: Info,
        provider: str
    ) -> SocialLoginResult:
        """
        Unlinks a social account from the user.

        Requires the user to have a password set or another social
        account linked before unlinking.

        Args:
            info: GraphQL resolve info
            provider: The OAuth provider to unlink

        Returns:
            SocialLoginResult with success status
        """
        user = info.context.request.user

        if not user.is_authenticated:
            return SocialLoginResult(
                success=False,
                message='Authentication required',
            )

        # Check if user can unlink
        has_password = user.has_usable_password()
        social_count = SocialAccount.objects.filter(user=user).count()

        if not has_password and social_count <= 1:
            return SocialLoginResult(
                success=False,
                message='Please set a password before unlinking your only social account.',
            )

        try:
            social_account = SocialAccount.objects.get(
                user=user,
                provider=provider,
            )
            social_account.delete()

            return SocialLoginResult(
                success=True,
                message=f'Successfully unlinked {provider} account.',
            )

        except SocialAccount.DoesNotExist:
            return SocialLoginResult(
                success=False,
                message=f'No {provider} account linked.',
            )
```

---

## React/Next.js Stack - Next.js 16.x

### NextAuth Configuration

#### lib/auth/auth.config.ts

```typescript
/**
 * auth.config.ts
 *
 * NextAuth v5 configuration for social login providers.
 * Supports Google, Facebook, GitHub, Twitter/X, and Instagram.
 *
 * @package Next.js 16.x / React 19.x / NextAuth v5
 * @version 2.0.0
 */

import type { NextAuthConfig } from 'next-auth';
import Google from 'next-auth/providers/google';
import Facebook from 'next-auth/providers/facebook';
import GitHub from 'next-auth/providers/github';
import Twitter from 'next-auth/providers/twitter';
import Instagram from 'next-auth/providers/instagram';

/**
 * NextAuth configuration with social providers.
 */
export const authConfig: NextAuthConfig = {
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      authorization: {
        params: {
          scope: 'openid email profile',
        },
      },
    }),
    Facebook({
      clientId: process.env.FACEBOOK_CLIENT_ID!,
      clientSecret: process.env.FACEBOOK_CLIENT_SECRET!,
    }),
    GitHub({
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
      authorization: {
        params: {
          scope: 'read:user user:email',
        },
      },
    }),
    Twitter({
      clientId: process.env.TWITTER_CLIENT_ID!,
      clientSecret: process.env.TWITTER_CLIENT_SECRET!,
    }),
    Instagram({
      clientId: process.env.INSTAGRAM_CLIENT_ID!,
      clientSecret: process.env.INSTAGRAM_CLIENT_SECRET!,
    }),
  ],
  callbacks: {
    async signIn({ user, account, profile }) {
      // Allow sign in
      return true;
    },
    async jwt({ token, user, account }) {
      if (account) {
        token.provider = account.provider;
        token.providerAccountId = account.providerAccountId;
      }
      return token;
    },
    async session({ session, token }) {
      if (token.provider) {
        session.provider = token.provider as string;
      }
      return session;
    },
  },
  pages: {
    signIn: '/login',
    error: '/login',
  },
};
```

#### auth.ts

```typescript
/**
 * auth.ts
 *
 * NextAuth v5 export with configured providers.
 *
 * @package Next.js 16.x / React 19.x / NextAuth v5
 * @version 2.0.0
 */

import NextAuth from 'next-auth';
import { authConfig } from './auth.config';

export const { handlers, auth, signIn, signOut } = NextAuth(authConfig);
```

---

### Social Login Buttons - Next.js

#### components/SocialLoginButtons.tsx

```typescript
/**
 * SocialLoginButtons.tsx
 *
 * Reusable social login button component for Next.js 16.x / React 19.x.
 * Displays all configured social login options with appropriate icons.
 *
 * @package Next.js 16.x / React 19.x / Tailwind 4.x
 * @version 2.0.0
 */

'use client';

import { signIn } from 'next-auth/react';

/**
 * Supported social providers.
 */
type Provider = 'google' | 'facebook' | 'github' | 'twitter' | 'instagram';

interface SocialLoginButtonsProps {
  providers?: Provider[];
  callbackUrl?: string;
  className?: string;
}

/**
 * Social login buttons component.
 *
 * Displays a grid of social login provider buttons with appropriate
 * icons and colours for each provider.
 *
 * @example
 * <SocialLoginButtons
 *   providers={['google', 'github']}
 *   callbackUrl="/dashboard"
 * />
 */
export function SocialLoginButtons({
  providers = ['google', 'facebook', 'github', 'twitter', 'instagram'],
  callbackUrl = '/dashboard',
  className = '',
}: SocialLoginButtonsProps) {
  const handleSocialLogin = async (provider: Provider) => {
    await signIn(provider, { callbackUrl });
  };

  return (
    <div className={`space-y-3 ${className}`}>
      <p className="text-centre text-grey-500 text-sm">Or continue with</p>

      <div className="grid grid-cols-5 gap-3">
        {providers.map((provider) => (
          <button
            key={provider}
            type="button"
            onClick={() => handleSocialLogin(provider)}
            className="flex items-centre justify-centre px-4 py-2 border border-grey-300 rounded-lg hover:bg-grey-50 transition-colours"
            title={`Continue with ${provider.charAt(0).toUpperCase() + provider.slice(1)}`}
          >
            <ProviderIcon provider={provider} />
          </button>
        ))}
      </div>
    </div>
  );
}

/**
 * Provider icon component.
 */
function ProviderIcon({ provider }: { provider: Provider }) {
  switch (provider) {
    case 'google':
      return (
        <svg className="w-5 h-5" viewBox="0 0 24 24" fill="none">
          <path d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z" fill="#4285F4"/>
          <path d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z" fill="#34A853"/>
          <path d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z" fill="#FBBC05"/>
          <path d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z" fill="#EA4335"/>
        </svg>
      );
    case 'facebook':
      return (
        <svg className="w-5 h-5" fill="#1877F2" viewBox="0 0 24 24">
          <path d="M24 12.073c0-6.627-5.373-12-12-12s-12 5.373-12 12c0 5.99 4.388 10.954 10.125 11.854v-8.385H7.078v-3.47h3.047V9.43c0-3.007 1.792-4.669 4.533-4.669 1.312 0 2.686.235 2.686.235v2.953H15.83c-1.491 0-1.956.925-1.956 1.874v2.25h3.328l-.532 3.47h-2.796v8.385C19.612 23.027 24 18.062 24 12.073z"/>
        </svg>
      );
    case 'github':
      return (
        <svg className="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
          <path fillRule="evenodd" clipRule="evenodd" d="M12 2C6.477 2 2 6.484 2 12.017c0 4.425 2.865 8.18 6.839 9.504.5.092.682-.217.682-.483 0-.237-.008-.868-.013-1.703-2.782.605-3.369-1.343-3.369-1.343-.454-1.158-1.11-1.466-1.11-1.466-.908-.62.069-.608.069-.608 1.003.07 1.531 1.032 1.531 1.032.892 1.53 2.341 1.088 2.91.832.092-.647.35-1.088.636-1.338-2.22-.253-4.555-1.113-4.555-4.951 0-1.093.39-1.988 1.029-2.688-.103-.253-.446-1.272.098-2.65 0 0 .84-.27 2.75 1.026A9.564 9.564 0 0112 6.844c.85.004 1.705.115 2.504.337 1.909-1.296 2.747-1.027 2.747-1.027.546 1.379.202 2.398.1 2.651.64.7 1.028 1.595 1.028 2.688 0 3.848-2.339 4.695-4.566 4.943.359.309.678.92.678 1.855 0 1.338-.012 2.419-.012 2.747 0 .268.18.58.688.482A10.019 10.019 0 0022 12.017C22 6.484 17.522 2 12 2z"/>
        </svg>
      );
    case 'twitter':
      return (
        <svg className="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
          <path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/>
        </svg>
      );
    case 'instagram':
      return (
        <svg className="w-5 h-5" fill="#E4405F" viewBox="0 0 24 24">
          <path d="M12 2.163c3.204 0 3.584.012 4.85.07 3.252.148 4.771 1.691 4.919 4.919.058 1.265.069 1.645.069 4.849 0 3.205-.012 3.584-.069 4.849-.149 3.225-1.664 4.771-4.919 4.919-1.266.058-1.644.07-4.85.07-3.204 0-3.584-.012-4.849-.07-3.26-.149-4.771-1.699-4.919-4.92-.058-1.265-.07-1.644-.07-4.849 0-3.204.013-3.583.07-4.849.149-3.227 1.664-4.771 4.919-4.919 1.266-.057 1.645-.069 4.849-.069zm0-2.163c-3.259 0-3.667.014-4.947.072-4.358.2-6.78 2.618-6.98 6.98-.059 1.281-.073 1.689-.073 4.948 0 3.259.014 3.668.072 4.948.2 4.358 2.618 6.78 6.98 6.98 1.281.058 1.689.072 4.948.072 3.259 0 3.668-.014 4.948-.072 4.354-.2 6.782-2.618 6.979-6.98.059-1.28.073-1.689.073-4.948 0-3.259-.014-3.667-.072-4.947-.196-4.354-2.617-6.78-6.979-6.98-1.281-.059-1.69-.073-4.949-.073z"/>
        </svg>
      );
  }
}
```

---

## React Native Stack - React Native 0.83.x

### Social Auth Configuration - React Native

#### config/social-auth.config.ts

```typescript
/**
 * social-auth.config.ts
 *
 * Social authentication configuration for React Native using Expo AuthSession.
 * Supports Google, Facebook, GitHub, Twitter/X, and Instagram OAuth.
 *
 * @package React Native 0.83.x / Expo AuthSession / TypeScript 5.9
 * @version 2.0.0
 */

import Constants from 'expo-constants';

/**
 * OAuth provider configuration interface.
 */
export interface OAuthConfig {
  clientId: string;
  clientSecret?: string;
  discoveryUrl?: string;
  authorizationEndpoint?: string;
  tokenEndpoint?: string;
  scopes: string[];
  redirectUri: string;
}

/**
 * Get the Expo AuthSession redirect URI.
 */
const getRedirectUri = (): string => {
  return `${Constants.expoConfig?.scheme}://oauth`;
};

/**
 * OAuth provider configurations.
 */
export const socialAuthConfig: Record<string, OAuthConfig> = {
  google: {
    clientId: process.env.EXPO_PUBLIC_GOOGLE_CLIENT_ID!,
    discoveryUrl: 'https://accounts.google.com',
    scopes: ['openid', 'email', 'profile'],
    redirectUri: getRedirectUri(),
  },
  facebook: {
    clientId: process.env.EXPO_PUBLIC_FACEBOOK_CLIENT_ID!,
    authorizationEndpoint: 'https://www.facebook.com/v18.0/dialog/oauth',
    tokenEndpoint: 'https://graph.facebook.com/v18.0/oauth/access_token',
    scopes: ['email', 'public_profile'],
    redirectUri: getRedirectUri(),
  },
  github: {
    clientId: process.env.EXPO_PUBLIC_GITHUB_CLIENT_ID!,
    authorizationEndpoint: 'https://github.com/login/oauth/authorize',
    tokenEndpoint: 'https://github.com/login/oauth/access_token',
    scopes: ['read:user', 'user:email'],
    redirectUri: getRedirectUri(),
  },
  twitter: {
    clientId: process.env.EXPO_PUBLIC_TWITTER_CLIENT_ID!,
    authorizationEndpoint: 'https://twitter.com/i/oauth2/authorize',
    tokenEndpoint: 'https://api.twitter.com/2/oauth2/token',
    scopes: ['users.read', 'tweet.read', 'offline.access'],
    redirectUri: getRedirectUri(),
  },
  instagram: {
    clientId: process.env.EXPO_PUBLIC_INSTAGRAM_CLIENT_ID!,
    authorizationEndpoint: 'https://api.instagram.com/oauth/authorize',
    tokenEndpoint: 'https://api.instagram.com/oauth/access_token',
    scopes: ['user_profile', 'user_media'],
    redirectUri: getRedirectUri(),
  },
};
```

---

### Social Login Hooks - React Native

#### hooks/useSocialAuth.ts

```typescript
/**
 * useSocialAuth.ts
 *
 * React Native hook for social authentication using Expo AuthSession.
 * Handles OAuth flow for Google, Facebook, GitHub, Twitter/X, and Instagram.
 *
 * @package React Native 0.83.x / Expo AuthSession / TypeScript 5.9
 * @version 2.0.0
 */

import { useState, useCallback, useEffect } from 'react';
import * as AuthSession from 'expo-auth-session';
import * as Google from 'expo-auth-session/providers/google';
import * as Facebook from 'expo-auth-session/providers/facebook';
import * as WebBrowser from 'expo-web-browser';
import { useMutation } from '@apollo/client';
import { gql } from '@apollo/client';
import { useAuth } from './useAuth';

// Complete auth session for web browser
WebBrowser.maybeCompleteAuthSession();

/**
 * GraphQL mutation for social login.
 */
const SOCIAL_LOGIN_MUTATION = gql`
  mutation SocialLogin($provider: String!, $token: String!) {
    socialLogin(provider: $provider, token: $token) {
      success
      message
      user {
        id
        email
        name
      }
      tokens {
        accessToken
        refreshToken
        expiresAt
      }
    }
  }
`;

/**
 * Supported social providers.
 */
type SocialProvider = 'google' | 'facebook' | 'github' | 'twitter' | 'instagram';

/**
 * Social login result interface.
 */
interface SocialLoginResult {
  success: boolean;
  message?: string;
  user?: {
    id: string;
    email: string;
    name: string;
  };
}

/**
 * Social auth hook return interface.
 */
interface UseSocialAuthReturn {
  loginWithGoogle: () => Promise<SocialLoginResult>;
  loginWithFacebook: () => Promise<SocialLoginResult>;
  loginWithGitHub: () => Promise<SocialLoginResult>;
  loginWithTwitter: () => Promise<SocialLoginResult>;
  loginWithInstagram: () => Promise<SocialLoginResult>;
  loading: boolean;
  error: string | null;
}

/**
 * Custom hook for social authentication.
 *
 * Provides methods for authenticating with Google, Facebook, GitHub,
 * Twitter/X, and Instagram using OAuth2.
 *
 * @returns Object with social login methods and state
 *
 * @example
 * const { loginWithGoogle, loading, error } = useSocialAuth();
 *
 * const handleGoogleLogin = async () => {
 *   const result = await loginWithGoogle();
 *   if (result.success) {
 *     navigation.navigate('Dashboard');
 *   }
 * };
 */
export function useSocialAuth(): UseSocialAuthReturn {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const { setTokens } = useAuth();

  const [socialLoginMutation] = useMutation(SOCIAL_LOGIN_MUTATION);

  // Google auth configuration
  const [googleRequest, googleResponse, googlePromptAsync] = Google.useAuthRequest({
    clientId: process.env.EXPO_PUBLIC_GOOGLE_CLIENT_ID!,
    scopes: ['openid', 'email', 'profile'],
  });

  // Facebook auth configuration
  const [facebookRequest, facebookResponse, facebookPromptAsync] = Facebook.useAuthRequest({
    clientId: process.env.EXPO_PUBLIC_FACEBOOK_CLIENT_ID!,
  });

  /**
   * Handles the OAuth token and sends to backend.
   */
  const handleSocialLogin = useCallback(
    async (provider: SocialProvider, token: string): Promise<SocialLoginResult> => {
      try {
        const { data } = await socialLoginMutation({
          variables: { provider, token },
        });

        if (data?.socialLogin?.success) {
          await setTokens(data.socialLogin.tokens);
          return {
            success: true,
            user: data.socialLogin.user,
          };
        }

        return {
          success: false,
          message: data?.socialLogin?.message || 'Authentication failed',
        };
      } catch (err) {
        const message = err instanceof Error ? err.message : 'Authentication failed';
        setError(message);
        return { success: false, message };
      }
    },
    [socialLoginMutation, setTokens]
  );

  /**
   * Login with Google.
   */
  const loginWithGoogle = useCallback(async (): Promise<SocialLoginResult> => {
    setLoading(true);
    setError(null);

    try {
      const result = await googlePromptAsync();

      if (result.type === 'success' && result.authentication?.accessToken) {
        return handleSocialLogin('google', result.authentication.accessToken);
      }

      if (result.type === 'cancel') {
        return { success: false, message: 'Login cancelled' };
      }

      return { success: false, message: 'Google login failed' };
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Google login failed';
      setError(message);
      return { success: false, message };
    } finally {
      setLoading(false);
    }
  }, [googlePromptAsync, handleSocialLogin]);

  /**
   * Login with Facebook.
   */
  const loginWithFacebook = useCallback(async (): Promise<SocialLoginResult> => {
    setLoading(true);
    setError(null);

    try {
      const result = await facebookPromptAsync();

      if (result.type === 'success' && result.authentication?.accessToken) {
        return handleSocialLogin('facebook', result.authentication.accessToken);
      }

      if (result.type === 'cancel') {
        return { success: false, message: 'Login cancelled' };
      }

      return { success: false, message: 'Facebook login failed' };
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Facebook login failed';
      setError(message);
      return { success: false, message };
    } finally {
      setLoading(false);
    }
  }, [facebookPromptAsync, handleSocialLogin]);

  /**
   * Login with GitHub using custom OAuth flow.
   */
  const loginWithGitHub = useCallback(async (): Promise<SocialLoginResult> => {
    setLoading(true);
    setError(null);

    try {
      const discovery = {
        authorizationEndpoint: 'https://github.com/login/oauth/authorize',
        tokenEndpoint: 'https://github.com/login/oauth/access_token',
      };

      const request = new AuthSession.AuthRequest({
        clientId: process.env.EXPO_PUBLIC_GITHUB_CLIENT_ID!,
        scopes: ['read:user', 'user:email'],
        redirectUri: AuthSession.makeRedirectUri({ scheme: 'yourapp' }),
      });

      const result = await request.promptAsync(discovery);

      if (result.type === 'success' && result.params.code) {
        // Exchange code for token via your backend
        return handleSocialLogin('github', result.params.code);
      }

      if (result.type === 'cancel') {
        return { success: false, message: 'Login cancelled' };
      }

      return { success: false, message: 'GitHub login failed' };
    } catch (err) {
      const message = err instanceof Error ? err.message : 'GitHub login failed';
      setError(message);
      return { success: false, message };
    } finally {
      setLoading(false);
    }
  }, [handleSocialLogin]);

  /**
   * Login with Twitter/X using custom OAuth flow.
   */
  const loginWithTwitter = useCallback(async (): Promise<SocialLoginResult> => {
    setLoading(true);
    setError(null);

    try {
      const discovery = {
        authorizationEndpoint: 'https://twitter.com/i/oauth2/authorize',
        tokenEndpoint: 'https://api.twitter.com/2/oauth2/token',
      };

      const request = new AuthSession.AuthRequest({
        clientId: process.env.EXPO_PUBLIC_TWITTER_CLIENT_ID!,
        scopes: ['users.read', 'tweet.read', 'offline.access'],
        redirectUri: AuthSession.makeRedirectUri({ scheme: 'yourapp' }),
        usePKCE: true,
      });

      const result = await request.promptAsync(discovery);

      if (result.type === 'success' && result.params.code) {
        return handleSocialLogin('twitter', result.params.code);
      }

      if (result.type === 'cancel') {
        return { success: false, message: 'Login cancelled' };
      }

      return { success: false, message: 'Twitter login failed' };
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Twitter login failed';
      setError(message);
      return { success: false, message };
    } finally {
      setLoading(false);
    }
  }, [handleSocialLogin]);

  /**
   * Login with Instagram using custom OAuth flow.
   */
  const loginWithInstagram = useCallback(async (): Promise<SocialLoginResult> => {
    setLoading(true);
    setError(null);

    try {
      const discovery = {
        authorizationEndpoint: 'https://api.instagram.com/oauth/authorize',
        tokenEndpoint: 'https://api.instagram.com/oauth/access_token',
      };

      const request = new AuthSession.AuthRequest({
        clientId: process.env.EXPO_PUBLIC_INSTAGRAM_CLIENT_ID!,
        scopes: ['user_profile', 'user_media'],
        redirectUri: AuthSession.makeRedirectUri({ scheme: 'yourapp' }),
        responseType: AuthSession.ResponseType.Code,
      });

      const result = await request.promptAsync(discovery);

      if (result.type === 'success' && result.params.code) {
        return handleSocialLogin('instagram', result.params.code);
      }

      if (result.type === 'cancel') {
        return { success: false, message: 'Login cancelled' };
      }

      return { success: false, message: 'Instagram login failed' };
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Instagram login failed';
      setError(message);
      return { success: false, message };
    } finally {
      setLoading(false);
    }
  }, [handleSocialLogin]);

  return {
    loginWithGoogle,
    loginWithFacebook,
    loginWithGitHub,
    loginWithTwitter,
    loginWithInstagram,
    loading,
    error,
  };
}
```

---

### Social Login Screen - React Native

#### screens/SocialLoginScreen.tsx

```typescript
/**
 * SocialLoginScreen.tsx
 *
 * Social login screen with buttons for all supported providers.
 * Styled with NativeWind 4.x (Tailwind CSS for React Native).
 *
 * @package React Native 0.83.x / TypeScript 5.9 / NativeWind 4.x
 * @version 2.0.0
 */

import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  ActivityIndicator,
  Alert,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { useSocialAuth } from '@/hooks/useSocialAuth';
import {
  GoogleIcon,
  FacebookIcon,
  GitHubIcon,
  TwitterIcon,
  InstagramIcon,
} from '@/components/icons/SocialIcons';

/**
 * Social login screen component.
 *
 * Displays social login options for Google, Facebook, GitHub,
 * Twitter/X, and Instagram.
 */
export function SocialLoginScreen() {
  const navigation = useNavigation();
  const {
    loginWithGoogle,
    loginWithFacebook,
    loginWithGitHub,
    loginWithTwitter,
    loginWithInstagram,
    loading,
    error,
  } = useSocialAuth();

  /**
   * Handles social login result.
   */
  const handleLoginResult = (
    result: { success: boolean; message?: string },
    provider: string
  ) => {
    if (result.success) {
      navigation.navigate('Dashboard' as never);
    } else {
      Alert.alert(
        `${provider} Login Failed`,
        result.message || 'An error occurred'
      );
    }
  };

  return (
    <View className="flex-1 bg-white dark:bg-grey-900 px-6 py-12">
      <View className="flex-1 justify-centre">
        <Text className="text-2xl font-bold text-centre mb-2 text-grey-900 dark:text-white">
          Welcome Back
        </Text>
        <Text className="text-grey-600 dark:text-grey-400 text-centre mb-8">
          Sign in to continue
        </Text>

        {/* Social Login Buttons */}
        <View className="space-y-3">
          {/* Google */}
          <TouchableOpacity
            onPress={async () => {
              const result = await loginWithGoogle();
              handleLoginResult(result, 'Google');
            }}
            disabled={loading}
            className="flex-row items-centre justify-centre px-4 py-3 bg-white border border-grey-300 rounded-lg"
          >
            <GoogleIcon className="w-5 h-5 mr-3" />
            <Text className="text-grey-700 font-medium">
              Continue with Google
            </Text>
          </TouchableOpacity>

          {/* Facebook */}
          <TouchableOpacity
            onPress={async () => {
              const result = await loginWithFacebook();
              handleLoginResult(result, 'Facebook');
            }}
            disabled={loading}
            className="flex-row items-centre justify-centre px-4 py-3 bg-[#1877F2] rounded-lg"
          >
            <FacebookIcon className="w-5 h-5 mr-3" fill="white" />
            <Text className="text-white font-medium">
              Continue with Facebook
            </Text>
          </TouchableOpacity>

          {/* GitHub */}
          <TouchableOpacity
            onPress={async () => {
              const result = await loginWithGitHub();
              handleLoginResult(result, 'GitHub');
            }}
            disabled={loading}
            className="flex-row items-centre justify-centre px-4 py-3 bg-grey-900 rounded-lg"
          >
            <GitHubIcon className="w-5 h-5 mr-3" fill="white" />
            <Text className="text-white font-medium">
              Continue with GitHub
            </Text>
          </TouchableOpacity>

          {/* Twitter/X */}
          <TouchableOpacity
            onPress={async () => {
              const result = await loginWithTwitter();
              handleLoginResult(result, 'Twitter');
            }}
            disabled={loading}
            className="flex-row items-centre justify-centre px-4 py-3 bg-black rounded-lg"
          >
            <TwitterIcon className="w-5 h-5 mr-3" fill="white" />
            <Text className="text-white font-medium">
              Continue with X
            </Text>
          </TouchableOpacity>

          {/* Instagram */}
          <TouchableOpacity
            onPress={async () => {
              const result = await loginWithInstagram();
              handleLoginResult(result, 'Instagram');
            }}
            disabled={loading}
            className="flex-row items-centre justify-centre px-4 py-3 bg-gradient-to-r from-[#FEDA75] via-[#D62976] to-[#4F5BD5] rounded-lg"
          >
            <InstagramIcon className="w-5 h-5 mr-3" fill="white" />
            <Text className="text-white font-medium">
              Continue with Instagram
            </Text>
          </TouchableOpacity>
        </View>

        {/* Loading Indicator */}
        {loading && (
          <View className="items-centre mt-6">
            <ActivityIndicator size="large" color="#3B82F6" />
            <Text className="text-grey-600 mt-2">Signing in...</Text>
          </View>
        )}

        {/* Error Message */}
        {error && (
          <View className="mt-6 p-4 bg-red-50 rounded-lg">
            <Text className="text-red-600 text-centre">{error}</Text>
          </View>
        )}

        {/* Divider */}
        <View className="flex-row items-centre my-6">
          <View className="flex-1 h-px bg-grey-300" />
          <Text className="mx-4 text-grey-500">or</Text>
          <View className="flex-1 h-px bg-grey-300" />
        </View>

        {/* Email Login Link */}
        <TouchableOpacity
          onPress={() => navigation.navigate('EmailLogin' as never)}
          className="py-3"
        >
          <Text className="text-blue-600 text-centre font-medium">
            Sign in with email
          </Text>
        </TouchableOpacity>
      </View>

      {/* Footer */}
      <View className="mt-6">
        <Text className="text-grey-500 text-centre text-sm">
          Don't have an account?{' '}
          <Text
            className="text-blue-600 font-medium"
            onPress={() => navigation.navigate('Register' as never)}
          >
            Sign up
          </Text>
        </Text>
      </View>
    </View>
  );
}
```
- [Overview](#)
- [Metadata](#)
- [Supported Providers](#)
- [TALL Stack - Laravel 12.x](#)
- [Django/Wagtail Stack - Django 6.x](#)
- [React/Next.js Stack - Next.js 16.x](#)
- [React Native Stack - React Native 0.83.x](#)
