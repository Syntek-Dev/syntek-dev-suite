# Password Validation

## Overview

Strong password validation implementations for secure user authentication. All implementations include:
- Minimum length requirements (12+ characters)
- Complexity requirements (uppercase, lowercase, numbers, special characters)
- Breached password detection via HaveIBeenPwned API

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
| **Laravel** | 12.x |
| **PHP** | 8.4 |
| **Django** | 6.x |
| **Python** | 3.14 |
| **Node.js** | 24.x |
| **TypeScript** | 5.9 |
| **Next.js** | 16.x |
| **React** | 19.x |
| **React Native** | 0.83.x |
| **Tailwind CSS** | 4.x |
| **NativeWind** | v4 |
| **Stacks** | TALL, Django/Wagtail, React/Next.js, React Native |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Password Requirements](#password-requirements)
- [Laravel (TALL Stack)](#laravel-tall-stack)
- [Django/Wagtail](#djangowagtail)
- [React/Next.js](#reactnextjs)
- [React Native](#react-native)


## Password Requirements

```
Minimum Requirements:
- Length: 12+ characters (NIST recommends up to 64)
- At least 1 uppercase letter (A-Z)
- At least 1 lowercase letter (a-z)
- At least 1 number (0-9)
- At least 1 special character (!@#$%^&*()_+-=[]{}|;:,.<>?)

Enhanced Checks:
- Not in common password lists (top 100k breached passwords)
- Not containing username or email
- Not a simple keyboard pattern (qwerty, 123456)
- Not a repeated character sequence (aaaa, 1111)
```

---

## Laravel (TALL Stack)

### Validation Rule

#### app/Rules/StrongPassword.php

```php
<?php

/**
 * StrongPassword.php
 *
 * Custom validation rule that enforces strong password requirements including
 * minimum length, character complexity, and breach detection via the
 * HaveIBeenPwned API.
 */

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;
use Illuminate\Support\Facades\Http;

class StrongPassword implements Rule
{
    protected string $message = '';

    /**
     * Validates that the password meets all strength requirements.
     *
     * Checks for minimum length, character complexity (uppercase, lowercase,
     * numbers, special characters), and verifies the password has not appeared
     * in known data breaches.
     *
     * @param string $attribute The name of the attribute being validated
     * @param mixed $value The password value to validate
     * @return bool True if password meets all requirements
     */
    public function passes($attribute, $value): bool
    {
        // Length check
        if (strlen($value) < 12) {
            $this->message = 'Password must be at least 12 characters.';
            return false;
        }

        // Complexity checks
        if (!preg_match('/[A-Z]/', $value)) {
            $this->message = 'Password must contain at least one uppercase letter.';
            return false;
        }

        if (!preg_match('/[a-z]/', $value)) {
            $this->message = 'Password must contain at least one lowercase letter.';
            return false;
        }

        if (!preg_match('/[0-9]/', $value)) {
            $this->message = 'Password must contain at least one number.';
            return false;
        }

        if (!preg_match('/[!@#$%^&*()_+\-=\[\]{}|;:,.<>?]/', $value)) {
            $this->message = 'Password must contain at least one special character.';
            return false;
        }

        // Check against breached passwords (HaveIBeenPwned API)
        if ($this->isBreachedPassword($value)) {
            $this->message = 'This password has been found in a data breach. Please choose a different password.';
            return false;
        }

        return true;
    }

    /**
     * Checks if the password appears in the HaveIBeenPwned database.
     *
     * Uses the k-Anonymity model to check passwords without sending the
     * full password hash to the API. Only the first 5 characters of the
     * SHA1 hash are sent.
     *
     * @param string $password The password to check
     * @return bool True if password has been breached
     */
    protected function isBreachedPassword(string $password): bool
    {
        $hash = strtoupper(sha1($password));
        $prefix = substr($hash, 0, 5);
        $suffix = substr($hash, 5);

        $response = Http::get("https://api.pwnedpasswords.com/range/{$prefix}");

        if ($response->failed()) {
            return false; // Fail open - don't block registration if API is down
        }

        return str_contains($response->body(), $suffix);
    }

    /**
     * Returns the validation error message.
     *
     * @return string The error message
     */
    public function message(): string
    {
        return $this->message;
    }
}
```

### Livewire Component

#### app/Livewire/Auth/RegisterForm.php

```php
<?php

/**
 * RegisterForm.php
 *
 * Livewire component for user registration with real-time password validation.
 * Uses Alpine.js for client-side interactivity and displays password strength
 * indicators as the user types.
 */

namespace App\Livewire\Auth;

use App\Models\User;
use App\Rules\StrongPassword;
use Illuminate\Support\Facades\Hash;
use Livewire\Component;

class RegisterForm extends Component
{
    public string $name = '';
    public string $email = '';
    public string $password = '';
    public string $password_confirmation = '';

    public array $passwordStrength = [
        'hasLength' => false,
        'hasUppercase' => false,
        'hasLowercase' => false,
        'hasNumber' => false,
        'hasSpecial' => false,
    ];

    /**
     * Real-time validation rules for form fields.
     *
     * Uses Laravel 12's validation syntax with custom StrongPassword rule.
     *
     * @return array<string, array<int, mixed>>
     */
    protected function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'confirmed', new StrongPassword()],
        ];
    }

    /**
     * Updates password strength indicators as user types.
     *
     * Checks each requirement in real-time to provide visual feedback
     * without making API calls until form submission.
     *
     * @return void
     */
    public function updatedPassword(): void
    {
        $this->passwordStrength = [
            'hasLength' => strlen($this->password) >= 12,
            'hasUppercase' => preg_match('/[A-Z]/', $this->password),
            'hasLowercase' => preg_match('/[a-z]/', $this->password),
            'hasNumber' => preg_match('/[0-9]/', $this->password),
            'hasSpecial' => preg_match('/[!@#$%^&*()_+\-=\[\]{}|;:,.<>?]/', $this->password),
        ];
    }

    /**
     * Handles user registration form submission.
     *
     * Validates all fields including password strength, then creates
     * the user account with a hashed password.
     *
     * @return void
     */
    public function register(): void
    {
        $validated = $this->validate();

        $user = User::create([
            'name' => $validated['name'],
            'email' => $validated['email'],
            'password' => Hash::make($validated['password']),
        ]);

        auth()->login($user);

        session()->flash('success', 'Account created successfully!');

        $this->redirect(route('dashboard'));
    }

    /**
     * Renders the registration form component.
     *
     * @return \Illuminate\View\View
     */
    public function render()
    {
        return view('livewire.auth.register-form');
    }
}
```

#### resources/views/livewire/auth/register-form.blade.php

```blade
{{--
  register-form.blade.php

  Registration form view with Tailwind 4.x CSS-first configuration styling
  and Alpine.js for password visibility toggle.
--}}

<div class="w-full max-w-md mx-auto p-6">
    <h2 class="text-2xl font-bold mb-6">Create Account</h2>

    <form wire:submit="register" class="space-y-4">
        {{-- Name Field --}}
        <div>
            <label for="name" class="block text-sm font-medium mb-1">
                Name
            </label>
            <input
                type="text"
                id="name"
                wire:model.blur="name"
                class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                required
            />
            @error('name')
                <p class="text-red-600 text-sm mt-1">{{ $message }}</p>
            @enderror
        </div>

        {{-- Email Field --}}
        <div>
            <label for="email" class="block text-sm font-medium mb-1">
                Email
            </label>
            <input
                type="email"
                id="email"
                wire:model.blur="email"
                class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                required
            />
            @error('email')
                <p class="text-red-600 text-sm mt-1">{{ $message }}</p>
            @enderror
        </div>

        {{-- Password Field with Strength Indicator --}}
        <div x-data="{ showPassword: false }">
            <label for="password" class="block text-sm font-medium mb-1">
                Password
            </label>
            <div class="relative">
                <input
                    :type="showPassword ? 'text' : 'password'"
                    id="password"
                    wire:model.live="password"
                    class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 pr-10"
                    required
                />
                <button
                    type="button"
                    @click="showPassword = !showPassword"
                    class="absolute right-2 top-2 text-grey-500 hover:text-grey-700"
                >
                    <span x-show="!showPassword">Show</span>
                    <span x-show="showPassword">Hide</span>
                </button>
            </div>

            {{-- Password Requirements Checklist --}}
            <div class="mt-2 space-y-1 text-sm">
                <div class="flex items-centre gap-2">
                    <span class="{{ $passwordStrength['hasLength'] ? 'text-green-600' : 'text-grey-500' }}">
                        {{ $passwordStrength['hasLength'] ? '✓' : '○' }}
                    </span>
                    <span>At least 12 characters</span>
                </div>
                <div class="flex items-centre gap-2">
                    <span class="{{ $passwordStrength['hasUppercase'] ? 'text-green-600' : 'text-grey-500' }}">
                        {{ $passwordStrength['hasUppercase'] ? '✓' : '○' }}
                    </span>
                    <span>One uppercase letter</span>
                </div>
                <div class="flex items-centre gap-2">
                    <span class="{{ $passwordStrength['hasLowercase'] ? 'text-green-600' : 'text-grey-500' }}">
                        {{ $passwordStrength['hasLowercase'] ? '✓' : '○' }}
                    </span>
                    <span>One lowercase letter</span>
                </div>
                <div class="flex items-centre gap-2">
                    <span class="{{ $passwordStrength['hasNumber'] ? 'text-green-600' : 'text-grey-500' }}">
                        {{ $passwordStrength['hasNumber'] ? '✓' : '○' }}
                    </span>
                    <span>One number</span>
                </div>
                <div class="flex items-centre gap-2">
                    <span class="{{ $passwordStrength['hasSpecial'] ? 'text-green-600' : 'text-grey-500' }}">
                        {{ $passwordStrength['hasSpecial'] ? '✓' : '○' }}
                    </span>
                    <span>One special character</span>
                </div>
            </div>

            @error('password')
                <p class="text-red-600 text-sm mt-1">{{ $message }}</p>
            @enderror
        </div>

        {{-- Password Confirmation Field --}}
        <div>
            <label for="password_confirmation" class="block text-sm font-medium mb-1">
                Confirm Password
            </label>
            <input
                type="password"
                id="password_confirmation"
                wire:model.blur="password_confirmation"
                class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                required
            />
        </div>

        {{-- Submit Button --}}
        <button
            type="submit"
            class="w-full py-2 px-4 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50"
            wire:loading.attr="disabled"
        >
            <span wire:loading.remove>Create Account</span>
            <span wire:loading>Creating...</span>
        </button>
    </form>
</div>
```

---

## Django/Wagtail

### Password Validator

#### validators.py

```python
"""
Password validation module for Django applications.

Provides strong password validation including complexity requirements
and breach detection via the HaveIBeenPwned API.
"""

import re
import hashlib
import requests
from django.core.exceptions import ValidationError


class StrongPasswordValidator:
    """
    Validates passwords meet security requirements.

    Enforces minimum length, character complexity, and checks passwords
    against the HaveIBeenPwned database of breached credentials.
    """

    def __init__(self, min_length: int = 12):
        """
        Initialises the validator with configurable minimum length.

        Args:
            min_length: Minimum required password length (default: 12)
        """
        self.min_length = min_length

    def validate(self, password: str, user=None) -> None:
        """
        Validates the password meets all security requirements.

        Raises ValidationError if any requirement is not met.

        Args:
            password: The password string to validate
            user: Optional user object for checking password contains email

        Raises:
            ValidationError: When password fails any validation check
        """
        errors = []

        if len(password) < self.min_length:
            errors.append(f'Password must be at least {self.min_length} characters.')

        if not re.search(r'[A-Z]', password):
            errors.append('Password must contain at least one uppercase letter.')

        if not re.search(r'[a-z]', password):
            errors.append('Password must contain at least one lowercase letter.')

        if not re.search(r'[0-9]', password):
            errors.append('Password must contain at least one number.')

        if not re.search(r'[!@#$%^&*()_+\-=\[\]{}|;:,.<>?]', password):
            errors.append('Password must contain at least one special character.')

        if user and user.email and user.email.split('@')[0].lower() in password.lower():
            errors.append('Password cannot contain your email address.')

        if self._is_breached_password(password):
            errors.append('This password has been found in a data breach.')

        if errors:
            raise ValidationError(errors)

    def _is_breached_password(self, password: str) -> bool:
        """
        Checks if password appears in HaveIBeenPwned database.

        Uses k-Anonymity model - only sends first 5 characters of SHA1 hash.

        Args:
            password: The password to check

        Returns:
            True if password has been breached, False otherwise
        """
        sha1_hash = hashlib.sha1(password.encode()).hexdigest().upper()
        prefix, suffix = sha1_hash[:5], sha1_hash[5:]

        try:
            response = requests.get(
                f'https://api.pwnedpasswords.com/range/{prefix}',
                timeout=2
            )
            return suffix in response.text
        except requests.RequestException:
            return False  # Fail open

    def get_help_text(self) -> str:
        """
        Returns help text describing password requirements.

        Returns:
            Help text string for display to users
        """
        return (
            'Your password must be at least 12 characters and include '
            'uppercase, lowercase, numbers, and special characters.'
        )
```

### User Registration Form

#### forms.py

```python
"""
User registration forms for Django 6 applications.

Provides form handling for user registration with integrated password
validation and strength checking.
"""

from django import forms
from django.contrib.auth import get_user_model
from django.contrib.auth.forms import UserCreationForm
from .validators import StrongPasswordValidator

User = get_user_model()


class UserRegistrationForm(UserCreationForm):
    """
    Extended user registration form with strong password validation.

    Includes email field and uses the StrongPasswordValidator to enforce
    password security requirements. Compatible with Django 6.x.
    """

    email = forms.EmailField(
        required=True,
        widget=forms.EmailInput(attrs={
            'class': 'w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500',
            'placeholder': 'user@example.com',
        }),
        help_text='Required. Enter a valid email address.',
    )

    password1 = forms.CharField(
        label='Password',
        widget=forms.PasswordInput(attrs={
            'class': 'w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500',
            'autocomplete': 'new-password',
        }),
        help_text=(
            'Must be at least 12 characters with uppercase, lowercase, '
            'numbers, and special characters.'
        ),
    )

    password2 = forms.CharField(
        label='Confirm Password',
        widget=forms.PasswordInput(attrs={
            'class': 'w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500',
            'autocomplete': 'new-password',
        }),
    )

    class Meta:
        model = User
        fields = ('username', 'email', 'password1', 'password2')
        widgets = {
            'username': forms.TextInput(attrs={
                'class': 'w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500',
            }),
        }

    def __init__(self, *args, **kwargs):
        """
        Initialises the form and adds password validators.

        Args:
            *args: Variable length argument list
            **kwargs: Arbitrary keyword arguments
        """
        super().__init__(*args, **kwargs)
        self.fields['password1'].validators.append(StrongPasswordValidator())

    def clean_email(self):
        """
        Validates email is unique in the database.

        Returns:
            The cleaned email value

        Raises:
            ValidationError: If email is already registered
        """
        email = self.cleaned_data.get('email')
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError('This email address is already registered.')
        return email

    def save(self, commit=True):
        """
        Saves the user with the provided email and hashed password.

        Args:
            commit: Whether to save to database immediately

        Returns:
            The created user instance
        """
        user = super().save(commit=False)
        user.email = self.cleaned_data['email']
        if commit:
            user.save()
        return user
```

#### views.py

```python
"""
Authentication views for Django 6 applications.

Handles user registration with password validation and provides
real-time feedback using HTMX.
"""

from django.shortcuts import render, redirect
from django.contrib.auth import login
from django.views import View
from .forms import UserRegistrationForm


class RegisterView(View):
    """
    Handles user registration with strong password validation.

    Provides both standard form rendering and HTMX-enhanced
    password strength checking.
    """

    template_name = 'auth/register.html'
    form_class = UserRegistrationForm

    def get(self, request):
        """
        Renders the registration form.

        Args:
            request: The HTTP request object

        Returns:
            Rendered registration template
        """
        form = self.form_class()
        return render(request, self.template_name, {'form': form})

    def post(self, request):
        """
        Processes registration form submission.

        Args:
            request: The HTTP request object with form data

        Returns:
            Redirect to dashboard on success, or re-rendered form with errors
        """
        form = self.form_class(request.POST)

        if form.is_valid():
            user = form.save()
            login(request, user)
            return redirect('dashboard')

        return render(request, self.template_name, {'form': form})
```

#### templates/auth/register.html

```django
{# register.html

User registration template with Tailwind 4.x CSS-first configuration.
Uses Alpine.js for password visibility toggle and HTMX for real-time validation.
#}

{% load static %}
<!DOCTYPE html>
<html lang="en-GB">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Register - Your App</title>

    {# Tailwind 4.x CSS-first configuration #}
    <link rel="stylesheet" href="{% static 'css/tailwind.css' %}">

    {# Alpine.js for interactivity #}
    <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
</head>
<body class="bg-grey-50">
    <div class="min-h-screen flex items-centre justify-centre py-12 px-4 sm:px-6 lg:px-8">
        <div class="max-w-md w-full bg-white p-8 rounded-lg shadow">
            <h2 class="text-2xl font-bold mb-6">Create Account</h2>

            <form method="post" class="space-y-4" x-data="{ showPassword: false }">
                {% csrf_token %}

                {# Username Field #}
                <div>
                    <label for="{{ form.username.id_for_label }}" class="block text-sm font-medium mb-1">
                        Username
                    </label>
                    {{ form.username }}
                    {% if form.username.errors %}
                        <p class="text-red-600 text-sm mt-1">{{ form.username.errors.0 }}</p>
                    {% endif %}
                </div>

                {# Email Field #}
                <div>
                    <label for="{{ form.email.id_for_label }}" class="block text-sm font-medium mb-1">
                        Email
                    </label>
                    {{ form.email }}
                    {% if form.email.errors %}
                        <p class="text-red-600 text-sm mt-1">{{ form.email.errors.0 }}</p>
                    {% endif %}
                </div>

                {# Password Field with Strength Indicator #}
                <div>
                    <label for="{{ form.password1.id_for_label }}" class="block text-sm font-medium mb-1">
                        Password
                    </label>
                    <div class="relative">
                        <input
                            :type="showPassword ? 'text' : 'password'"
                            name="password1"
                            id="{{ form.password1.id_for_label }}"
                            class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 pr-10"
                            required
                        />
                        <button
                            type="button"
                            @click="showPassword = !showPassword"
                            class="absolute right-2 top-2 text-grey-500 hover:text-grey-700"
                        >
                            <span x-show="!showPassword">Show</span>
                            <span x-show="showPassword">Hide</span>
                        </button>
                    </div>

                    {# Password Requirements Checklist #}
                    <div class="mt-2 text-sm text-grey-600">
                        <p class="font-medium">Password must contain:</p>
                        <ul class="list-disc list-inside space-y-1 mt-1">
                            <li>At least 12 characters</li>
                            <li>One uppercase letter (A-Z)</li>
                            <li>One lowercase letter (a-z)</li>
                            <li>One number (0-9)</li>
                            <li>One special character (!@#$%...)</li>
                        </ul>
                    </div>

                    {% if form.password1.errors %}
                        <p class="text-red-600 text-sm mt-1">{{ form.password1.errors.0 }}</p>
                    {% endif %}
                </div>

                {# Password Confirmation Field #}
                <div>
                    <label for="{{ form.password2.id_for_label }}" class="block text-sm font-medium mb-1">
                        Confirm Password
                    </label>
                    {{ form.password2 }}
                    {% if form.password2.errors %}
                        <p class="text-red-600 text-sm mt-1">{{ form.password2.errors.0 }}</p>
                    {% endif %}
                </div>

                {# Submit Button #}
                <button
                    type="submit"
                    class="w-full py-2 px-4 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50"
                >
                    Create Account
                </button>
            </form>

            <p class="mt-4 text-centre text-sm text-grey-600">
                Already have an account?
                <a href="{% url 'login' %}" class="text-blue-600 hover:text-blue-700">Sign in</a>
            </p>
        </div>
    </div>
</body>
</html>
```

---

## React/Next.js

### Validation Hook

#### hooks/usePasswordValidation.ts

```typescript
/**
 * usePasswordValidation.ts
 *
 * Custom React hook for password validation in Next.js 16 / React 19 applications.
 * Provides real-time password strength checking and breach detection.
 *
 * Compatible with Next.js 16.x, React 19.x, TypeScript 5.9
 */

'use client';

import { useState, useEffect, useCallback } from 'react';

/**
 * Password strength requirements interface.
 */
interface PasswordStrength {
  hasLength: boolean;
  hasUppercase: boolean;
  hasLowercase: boolean;
  hasNumber: boolean;
  hasSpecial: boolean;
}

/**
 * Validation result interface.
 */
interface ValidationResult {
  isValid: boolean;
  errors: string[];
  strength: PasswordStrength;
  isBreached: boolean | null;
}

/**
 * Custom hook for password validation with real-time feedback.
 *
 * Validates password strength requirements and optionally checks against
 * the HaveIBeenPwned API for known breached passwords.
 *
 * @param password - The password to validate
 * @param checkBreached - Whether to check against breach database (default: true)
 * @returns Validation result with strength indicators and errors
 *
 * @example
 * const { isValid, errors, strength } = usePasswordValidation(password);
 */
export function usePasswordValidation(
  password: string,
  checkBreached: boolean = true
): ValidationResult {
  const [isBreached, setIsBreached] = useState<boolean | null>(null);
  const [strength, setStrength] = useState<PasswordStrength>({
    hasLength: false,
    hasUppercase: false,
    hasLowercase: false,
    hasNumber: false,
    hasSpecial: false,
  });

  /**
   * Checks if password has been found in data breaches.
   */
  const checkBreachedPassword = useCallback(async (pwd: string) => {
    if (!pwd || pwd.length < 12) return;

    try {
      const encoder = new TextEncoder();
      const data = encoder.encode(pwd);
      const hashBuffer = await crypto.subtle.digest('SHA-1', data);
      const hashArray = Array.from(new Uint8Array(hashBuffer));
      const hashHex = hashArray
        .map((b) => b.toString(16).padStart(2, '0'))
        .join('')
        .toUpperCase();

      const prefix = hashHex.slice(0, 5);
      const suffix = hashHex.slice(5);

      const response = await fetch(
        `https://api.pwnedpasswords.com/range/${prefix}`,
        { signal: AbortSignal.timeout(2000) }
      );

      if (!response.ok) {
        setIsBreached(null);
        return;
      }

      const text = await response.text();
      setIsBreached(text.includes(suffix));
    } catch {
      setIsBreached(null); // Fail open
    }
  }, []);

  /**
   * Updates password strength indicators in real-time.
   */
  useEffect(() => {
    const newStrength: PasswordStrength = {
      hasLength: password.length >= 12,
      hasUppercase: /[A-Z]/.test(password),
      hasLowercase: /[a-z]/.test(password),
      hasNumber: /[0-9]/.test(password),
      hasSpecial: /[!@#$%^&*()_+\-=\[\]{}|;:,.<>?]/.test(password),
    };

    setStrength(newStrength);

    // Debounce breach check
    const timer = setTimeout(() => {
      if (checkBreached && password.length >= 12) {
        checkBreachedPassword(password);
      } else {
        setIsBreached(null);
      }
    }, 500);

    return () => clearTimeout(timer);
  }, [password, checkBreached, checkBreachedPassword]);

  /**
   * Generates validation errors based on requirements.
   */
  const errors: string[] = [];
  if (!strength.hasLength) errors.push('Password must be at least 12 characters.');
  if (!strength.hasUppercase) errors.push('Password must contain an uppercase letter.');
  if (!strength.hasLowercase) errors.push('Password must contain a lowercase letter.');
  if (!strength.hasNumber) errors.push('Password must contain a number.');
  if (!strength.hasSpecial) errors.push('Password must contain a special character.');
  if (isBreached === true) errors.push('This password has been found in a data breach.');

  const isValid =
    Object.values(strength).every(Boolean) && isBreached !== true;

  return {
    isValid,
    errors,
    strength,
    isBreached,
  };
}
```

### Password Input Component

#### components/PasswordInput.tsx

```typescript
/**
 * PasswordInput.tsx
 *
 * Reusable password input component with strength indicator and real-time validation.
 * Built for Next.js 16 / React 19 with Tailwind 4.x CSS-first configuration.
 *
 * Compatible with Next.js 16.x, React 19.x, TypeScript 5.9, Tailwind CSS 4.x
 */

'use client';

import { useState } from 'react';
import { usePasswordValidation } from '@/hooks/usePasswordValidation';

/**
 * Props for the PasswordInput component.
 */
interface PasswordInputProps {
  /** Current password value */
  value: string;
  /** Callback when password changes */
  onChange: (value: string) => void;
  /** Input name attribute */
  name?: string;
  /** Input ID attribute */
  id?: string;
  /** Whether to show strength indicator */
  showStrength?: boolean;
  /** Whether to check against breach database */
  checkBreached?: boolean;
  /** Additional CSS classes */
  className?: string;
}

/**
 * Password input field with real-time validation and strength indicator.
 *
 * Displays password requirements as a checklist and optionally checks
 * against the HaveIBeenPwned database.
 *
 * @example
 * <PasswordInput
 *   value={password}
 *   onChange={setPassword}
 *   showStrength={true}
 *   checkBreached={true}
 * />
 */
export function PasswordInput({
  value,
  onChange,
  name = 'password',
  id = 'password',
  showStrength = true,
  checkBreached = true,
  className = '',
}: PasswordInputProps) {
  const [showPassword, setShowPassword] = useState(false);
  const { strength, isBreached, errors } = usePasswordValidation(
    value,
    checkBreached
  );

  return (
    <div className={className}>
      <label htmlFor={id} className="block text-sm font-medium mb-1">
        Password
      </label>

      {/* Password Input with Toggle */}
      <div className="relative">
        <input
          type={showPassword ? 'text' : 'password'}
          id={id}
          name={name}
          value={value}
          onChange={(e) => onChange(e.target.value)}
          className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 pr-20"
          autoComplete="new-password"
          required
        />
        <button
          type="button"
          onClick={() => setShowPassword(!showPassword)}
          className="absolute right-2 top-2 px-2 py-1 text-sm text-grey-600 hover:text-grey-800"
        >
          {showPassword ? 'Hide' : 'Show'}
        </button>
      </div>

      {/* Strength Indicator */}
      {showStrength && value && (
        <div className="mt-2 space-y-1">
          <div className="text-sm font-medium mb-1">Password Requirements:</div>

          <div className="space-y-1 text-sm">
            <RequirementItem met={strength.hasLength}>
              At least 12 characters
            </RequirementItem>
            <RequirementItem met={strength.hasUppercase}>
              One uppercase letter (A-Z)
            </RequirementItem>
            <RequirementItem met={strength.hasLowercase}>
              One lowercase letter (a-z)
            </RequirementItem>
            <RequirementItem met={strength.hasNumber}>
              One number (0-9)
            </RequirementItem>
            <RequirementItem met={strength.hasSpecial}>
              One special character (!@#$%...)
            </RequirementItem>
            {checkBreached && isBreached !== null && (
              <RequirementItem met={!isBreached}>
                {isBreached
                  ? 'Found in data breach - choose different password'
                  : 'Not found in known data breaches'}
              </RequirementItem>
            )}
          </div>
        </div>
      )}

      {/* Error Messages */}
      {errors.length > 0 && (
        <div className="mt-2 space-y-1">
          {errors.map((error, index) => (
            <p key={index} className="text-sm text-red-600">
              {error}
            </p>
          ))}
        </div>
      )}
    </div>
  );
}

/**
 * Individual requirement indicator component.
 */
function RequirementItem({
  met,
  children,
}: {
  met: boolean;
  children: React.ReactNode;
}) {
  return (
    <div className="flex items-centre gap-2">
      <span
        className={met ? 'text-green-600' : 'text-grey-400'}
        aria-label={met ? 'Requirement met' : 'Requirement not met'}
      >
        {met ? '✓' : '○'}
      </span>
      <span className={met ? 'text-green-600' : 'text-grey-600'}>
        {children}
      </span>
    </div>
  );
}
```

#### app/register/page.tsx

```typescript
/**
 * page.tsx
 *
 * Registration page for Next.js 16 application.
 * Uses server actions for form submission with client-side validation.
 */

'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { PasswordInput } from '@/components/PasswordInput';

/**
 * User registration page component.
 *
 * Provides a registration form with real-time password validation
 * and server-side processing.
 */
export default function RegisterPage() {
  const router = useRouter();
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    password: '',
    confirmPassword: '',
  });
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  /**
   * Handles form submission with validation.
   */
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setErrors({});
    setIsSubmitting(true);

    try {
      // Validate password confirmation
      if (formData.password !== formData.confirmPassword) {
        setErrors({ confirmPassword: 'Passwords do not match.' });
        setIsSubmitting(false);
        return;
      }

      // Submit to API
      const response = await fetch('/api/auth/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          name: formData.name,
          email: formData.email,
          password: formData.password,
        }),
      });

      const data = await response.json();

      if (!response.ok) {
        setErrors(data.errors || { general: 'Registration failed.' });
        setIsSubmitting(false);
        return;
      }

      // Redirect to dashboard on success
      router.push('/dashboard');
    } catch (error) {
      setErrors({ general: 'An error occurred. Please try again.' });
      setIsSubmitting(false);
    }
  };

  return (
    <div className="min-h-screen flex items-centre justify-centre bg-grey-50 py-12 px-4">
      <div className="max-w-md w-full bg-white p-8 rounded-lg shadow">
        <h1 className="text-2xl font-bold mb-6">Create Account</h1>

        <form onSubmit={handleSubmit} className="space-y-4">
          {/* Name Field */}
          <div>
            <label htmlFor="name" className="block text-sm font-medium mb-1">
              Name
            </label>
            <input
              type="text"
              id="name"
              value={formData.name}
              onChange={(e) =>
                setFormData({ ...formData, name: e.target.value })
              }
              className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              required
            />
            {errors.name && (
              <p className="text-sm text-red-600 mt-1">{errors.name}</p>
            )}
          </div>

          {/* Email Field */}
          <div>
            <label htmlFor="email" className="block text-sm font-medium mb-1">
              Email
            </label>
            <input
              type="email"
              id="email"
              value={formData.email}
              onChange={(e) =>
                setFormData({ ...formData, email: e.target.value })
              }
              className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              required
            />
            {errors.email && (
              <p className="text-sm text-red-600 mt-1">{errors.email}</p>
            )}
          </div>

          {/* Password Field */}
          <PasswordInput
            value={formData.password}
            onChange={(value) =>
              setFormData({ ...formData, password: value })
            }
            showStrength={true}
            checkBreached={true}
          />

          {/* Confirm Password Field */}
          <div>
            <label
              htmlFor="confirmPassword"
              className="block text-sm font-medium mb-1"
            >
              Confirm Password
            </label>
            <input
              type="password"
              id="confirmPassword"
              value={formData.confirmPassword}
              onChange={(e) =>
                setFormData({ ...formData, confirmPassword: e.target.value })
              }
              className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              required
            />
            {errors.confirmPassword && (
              <p className="text-sm text-red-600 mt-1">
                {errors.confirmPassword}
              </p>
            )}
          </div>

          {/* General Errors */}
          {errors.general && (
            <p className="text-sm text-red-600">{errors.general}</p>
          )}

          {/* Submit Button */}
          <button
            type="submit"
            disabled={isSubmitting}
            className="w-full py-2 px-4 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            {isSubmitting ? 'Creating Account...' : 'Create Account'}
          </button>
        </form>

        <p className="mt-4 text-centre text-sm text-grey-600">
          Already have an account?{' '}
          <a href="/login" className="text-blue-600 hover:text-blue-700">
            Sign in
          </a>
        </p>
      </div>
    </div>
  );
}
```

---

## React Native

### Validation Service

#### services/passwordValidation.service.ts

```typescript
/**
 * passwordValidation.service.ts
 *
 * Password validation service for React Native 0.83.x applications.
 * Provides password strength checking and breach detection compatible
 * with React Native's crypto APIs.
 *
 * Compatible with React Native 0.83.x, TypeScript 5.9
 */

import * as Crypto from 'expo-crypto';

/**
 * Password strength requirements interface.
 */
export interface PasswordStrength {
  hasLength: boolean;
  hasUppercase: boolean;
  hasLowercase: boolean;
  hasNumber: boolean;
  hasSpecial: boolean;
}

/**
 * Password validation result interface.
 */
export interface PasswordValidationResult {
  isValid: boolean;
  errors: string[];
  strength: PasswordStrength;
}

/**
 * Validates password strength requirements.
 *
 * Checks minimum length, character complexity, and optionally verifies
 * the password has not been found in known data breaches.
 *
 * @param password - The password to validate
 * @param email - Optional email to ensure password doesn't contain it
 * @returns Validation result with strength indicators and errors
 *
 * @example
 * const result = validatePasswordStrength('MyP@ssw0rd123');
 * if (result.isValid) {
 *   // Password meets all requirements
 * }
 */
export function validatePasswordStrength(
  password: string,
  email?: string
): PasswordValidationResult {
  const strength: PasswordStrength = {
    hasLength: password.length >= 12,
    hasUppercase: /[A-Z]/.test(password),
    hasLowercase: /[a-z]/.test(password),
    hasNumber: /[0-9]/.test(password),
    hasSpecial: /[!@#$%^&*()_+\-=\[\]{}|;:,.<>?]/.test(password),
  };

  const errors: string[] = [];

  if (!strength.hasLength) {
    errors.push('Password must be at least 12 characters.');
  }
  if (!strength.hasUppercase) {
    errors.push('Password must contain an uppercase letter.');
  }
  if (!strength.hasLowercase) {
    errors.push('Password must contain a lowercase letter.');
  }
  if (!strength.hasNumber) {
    errors.push('Password must contain a number.');
  }
  if (!strength.hasSpecial) {
    errors.push('Password must contain a special character.');
  }

  // Check if password contains email
  if (email) {
    const emailPrefix = email.split('@')[0].toLowerCase();
    if (password.toLowerCase().includes(emailPrefix)) {
      errors.push('Password cannot contain your email address.');
    }
  }

  return {
    isValid: Object.values(strength).every(Boolean) && errors.length === 0,
    errors,
    strength,
  };
}

/**
 * Checks if password has been found in data breaches.
 *
 * Uses the HaveIBeenPwned API with k-Anonymity model to check passwords
 * without sending the full password hash.
 *
 * @param password - The password to check
 * @returns Promise resolving to true if password has been breached
 *
 * @example
 * const breached = await checkPasswordBreached('password123');
 * if (breached) {
 *   // Show warning to user
 * }
 */
export async function checkPasswordBreached(
  password: string
): Promise<boolean> {
  try {
    // Generate SHA-1 hash using expo-crypto
    const hash = await Crypto.digestStringAsync(
      Crypto.CryptoDigestAlgorithm.SHA1,
      password
    );
    const hashUpper = hash.toUpperCase();
    const prefix = hashUpper.slice(0, 5);
    const suffix = hashUpper.slice(5);

    // Query HaveIBeenPwned API
    const response = await fetch(
      `https://api.pwnedpasswords.com/range/${prefix}`,
      {
        method: 'GET',
        headers: {
          'Add-Padding': 'true',
        },
      }
    );

    if (!response.ok) {
      return false; // Fail open
    }

    const text = await response.text();
    return text.includes(suffix);
  } catch (error) {
    console.warn('Password breach check failed:', error);
    return false; // Fail open
  }
}
```

### Password Input Component

#### components/PasswordInput.tsx

```typescript
/**
 * PasswordInput.tsx
 *
 * Password input component for React Native with real-time validation.
 * Styled with NativeWind 4.x (Tailwind CSS for React Native).
 *
 * Compatible with React Native 0.83.x, TypeScript 5.9, NativeWind 4.x
 */

import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  ActivityIndicator,
} from 'react-native';
import {
  validatePasswordStrength,
  checkPasswordBreached,
  type PasswordStrength,
} from '@/services/passwordValidation.service';

/**
 * Props for the PasswordInput component.
 */
interface PasswordInputProps {
  /** Current password value */
  value: string;
  /** Callback when password changes */
  onChangeText: (text: string) => void;
  /** Optional email for validation */
  email?: string;
  /** Whether to show strength indicator */
  showStrength?: boolean;
  /** Whether to check against breach database */
  checkBreached?: boolean;
  /** Additional className for NativeWind */
  className?: string;
}

/**
 * Password input field with real-time validation and strength indicator.
 *
 * Displays password requirements and optionally checks against the
 * HaveIBeenPwned database.
 *
 * @example
 * <PasswordInput
 *   value={password}
 *   onChangeText={setPassword}
 *   email={email}
 *   showStrength={true}
 *   checkBreached={true}
 * />
 */
export function PasswordInput({
  value,
  onChangeText,
  email,
  showStrength = true,
  checkBreached = true,
  className = '',
}: PasswordInputProps) {
  const [showPassword, setShowPassword] = useState(false);
  const [strength, setStrength] = useState<PasswordStrength>({
    hasLength: false,
    hasUppercase: false,
    hasLowercase: false,
    hasNumber: false,
    hasSpecial: false,
  });
  const [isBreached, setIsBreached] = useState<boolean | null>(null);
  const [isCheckingBreached, setIsCheckingBreached] = useState(false);

  /**
   * Updates password strength in real-time.
   */
  useEffect(() => {
    const result = validatePasswordStrength(value, email);
    setStrength(result.strength);

    // Debounce breach check
    const timer = setTimeout(async () => {
      if (checkBreached && value.length >= 12) {
        setIsCheckingBreached(true);
        const breached = await checkPasswordBreached(value);
        setIsBreached(breached);
        setIsCheckingBreached(false);
      } else {
        setIsBreached(null);
      }
    }, 500);

    return () => clearTimeout(timer);
  }, [value, email, checkBreached]);

  return (
    <View className={`${className}`}>
      <Text className="text-sm font-medium mb-1 text-grey-700">Password</Text>

      {/* Password Input with Toggle */}
      <View className="relative">
        <TextInput
          value={value}
          onChangeText={onChangeText}
          secureTextEntry={!showPassword}
          autoCapitalize="none"
          autoCorrect={false}
          textContentType="newPassword"
          className="w-full px-3 py-3 border border-grey-300 rounded-lg text-base pr-20"
        />
        <TouchableOpacity
          onPress={() => setShowPassword(!showPassword)}
          className="absolute right-2 top-3"
        >
          <Text className="text-sm text-grey-600 font-medium">
            {showPassword ? 'Hide' : 'Show'}
          </Text>
        </TouchableOpacity>
      </View>

      {/* Strength Indicator */}
      {showStrength && value.length > 0 && (
        <View className="mt-3 space-y-2">
          <Text className="text-sm font-medium text-grey-700">
            Password Requirements:
          </Text>

          <RequirementItem met={strength.hasLength}>
            At least 12 characters
          </RequirementItem>
          <RequirementItem met={strength.hasUppercase}>
            One uppercase letter (A-Z)
          </RequirementItem>
          <RequirementItem met={strength.hasLowercase}>
            One lowercase letter (a-z)
          </RequirementItem>
          <RequirementItem met={strength.hasNumber}>
            One number (0-9)
          </RequirementItem>
          <RequirementItem met={strength.hasSpecial}>
            One special character (!@#$%...)
          </RequirementItem>

          {/* Breach Check Indicator */}
          {checkBreached && (
            <View className="flex-row items-centre gap-2 mt-2">
              {isCheckingBreached ? (
                <>
                  <ActivityIndicator size="small" color="#6B7280" />
                  <Text className="text-sm text-grey-600">
                    Checking password security...
                  </Text>
                </>
              ) : isBreached !== null ? (
                <RequirementItem met={!isBreached}>
                  {isBreached
                    ? 'Found in data breach - choose different password'
                    : 'Not found in known data breaches'}
                </RequirementItem>
              ) : null}
            </View>
          )}
        </View>
      )}
    </View>
  );
}

/**
 * Individual requirement indicator component.
 */
function RequirementItem({
  met,
  children,
}: {
  met: boolean;
  children: React.ReactNode;
}) {
  return (
    <View className="flex-row items-centre gap-2">
      <Text className={met ? 'text-green-600' : 'text-grey-400'}>
        {met ? '✓' : '○'}
      </Text>
      <Text className={`text-sm ${met ? 'text-green-600' : 'text-grey-600'}`}>
        {children}
      </Text>
    </View>
  );
}
```

#### screens/RegisterScreen.tsx

```typescript
/**
 * RegisterScreen.tsx
 *
 * User registration screen for React Native application.
 * Uses NativeWind 4.x for styling and includes password validation.
 *
 * Compatible with React Native 0.83.x, TypeScript 5.9, NativeWind 4.x
 */

import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  ScrollView,
  KeyboardAvoidingView,
  Platform,
  Alert,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { PasswordInput } from '@/components/PasswordInput';
import { validatePasswordStrength } from '@/services/passwordValidation.service';

/**
 * Registration screen component.
 *
 * Provides a registration form with real-time password validation
 * and API submission.
 */
export function RegisterScreen() {
  const navigation = useNavigation();
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    password: '',
    confirmPassword: '',
  });
  const [isSubmitting, setIsSubmitting] = useState(false);

  /**
   * Handles form submission with validation.
   */
  const handleSubmit = async () => {
    // Validate password
    const passwordValidation = validatePasswordStrength(
      formData.password,
      formData.email
    );

    if (!passwordValidation.isValid) {
      Alert.alert(
        'Invalid Password',
        passwordValidation.errors.join('\n'),
        [{ text: 'OK' }]
      );
      return;
    }

    // Validate password confirmation
    if (formData.password !== formData.confirmPassword) {
      Alert.alert('Error', 'Passwords do not match.', [{ text: 'OK' }]);
      return;
    }

    setIsSubmitting(true);

    try {
      const response = await fetch('https://api.yourapp.com/auth/register', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          name: formData.name,
          email: formData.email,
          password: formData.password,
        }),
      });

      const data = await response.json();

      if (!response.ok) {
        Alert.alert('Registration Failed', data.message || 'Please try again.');
        setIsSubmitting(false);
        return;
      }

      Alert.alert('Success', 'Account created successfully!', [
        { text: 'OK', onPress: () => navigation.navigate('Dashboard' as never) },
      ]);
    } catch (error) {
      Alert.alert('Error', 'An error occurred. Please try again.');
      setIsSubmitting(false);
    }
  };

  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      className="flex-1"
    >
      <ScrollView className="flex-1 bg-grey-50">
        <View className="flex-1 px-6 py-12">
          <View className="bg-white p-6 rounded-lg shadow">
            <Text className="text-2xl font-bold mb-6">Create Account</Text>

            {/* Name Field */}
            <View className="mb-4">
              <Text className="text-sm font-medium mb-1 text-grey-700">
                Name
              </Text>
              <TextInput
                value={formData.name}
                onChangeText={(text) =>
                  setFormData({ ...formData, name: text })
                }
                placeholder="Your Name"
                autoCapitalize="words"
                className="w-full px-3 py-3 border border-grey-300 rounded-lg text-base"
              />
            </View>

            {/* Email Field */}
            <View className="mb-4">
              <Text className="text-sm font-medium mb-1 text-grey-700">
                Email
              </Text>
              <TextInput
                value={formData.email}
                onChangeText={(text) =>
                  setFormData({ ...formData, email: text })
                }
                placeholder="user@example.com"
                keyboardType="email-address"
                autoCapitalize="none"
                autoCorrect={false}
                textContentType="emailAddress"
                className="w-full px-3 py-3 border border-grey-300 rounded-lg text-base"
              />
            </View>

            {/* Password Field */}
            <View className="mb-4">
              <PasswordInput
                value={formData.password}
                onChangeText={(text) =>
                  setFormData({ ...formData, password: text })
                }
                email={formData.email}
                showStrength={true}
                checkBreached={true}
              />
            </View>

            {/* Confirm Password Field */}
            <View className="mb-6">
              <Text className="text-sm font-medium mb-1 text-grey-700">
                Confirm Password
              </Text>
              <TextInput
                value={formData.confirmPassword}
                onChangeText={(text) =>
                  setFormData({ ...formData, confirmPassword: text })
                }
                secureTextEntry
                autoCapitalize="none"
                autoCorrect={false}
                textContentType="newPassword"
                className="w-full px-3 py-3 border border-grey-300 rounded-lg text-base"
              />
            </View>

            {/* Submit Button */}
            <TouchableOpacity
              onPress={handleSubmit}
              disabled={isSubmitting}
              className={`w-full py-3 px-4 rounded-lg ${
                isSubmitting ? 'bg-blue-400' : 'bg-blue-600'
              }`}
            >
              <Text className="text-white text-centre text-base font-semibold">
                {isSubmitting ? 'Creating Account...' : 'Create Account'}
              </Text>
            </TouchableOpacity>

            {/* Login Link */}
            <View className="mt-4 flex-row justify-centre">
              <Text className="text-sm text-grey-600">
                Already have an account?{' '}
              </Text>
              <TouchableOpacity onPress={() => navigation.navigate('Login' as never)}>
                <Text className="text-sm text-blue-600 font-medium">
                  Sign in
                </Text>
              </TouchableOpacity>
            </View>
          </View>
        </View>
      </ScrollView>
    </KeyboardAvoidingView>
  );
}
```
