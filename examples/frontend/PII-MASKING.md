# Frontend PII Masking Components

## Overview

Frontend components for masking Personally Identifiable Information (PII) in user interfaces across multiple stacks. Provides permission-gated reveal functionality with audit logging, following British English conventions.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
| **TALL Stack** | Laravel 12.x / PHP 8.4 / Alpine.js 3.x / Livewire 3.x / Tailwind 4.x |
| **Django/Wagtail** | Django 6.x / Python 3.14 / Jinja2 Templates |
| **React/Next.js** | Next.js 16.x / React 19.x / TypeScript 5.9 / Tailwind 4.x |
| **React Native** | React Native 0.83.x / TypeScript 5.9 / NativeWind 4.x |
| **Language** | British English |

---

## Table of Contents

- [Frontend PII Masking Components](#frontend-pii-masking-components)
  - [Overview](#overview)
  - [Metadata](#metadata)
  - [Table of Contents](#table-of-contents)
  - [TALL Stack - Laravel/Livewire/Alpine.js](#tall-stack---laravellivewirealpinejs)
    - [Blade Components](#blade-components)
    - [resources/views/components/masked-email.blade.php](#resourcesviewscomponentsmasked-emailbladephp)
    - [resources/views/components/masked-phone.blade.php](#resourcesviewscomponentsmasked-phonebladephp)
    - [Livewire Component](#livewire-component)
      - [app/Livewire/MaskedPiiField.php](#applivewiremaskedpiifieldphp)
      - [resources/views/livewire/masked-pii-field.blade.php](#resourcesviewslivewiremasked-pii-fieldbladephp)
    - [Alpine.js Masking Directive](#alpinejs-masking-directive)
      - [resources/js/alpine/pii-mask.js](#resourcesjsalpinepii-maskjs)
  - [Django/Wagtail Stack](#djangowagtail-stack)
    - [Django Template Tags](#django-template-tags)
      - [app/templatetags/pii\_filters.py](#apptemplatetagspii_filterspy)
    - [Jinja2 Filters](#jinja2-filters)
      - [app/jinja2.py](#appjinja2py)
    - [Django View Helpers](#django-view-helpers)
      - [app/utils/pii\_helpers.py](#apputilspii_helperspy)
      - [templates/pii/masked\_field.html](#templatespiimasked_fieldhtml)
  - [React/Next.js Stack](#reactnextjs-stack)
    - [PII Masking Hook](#pii-masking-hook)
  - [PII Masking - React Native](#pii-masking---react-native)
    - [src/components/MaskedText.tsx](#srccomponentsmaskedtexttsx)



---

## TALL Stack - Laravel/Livewire/Alpine.js

### Blade Components

### resources/views/components/masked-email.blade.php

```html
{{--
    Masked Email Component (Blade)

    Laravel 12.x / Alpine.js 3.x / Tailwind 4.x

    Displays a masked email address with a reveal toggle.
    Uses Alpine.js for interactive reveal functionality and audit logging.

    Usage:
    <x-masked-email :email="$user->email" :record-id="$user->id" />
--}}

@props(['email', 'recordId'])

@php
    // Mask email showing only first character and domain
    $masked = Str::mask($email, '*', 1, strpos($email, '@') - 1);
    $canReveal = auth()->user()?->hasPermission('pii.view');
@endphp

<span
    x-data="{ revealed: false }"
    class="inline-flex items-center gap-2"
>
    <span
        class="font-mono text-sm"
        x-text="revealed ? '{{ $email }}' : '{{ $masked }}'"
    ></span>

    @if($canReveal)
        <button
            @click="revealed = !revealed; if(revealed) $dispatch('pii-revealed', { type: 'email', recordId: '{{ $recordId }}' })"
            class="p-1 text-[var(--color-grey-500)] hover:text-[var(--color-grey-700)] rounded transition-colours"
            :aria-label="revealed ? 'Hide email' : 'Show email'"
            type="button"
        >
            <template x-if="!revealed">
                <x-heroicon-o-eye class="h-4 w-4" />
            </template>
            <template x-if="revealed">
                <x-heroicon-o-eye-slash class="h-4 w-4" />
            </template>
        </button>
    @endif
</span>
```

### resources/views/components/masked-phone.blade.php

```html
{{--
    Masked Phone Component (Blade)

    Laravel 12.x / Alpine.js 3.x / Tailwind 4.x

    Displays a masked phone number with a reveal toggle.
    Shows only the last 4 digits by default.

    Usage:
    <x-masked-phone :phone="$user->phone" :record-id="$user->id" />
--}}

@props(['phone', 'recordId'])

@php
    // Extract digits and mask all but last 4
    $digits = preg_replace('/\D/', '', $phone);
    $masked = '••••••' . substr($digits, -4);
    $canReveal = auth()->user()?->hasPermission('pii.view');
@endphp

<span
    x-data="{ revealed: false }"
    class="inline-flex items-center gap-2"
>
    <span
        class="font-mono text-sm"
        x-text="revealed ? '{{ $phone }}' : '{{ $masked }}'"
    ></span>

    @if($canReveal)
        <button
            @click="revealed = !revealed; if(revealed) $dispatch('pii-revealed', { type: 'phone', recordId: '{{ $recordId }}' })"
            class="p-1 text-[var(--color-grey-500)] hover:text-[var(--color-grey-700)] rounded transition-colours"
            :aria-label="revealed ? 'Hide phone' : 'Show phone'"
            type="button"
        >
            <template x-if="!revealed">
                <x-heroicon-o-eye class="h-4 w-4" />
            </template>
            <template x-if="revealed">
                <x-heroicon-o-eye-slash class="h-4 w-4" />
            </template>
        </button>
    @else
        <x-heroicon-o-lock-closed
            class="h-4 w-4 text-[var(--color-grey-400)]"
            title="Insufficient permissions"
        />
    @endif
</span>
```

### Livewire Component

#### app/Livewire/MaskedPiiField.php

```php
<?php

declare(strict_types=1);

namespace App\Livewire;

use Livewire\Component;
use App\Services\AuditLogger;

/**
 * Masked PII Field Livewire Component
 *
 * Laravel 12.x / Livewire 3.x / PHP 8.4
 *
 * Provides server-side PII masking with client-side reveal toggle.
 * Logs all PII reveal actions for audit compliance.
 */
class MaskedPiiField extends Component
{
    /**
     * The PII value to display (masked or revealed).
     */
    public string $value;

    /**
     * The type of PII being masked.
     */
    public string $piiType;

    /**
     * The record ID this PII belongs to.
     */
    public string $recordId;

    /**
     * Whether the PII is currently revealed.
     */
    public bool $revealed = false;

    /**
     * Initialise the component.
     *
     * @param string $value The PII value
     * @param string $piiType The type of PII
     * @param string $recordId The record ID
     */
    public function mount(string $value, string $piiType, string $recordId): void
    {
        $this->value = $value;
        $this->piiType = $piiType;
        $this->recordId = $recordId;
    }

    /**
     * Toggle the reveal state of the PII.
     *
     * Logs reveal actions for audit purposes.
     */
    public function toggleReveal(): void
    {
        // Check permission
        if (!auth()->user()?->can('pii.view')) {
            return;
        }

        $this->revealed = !$this->revealed;

        // Log reveal action
        if ($this->revealed) {
            app(AuditLogger::class)->log([
                'action' => 'pii.reveal',
                'pii_type' => $this->piiType,
                'record_id' => $this->recordId,
                'user_id' => auth()->id(),
                'timestamp' => now(),
            ]);
        }
    }

    /**
     * Get the masked version of the value.
     *
     * @return string The masked value
     */
    public function getMaskedValueProperty(): string
    {
        return match ($this->piiType) {
            'email' => $this->maskEmail($this->value),
            'phone' => $this->maskPhone($this->value),
            'name' => $this->maskName($this->value),
            'address' => '••••••••••',
            'dob' => '••/••/••••',
            default => '••••••••',
        };
    }

    /**
     * Mask an email address.
     */
    private function maskEmail(string $email): string
    {
        [$local, $domain] = explode('@', $email, 2);
        return ($domain) ? "{$local[0]}••••@{$domain}" : '••••@••••';
    }

    /**
     * Mask a phone number.
     */
    private function maskPhone(string $phone): string
    {
        $digits = preg_replace('/\D/', '', $phone);
        return '••••••' . substr($digits, -4);
    }

    /**
     * Mask a name.
     */
    private function maskName(string $name): string
    {
        $parts = explode(' ', $name);
        return implode(' ', array_map(fn($p) => $p[0] . '•••', $parts));
    }

    /**
     * Render the component.
     */
    public function render()
    {
        return view('livewire.masked-pii-field');
    }
}
```

#### resources/views/livewire/masked-pii-field.blade.php

```html
{{--
    Masked PII Field Livewire View

    Livewire 3.x / Alpine.js 3.x / Tailwind 4.x
--}}

<div class="inline-flex items-center gap-2">
    <span class="font-mono text-sm">
        {{ $revealed ? $value : $maskedValue }}
    </span>

    @can('pii.view')
        <button
            wire:click="toggleReveal"
            class="p-1 text-[var(--color-grey-500)] hover:text-[var(--color-grey-700)] rounded transition-colours"
            aria-label="{{ $revealed ? 'Hide value' : 'Show value' }}"
            type="button"
        >
            @if($revealed)
                <x-heroicon-o-eye-slash class="h-4 w-4" />
            @else
                <x-heroicon-o-eye class="h-4 w-4" />
            @endif
        </button>
    @else
        <x-heroicon-o-lock-closed
            class="h-4 w-4 text-[var(--color-grey-400)]"
            title="Insufficient permissions"
        />
    @endcan
</div>
```

### Alpine.js Masking Directive

#### resources/js/alpine/pii-mask.js

```javascript
/**
 * Alpine.js PII Masking Directive
 *
 * Alpine.js 3.x
 *
 * Provides a reusable Alpine.js directive for PII masking.
 *
 * Usage:
 * <div x-data x-pii-mask="{ value: 'user@example.com', type: 'email', recordId: '123' }"></div>
 */

import Alpine from 'alpinejs';

Alpine.directive('pii-mask', (el, { expression }, { evaluateLater, effect }) => {
    const getConfig = evaluateLater(expression);

    effect(() => {
        getConfig(config => {
            const { value, type, recordId } = config;

            // Initialise Alpine data
            Alpine.bind(el, {
                'x-data'() {
                    return {
                        revealed: false,
                        value: value,
                        type: type,
                        recordId: recordId,

                        /**
                         * Get the masked value based on PII type.
                         */
                        get maskedValue() {
                            if (this.revealed) return this.value;

                            switch (this.type) {
                                case 'email':
                                    return this.maskEmail(this.value);
                                case 'phone':
                                    return this.maskPhone(this.value);
                                case 'name':
                                    return this.maskName(this.value);
                                case 'address':
                                    return '••••••••••';
                                case 'dob':
                                    return '••/••/••••';
                                default:
                                    return '••••••••';
                            }
                        },

                        /**
                         * Mask an email address.
                         */
                        maskEmail(email) {
                            const [local, domain] = email.split('@');
                            return domain ? `${local[0]}••••@${domain}` : '••••@••••';
                        },

                        /**
                         * Mask a phone number.
                         */
                        maskPhone(phone) {
                            const digits = phone.replace(/\D/g, '');
                            return `••••••${digits.slice(-4)}`;
                        },

                        /**
                         * Mask a name.
                         */
                        maskName(name) {
                            return name.split(' ').map(p => p[0] + '•••').join(' ');
                        },

                        /**
                         * Toggle reveal state and dispatch event.
                         */
                        toggleReveal() {
                            this.revealed = !this.revealed;

                            if (this.revealed) {
                                this.$dispatch('pii-revealed', {
                                    type: this.type,
                                    recordId: this.recordId
                                });
                            }
                        }
                    };
                }
            });
        });
    });
});
```

---

## Django/Wagtail Stack

### Django Template Tags

#### app/templatetags/pii_filters.py

```python
"""
Django PII Masking Template Tags

Django 6.x / Python 3.14

Provides template filters for masking Personally Identifiable Information
in Django and Jinja2 templates.
"""

from django import template
from django.utils.safestring import mark_safe
import re

register = template.Library()


@register.filter(name='mask_email')
def mask_email(email: str) -> str:
    """
    Mask an email address showing only first character and domain.

    Args:
        email: The email address to mask

    Returns:
        The masked email address

    Example:
        {{ user.email|mask_email }}  # Output: j••••@example.com
    """
    if not email or '@' not in email:
        return '••••@••••'

    local, domain = email.split('@', 1)
    return f"{local[0]}••••@{domain}"


@register.filter(name='mask_phone')
def mask_phone(phone: str) -> str:
    """
    Mask a phone number showing only last 4 digits.

    Args:
        phone: The phone number to mask

    Returns:
        The masked phone number

    Example:
        {{ user.phone|mask_phone }}  # Output: ••••••1234
    """
    if not phone:
        return '••••'

    digits = re.sub(r'\D', '', phone)
    if len(digits) < 4:
        return '••••'

    return f"••••••{digits[-4:]}"


@register.filter(name='mask_name')
def mask_name(name: str) -> str:
    """
    Mask a name showing only initials.

    Args:
        name: The name to mask

    Returns:
        The masked name

    Example:
        {{ user.name|mask_name }}  # Output: J••• S•••
    """
    if not name:
        return '•••'

    parts = name.split(' ')
    return ' '.join(f"{part[0]}•••" for part in parts if part)


@register.filter(name='mask_pii')
def mask_pii(value: str, pii_type: str) -> str:
    """
    Generic PII masking filter.

    Args:
        value: The value to mask
        pii_type: The type of PII (email, phone, name, address, dob)

    Returns:
        The masked value

    Example:
        {{ user.email|mask_pii:"email" }}
    """
    match pii_type:
        case 'email':
            return mask_email(value)
        case 'phone':
            return mask_phone(value)
        case 'name':
            return mask_name(value)
        case 'address':
            return '••••••••••'
        case 'dob':
            return '••/••/••••'
        case _:
            return '••••••••'


@register.inclusion_tag('pii/masked_field.html')
def masked_pii_field(value: str, pii_type: str, record_id: str, label: str = None):
    """
    Render a masked PII field with reveal toggle.

    Args:
        value: The PII value to display
        pii_type: The type of PII
        record_id: The record ID
        label: Optional field label

    Returns:
        Context for the masked field template

    Example:
        {% masked_pii_field user.email "email" user.id "Email Address" %}
    """
    return {
        'value': value,
        'masked_value': mask_pii(value, pii_type),
        'pii_type': pii_type,
        'record_id': record_id,
        'label': label,
        'can_reveal': True,  # Set based on permission check
    }
```

### Jinja2 Filters

#### app/jinja2.py

```python
"""
Jinja2 Environment Configuration

Django 6.x / Python 3.14 / Jinja2

Configures Jinja2 environment with PII masking filters.
"""

from django.templatetags.static import static
from django.urls import reverse
from jinja2 import Environment


def environment(**options):
    """
    Configure the Jinja2 environment with custom filters and globals.

    Args:
        **options: Jinja2 environment options

    Returns:
        Configured Jinja2 environment
    """
    env = Environment(**options)

    # Add Django helpers
    env.globals.update({
        'static': static,
        'url': reverse,
    })

    # Add PII masking filters
    from app.templatetags.pii_filters import (
        mask_email,
        mask_phone,
        mask_name,
        mask_pii,
    )

    env.filters.update({
        'mask_email': mask_email,
        'mask_phone': mask_phone,
        'mask_name': mask_name,
        'mask_pii': mask_pii,
    })

    return env
```

### Django View Helpers

#### app/utils/pii_helpers.py

```python
"""
Django PII Masking Helpers

Django 6.x / Python 3.14

Provides utility functions for PII masking in Django views and templates.
"""

from typing import Any, Dict
from django.contrib.auth.models import User
from django.utils.html import format_html
from django.utils.safestring import SafeString
import re


def mask_email(email: str) -> str:
    """
    Mask an email address showing only first character and domain.

    Args:
        email: The email address to mask

    Returns:
        The masked email address
    """
    if not email or '@' not in email:
        return '••••@••••'

    local, domain = email.split('@', 1)
    return f"{local[0]}••••@{domain}"


def mask_phone(phone: str) -> str:
    """
    Mask a phone number showing only last 4 digits.

    Args:
        phone: The phone number to mask

    Returns:
        The masked phone number
    """
    if not phone:
        return '••••'

    digits = re.sub(r'\D', '', phone)
    if len(digits) < 4:
        return '••••'

    return f"••••••{digits[-4:]}"


def mask_name(name: str) -> str:
    """
    Mask a name showing only initials.

    Args:
        name: The name to mask

    Returns:
        The masked name
    """
    if not name:
        return '•••'

    parts = name.split(' ')
    return ' '.join(f"{part[0]}•••" for part in parts if part)


def get_masked_user_context(user: User, can_reveal: bool = False) -> Dict[str, Any]:
    """
    Get masked user data for template context.

    Args:
        user: The user instance
        can_reveal: Whether the current user can reveal PII

    Returns:
        Dictionary containing masked user data

    Example:
        context = get_masked_user_context(user, request.user.has_perm('pii.view'))
    """
    return {
        'id': user.id,
        'email': user.email if can_reveal else mask_email(user.email),
        'email_masked': mask_email(user.email),
        'first_name': user.first_name if can_reveal else mask_name(user.first_name),
        'last_name': user.last_name if can_reveal else mask_name(user.last_name),
        'can_reveal_pii': can_reveal,
    }


def render_masked_field(
    value: str,
    pii_type: str,
    record_id: str,
    can_reveal: bool = False
) -> SafeString:
    """
    Render a masked PII field as HTML.

    Args:
        value: The PII value
        pii_type: The type of PII
        record_id: The record ID
        can_reveal: Whether the user can reveal PII

    Returns:
        Safe HTML string for the masked field

    Example:
        html = render_masked_field(user.email, 'email', str(user.id), True)
    """
    masked = mask_email(value) if pii_type == 'email' else \
             mask_phone(value) if pii_type == 'phone' else \
             mask_name(value) if pii_type == 'name' else '••••••••'

    if can_reveal:
        return format_html(
            '<span class="pii-field" data-pii-type="{}" data-record-id="{}">'
            '<span class="pii-value font-mono text-sm">{}</span>'
            '<button type="button" class="pii-toggle" aria-label="Toggle visibility">'
            '<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">'
            '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" '
            'd="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>'
            '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" '
            'd="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"/>'
            '</svg>'
            '</button>'
            '</span>',
            pii_type,
            record_id,
            masked
        )
    else:
        return format_html(
            '<span class="pii-field">'
            '<span class="pii-value font-mono text-sm">{}</span>'
            '<svg class="h-4 w-4 text-grey-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">'
            '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" '
            'd="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>'
            '</svg>'
            '</span>',
            masked
        )
```

#### templates/pii/masked_field.html

```html
{# Masked PII Field Template - Django 6.x / Tailwind 4.x #}

<div class="inline-flex items-center gap-2"
     data-pii-type="{{ pii_type }}"
     data-record-id="{{ record_id }}">

    {% if label %}
    <label class="text-sm font-medium text-[var(--color-grey-600)] mb-1">
        {{ label }}
    </label>
    {% endif %}

    <span class="pii-value font-mono text-sm" data-revealed="false">
        {{ masked_value }}
    </span>

    {% if can_reveal %}
    <button type="button"
            class="pii-toggle p-1 text-[var(--color-grey-500)] hover:text-[var(--color-grey-700)] rounded transition-colours"
            aria-label="Toggle visibility"
            data-value="{{ value }}"
            data-masked="{{ masked_value }}">
        <svg class="h-4 w-4 pii-icon-show" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"/>
        </svg>
        <svg class="h-4 w-4 pii-icon-hide hidden" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.543-7a9.97 9.97 0 011.563-3.029m5.858.908a3 3 0 114.243 4.243M9.878 9.878l4.242 4.242M9.88 9.88l-3.29-3.29m7.532 7.532l3.29 3.29M3 3l3.59 3.59m0 0A9.953 9.953 0 0112 5c4.478 0 8.268 2.943 9.543 7a10.025 10.025 0 01-4.132 5.411m0 0L21 21"/>
        </svg>
    </button>
    {% else %}
    <svg class="h-4 w-4 text-[var(--color-grey-400)]" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
              d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
    </svg>
    {% endif %}
</div>
```

---

## React/Next.js Stack

### PII Masking Hook

## PII Masking - React Native

### src/components/MaskedText.tsx

```tsx
/**
 * MaskedText.tsx
 *
 * React Native component for displaying masked PII.
 * Provides haptic feedback on reveal for accessibility.
 */

import React, { useState, useCallback } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import * as Haptics from 'expo-haptics';
import { Ionicons } from '@expo/vector-icons';
import { useAuth } from '../hooks/useAuth';
import { auditLog } from '../services/audit';

type PiiType = 'email' | 'phone' | 'name' | 'address' | 'dob';

interface MaskedTextProps {
  /** The value to display */
  value: string;
  /** The type of PII */
  piiType: PiiType;
  /** The ID of the record this value belongs to */
  recordId: string;
  /** Text style overrides */
  textStyle?: object;
}

/**
 * Masks PII values based on type.
 */
function maskValue(value: string, type: PiiType): string {
  switch (type) {
    case 'email': {
      const [local, domain] = value.split('@');
      return domain ? `${local[0]}••••@${domain}` : '••••@••••';
    }
    case 'phone': {
      const digits = value.replace(/\D/g, '');
      return `••••••${digits.slice(-4)}`;
    }
    case 'name': {
      return value
        .split(' ')
        .map((p) => p[0] + '•••')
        .join(' ');
    }
    case 'address':
      return '••••••••••';
    case 'dob':
      return '••/••/••••';
    default:
      return '••••••••';
  }
}

/**
 * React Native component for masked PII display.
 * Provides haptic feedback and audit logging on reveal.
 */
export function MaskedText({
  value,
  piiType,
  recordId,
  textStyle,
}: MaskedTextProps) {
  const [isRevealed, setIsRevealed] = useState(false);
  const { user, hasPermission } = useAuth();
  const canReveal = hasPermission('pii.view');

  const toggleReveal = useCallback(async () => {
    if (!canReveal) return;

    const newState = !isRevealed;
    setIsRevealed(newState);

    // Haptic feedback
    await Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);

    // Audit log on reveal
    if (newState) {
      auditLog({
        action: 'pii.reveal',
        piiType,
        recordId,
        userId: user?.id,
      });
    }
  }, [isRevealed, canReveal, piiType, recordId, user?.id]);

  const displayValue = isRevealed ? value : maskValue(value, piiType);

  return (
    <View style={styles.container}>
      <Text style={[styles.text, textStyle]}>{displayValue}</Text>
      {canReveal && (
        <TouchableOpacity
          onPress={toggleReveal}
          style={styles.button}
          accessibilityLabel={isRevealed ? 'Hide value' : 'Show value'}
          accessibilityRole="button"
        >
          <Ionicons
            name={isRevealed ? 'eye-off-outline' : 'eye-outline'}
            size={18}
            color="#6b7280"
          />
        </TouchableOpacity>
      )}
      {!canReveal && (
        <Ionicons name="lock-closed-outline" size={16} color="#9ca3af" />
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    gap: 8,
  },
  text: {
    fontFamily: 'monospace',
    fontSize: 14,
    color: '#111827',
  },
  button: {
    padding: 4,
  },
});
```