# PII Masking for Notifications

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
| Django | 6.x | 20/12/2025 |
| Python | 3.14 | 20/12/2025 |
| Next.js | 16.x | 20/12/2025 |
| Node.js | 24.x LTS | 20/12/2025 |
| TypeScript | 5.9 | 20/12/2025 |
| React Native | 0.83.x | 20/12/2025 |
| NativeWind | 4.x | 20/12/2025 |

---

## Table of Contents

- [Metadata](#metadata)
- [Framework Versions Tested](#framework-versions-tested)
- [Overview](#overview)
- [Laravel (TALL Stack)](#laravel-tall-stack)
- [Django/Wagtail](#djangowagtail)
- [Node.js/TypeScript (React Native)](#nodejstypescript-react-native)

## Overview

PII masking helpers protect personal data in notifications by showing only partial information for verification purposes:
- **Email:** `jo****@example.com`
- **Phone:** `***-***-1234`
- **Address:** `London, SW1...`
- **Name:** `John S.`

---

## Laravel (TALL Stack)

```php
<?php
/**
 * NotificationPiiHelper.php
 *
 * Provides PII masking methods for use in notification templates.
 * Masks personal data to show only partial information for verification.
 */

namespace App\Helpers;

class NotificationPiiHelper
{
    /**
     * Masks an email address, showing only first 2 chars and domain.
     *
     * @param string $email The email address to mask
     * @return string The masked email (e.g., "jo****@example.com")
     */
    public static function maskEmail(string $email): string
    {
        [$local, $domain] = explode('@', $email);

        if (strlen($local) > 2) {
            $masked = substr($local, 0, 2) . str_repeat('*', min(strlen($local) - 2, 4));
        } else {
            $masked = $local[0] . '***';
        }

        return "{$masked}@{$domain}";
    }

    /**
     * Masks a phone number, showing only last 4 digits.
     *
     * @param string $phone The phone number to mask
     * @return string The masked phone (e.g., "***-***-1234")
     */
    public static function maskPhone(string $phone): string
    {
        $digits = preg_replace('/\D/', '', $phone);
        return '***-***-' . substr($digits, -4);
    }

    /**
     * Masks an address, showing only city and partial postcode.
     *
     * @param string $city The city name
     * @param string $postcode The full postcode
     * @return string The masked address (e.g., "London, SW1...")
     */
    public static function maskAddress(string $city, string $postcode): string
    {
        $postcodePrefix = strlen($postcode) > 3
            ? substr($postcode, 0, 3) . '...'
            : $postcode;

        return "{$city}, {$postcodePrefix}";
    }

    /**
     * Masks a name, showing only first letter of surname.
     *
     * @param string $fullName The full name to mask
     * @return string The masked name (e.g., "John S.")
     */
    public static function maskName(string $fullName): string
    {
        $parts = explode(' ', trim($fullName));

        if (count($parts) < 2) {
            return $fullName;
        }

        $firstName = $parts[0];
        $lastInitial = strtoupper(substr(end($parts), 0, 1)) . '.';

        return "{$firstName} {$lastInitial}";
    }
}
```

### Usage in Blade Template

```blade
{{-- Order confirmation email with masked PII --}}
@extends('emails.layouts.base')

@section('content')
    <p>Hi {{ NotificationPiiHelper::maskName($customer->full_name) }},</p>
    <p>Order confirmation sent to: {{ NotificationPiiHelper::maskEmail($customer->email) }}</p>
    <p>Shipping to: {{ NotificationPiiHelper::maskAddress($shippingCity, $shippingPostcode) }}</p>
@endsection
```

### Usage in Notification Class

```php
<?php
/**
 * OrderConfirmedNotification.php
 *
 * Sends order confirmation notification with masked PII for security.
 * Uses NotificationPiiHelper to mask sensitive customer data.
 */

namespace App\Notifications;

use App\Helpers\NotificationPiiHelper;
use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Notification;

class OrderConfirmedNotification extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new notification instance.
     *
     * @param Order $order The order instance
     */
    public function __construct(
        public Order $order
    ) {
        //
    }

    /**
     * Get the notification's delivery channels.
     *
     * @param mixed $notifiable The notifiable entity
     * @return array<int, string> Array of channels
     */
    public function via(mixed $notifiable): array
    {
        return ['mail', 'database'];
    }

    /**
     * Get the mail representation of the notification.
     *
     * @param mixed $notifiable The notifiable entity
     * @return MailMessage The mail message instance
     */
    public function toMail(mixed $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('Order Confirmation - Order #' . $this->order->id)
            ->greeting('Hi ' . NotificationPiiHelper::maskName($notifiable->full_name) . ',')
            ->line('Your order has been confirmed.')
            ->line('Confirmation sent to: ' . NotificationPiiHelper::maskEmail($notifiable->email))
            ->line('Shipping to: ' . NotificationPiiHelper::maskAddress(
                $this->order->shipping_city,
                $this->order->shipping_postcode
            ))
            ->action('View Order', url('/orders/' . $this->order->id))
            ->line('Thank you for your order!');
    }

    /**
     * Get the array representation of the notification.
     *
     * @param mixed $notifiable The notifiable entity
     * @return array<string, mixed> Array data for database storage
     */
    public function toArray(mixed $notifiable): array
    {
        return [
            'order_id' => $this->order->id,
            'masked_email' => NotificationPiiHelper::maskEmail($notifiable->email),
            'masked_name' => NotificationPiiHelper::maskName($notifiable->full_name),
            'masked_address' => NotificationPiiHelper::maskAddress(
                $this->order->shipping_city,
                $this->order->shipping_postcode
            ),
            'order_total' => $this->order->total,
        ];
    }
}
```

---

## Django/Wagtail

```python
"""
services/notification_pii_helper.py

Provides PII masking methods for use in notification templates.
Masks personal data to show only partial information for verification.
"""

import re
from typing import Optional


class NotificationPiiHelper:
    """Utility class for masking PII in notifications."""

    @staticmethod
    def mask_email(email: str) -> str:
        """
        Masks an email address, showing only first 2 chars and domain.

        Args:
            email: The email address to mask

        Returns:
            str: The masked email (e.g., "jo****@example.com")
        """
        local, domain = email.split('@')
        if len(local) > 2:
            masked = local[:2] + '*' * min(len(local) - 2, 4)
        else:
            masked = local[0] + '***'
        return f"{masked}@{domain}"

    @staticmethod
    def mask_phone(phone: str) -> str:
        """
        Masks a phone number, showing only last 4 digits.

        Args:
            phone: The phone number to mask

        Returns:
            str: The masked phone (e.g., "***-***-1234")
        """
        digits = re.sub(r'\D', '', phone)
        return f"***-***-{digits[-4:]}"

    @staticmethod
    def mask_address(city: str, postcode: str) -> str:
        """
        Masks an address, showing only city and partial postcode.

        Args:
            city: The city name
            postcode: The full postcode

        Returns:
            str: The masked address (e.g., "London, SW1...")
        """
        postcode_prefix = postcode[:3] + '...' if len(postcode) > 3 else postcode
        return f"{city}, {postcode_prefix}"

    @staticmethod
    def mask_name(full_name: str) -> str:
        """
        Masks a name, showing only first letter of surname.

        Args:
            full_name: The full name to mask

        Returns:
            str: The masked name (e.g., "John S.")
        """
        parts = full_name.strip().split()
        if len(parts) < 2:
            return full_name
        first_name = parts[0]
        last_initial = parts[-1][0].upper() + '.'
        return f"{first_name} {last_initial}"
```

### Template Filters

```python
"""
templatetags/pii_filters.py

Django template filters for masking PII in notification templates.
Provides convenient template tags for use in email and notification templates.
"""

from django import template
from services.notification_pii_helper import NotificationPiiHelper

register = template.Library()


@register.filter(name='mask_email')
def mask_email(value: str) -> str:
    """
    Template filter to mask email addresses.

    Args:
        value: The email address to mask

    Returns:
        str: The masked email address

    Usage:
        {{ customer.email|mask_email }}
    """
    if not value:
        return ''
    return NotificationPiiHelper.mask_email(value)


@register.filter(name='mask_phone')
def mask_phone(value: str) -> str:
    """
    Template filter to mask phone numbers.

    Args:
        value: The phone number to mask

    Returns:
        str: The masked phone number

    Usage:
        {{ customer.phone|mask_phone }}
    """
    if not value:
        return ''
    return NotificationPiiHelper.mask_phone(value)


@register.filter(name='mask_name')
def mask_name(value: str) -> str:
    """
    Template filter to mask names.

    Args:
        value: The full name to mask

    Returns:
        str: The masked name

    Usage:
        {{ customer.full_name|mask_name }}
    """
    if not value:
        return ''
    return NotificationPiiHelper.mask_name(value)


@register.filter(name='mask_postcode')
def mask_postcode(postcode: str, city: str = '') -> str:
    """
    Template filter to mask postcodes.

    Args:
        postcode: The postcode to mask
        city: Optional city name

    Returns:
        str: The masked postcode or address

    Usage:
        {{ shipping_postcode|mask_postcode }}
        {{ shipping_postcode|mask_postcode:shipping_city }}
    """
    if not postcode:
        return ''
    if city:
        return NotificationPiiHelper.mask_address(city, postcode)
    postcode_prefix = postcode[:3] + '...' if len(postcode) > 3 else postcode
    return postcode_prefix
```

### Usage in Django Template

```html
<!-- templates/emails/order_confirmation.html -->
{% extends "emails/layouts/base.html" %}
{% load pii_filters %}

{% block content %}
    <p>Hi {{ customer.full_name|mask_name }},</p>
    <p>Order confirmation sent to: {{ customer.email|mask_email }}</p>
    <p>Shipping to: {{ shipping_city }}, {{ shipping_postcode|mask_postcode }}</p>
{% endblock %}
```

### Usage in Notification Service

```python
"""
services/notification_service.py

Notification service for sending order confirmations with masked PII.
Handles email and database notifications with appropriate masking.
"""

from django.core.mail import send_mail
from django.template.loader import render_to_string
from django.utils.html import strip_tags
from typing import Dict, Any

from models import Order, Customer, Notification
from services.notification_pii_helper import NotificationPiiHelper


class NotificationService:
    """Service for handling notification delivery with PII masking."""

    @staticmethod
    def send_order_confirmation(order: Order, customer: Customer) -> None:
        """
        Sends order confirmation notification with masked PII.

        Args:
            order: The order instance
            customer: The customer instance

        Returns:
            None

        Raises:
            Exception: If email sending fails
        """
        # Prepare masked data for email
        context = {
            'customer': customer,
            'order': order,
            'masked_name': NotificationPiiHelper.mask_name(customer.full_name),
            'masked_email': NotificationPiiHelper.mask_email(customer.email),
            'masked_address': NotificationPiiHelper.mask_address(
                order.shipping_city,
                order.shipping_postcode
            ),
        }

        # Render email template
        html_message = render_to_string(
            'emails/order_confirmation.html',
            context
        )
        plain_message = strip_tags(html_message)

        # Send email
        send_mail(
            subject=f'Order Confirmation - Order #{order.id}',
            message=plain_message,
            from_email='noreply@example.com',
            recipient_list=[customer.email],
            html_message=html_message,
            fail_silently=False,
        )

        # Store notification in database with masked data
        Notification.objects.create(
            customer=customer,
            notification_type='order_confirmation',
            title=f'Order #{order.id} Confirmed',
            message=f'Your order has been confirmed. Shipping to: {context["masked_address"]}',
            metadata={
                'order_id': order.id,
                'masked_email': context['masked_email'],
                'masked_name': context['masked_name'],
                'masked_address': context['masked_address'],
                'order_total': str(order.total),
            }
        )

    @staticmethod
    def get_notification_data(notification: Notification) -> Dict[str, Any]:
        """
        Retrieves notification data with PII already masked.

        Args:
            notification: The notification instance

        Returns:
            dict: Notification data with masked PII

        Usage:
            data = NotificationService.get_notification_data(notification)
        """
        return {
            'id': notification.id,
            'type': notification.notification_type,
            'title': notification.title,
            'message': notification.message,
            'metadata': notification.metadata,
            'created_at': notification.created_at.isoformat(),
            'read': notification.read,
        }
```

---

## Node.js/TypeScript (React Native)

```typescript
/**
 * notificationPiiHelper.ts
 *
 * Provides PII masking methods for use in notification templates.
 * Masks personal data to show only partial information for verification.
 */

/**
 * Masks an email address, showing only first 2 chars and domain.
 *
 * @param email - The email address to mask
 * @returns The masked email (e.g., "jo****@example.com")
 */
export function maskEmail(email: string): string {
  const [local, domain] = email.split('@');
  const masked =
    local.length > 2
      ? local.slice(0, 2) + '*'.repeat(Math.min(local.length - 2, 4))
      : local[0] + '***';
  return `${masked}@${domain}`;
}

/**
 * Masks a phone number, showing only last 4 digits.
 *
 * @param phone - The phone number to mask
 * @returns The masked phone (e.g., "***-***-1234")
 */
export function maskPhone(phone: string): string {
  const digits = phone.replace(/\D/g, '');
  return `***-***-${digits.slice(-4)}`;
}

/**
 * Masks an address, showing only city and partial postcode.
 *
 * @param city - The city name
 * @param postcode - The full postcode
 * @returns The masked address (e.g., "London, SW1...")
 */
export function maskAddress(city: string, postcode: string): string {
  const postcodePrefix = postcode.length > 3 ? `${postcode.slice(0, 3)}...` : postcode;
  return `${city}, ${postcodePrefix}`;
}

/**
 * Masks a name, showing only first letter of surname.
 *
 * @param fullName - The full name to mask
 * @returns The masked name (e.g., "John S.")
 */
export function maskName(fullName: string): string {
  const parts = fullName.trim().split(' ');
  if (parts.length < 2) {
    return fullName;
  }
  const firstName = parts[0];
  const lastInitial = `${parts[parts.length - 1][0].toUpperCase()}.`;
  return `${firstName} ${lastInitial}`;
}
```

### Usage in React Native

```tsx
import { maskEmail, maskName, maskAddress } from '@/utils/notificationPiiHelper';

export function OrderConfirmation({ customer, shipping }) {
  return (
    <View>
      <Text>Hi {maskName(customer.fullName)},</Text>
      <Text>Order sent to: {maskEmail(customer.email)}</Text>
      <Text>Shipping to: {maskAddress(shipping.city, shipping.postcode)}</Text>
    </View>
  );
}
```
