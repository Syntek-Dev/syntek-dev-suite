# Email Templates

## Overview

Reusable email template architecture with shared header, footer, and styling components. Individual notifications only define their specific body content.

## Metadata

| Property            | Value                                             |
| ------------------- | ------------------------------------------------- |
| **Example Version** | 2.0.0                                             |
| **Last Updated**    | 2025-12                                           |
| **Laravel**         | 12.x                                              |
| **PHP**             | 8.4                                               |
| **Django**          | 6.x                                               |
| **Python**          | 3.14                                              |
| **Next.js**         | 16.x                                              |
| **React**           | 19.x                                              |
| **React Native**    | N/A (emails sent from backend)                    |
| **Stacks**          | TALL, Django/Wagtail, React/Next.js, React Native |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [Template Structure](#template-structure)
  - [Laravel (TALL Stack)](#laravel-tall-stack)
  - [Django/Wagtail](#djangowagtail)
  - [React/Next.js](#reactnextjs)
  - [React Native](#react-native)
- [TALL Stack (Laravel 12.x / PHP 8.4)](#tall-stack-laravel-12x--php-84)
- [Base Layout - Laravel](#base-layout---laravel)
  - [resources/views/emails/layouts/base.blade.php](#resourcesviewsemailslayoutsbasebladephp)
- [Shared Header - Laravel](#shared-header---laravel)
  - [resources/views/emails/components/header.blade.php](#resourcesviewsemailscomponentsheaderbladephp)
- [Shared Footer - Laravel](#shared-footer---laravel)
  - [resources/views/emails/components/footer.blade.php](#resourcesviewsemailscomponentsfooterbladephp)
- [Shared Styles - Laravel](#shared-styles---laravel)
  - [resources/views/emails/components/styles.blade.php](#resourcesviewsemailscomponentsstylesbladephp)
- [Partials - Laravel](#partials---laravel)
  - [resources/views/emails/partials/button.blade.php](#resourcesviewsemailspartialsbuttonbladephp)
  - [resources/views/emails/partials/card.blade.php](#resourcesviewsemailspartialscardbladephp)
  - [resources/views/emails/partials/alert.blade.php](#resourcesviewsemailspartialsalertbladephp)
- [Example Notifications - Laravel](#example-notifications---laravel)
  - [resources/views/emails/notifications/welcome.blade.php](#resourcesviewsemailsnotificationswelcomebladephp)
  - [resources/views/emails/notifications/password-reset.blade.php](#resourcesviewsemailsnotificationspassword-resetbladephp)
  - [resources/views/emails/notifications/order-confirmation.blade.php](#resourcesviewsemailsnotificationsorder-confirmationbladephp)
- [Mailable Class - Laravel](#mailable-class---laravel)
  - [app/Mail/WelcomeEmail.php](#appmailwelcomeemailphp)
- [Brand Configuration - Laravel](#brand-configuration---laravel)
  - [templates/emails/layouts/base.html](#templatesemailslayoutsbasehtml)
  - [config/brand.php](#configbrandphp)


## Template Structure

### Laravel (TALL Stack)

```
resources/views/emails/
├── layouts/
│   └── base.blade.php          # Master layout (pulls in header, footer, styles)
├── components/
│   ├── header.blade.php        # Reusable branded header (logo, navigation)
│   ├── footer.blade.php        # Reusable footer (legal, unsubscribe, social)
│   └── styles.blade.php        # Shared CSS/inline styles
├── partials/
│   ├── button.blade.php        # Branded CTA button component
│   ├── card.blade.php          # Content card component
│   └── alert.blade.php         # Alert/warning box component
└── notifications/
    ├── welcome.blade.php       # Body content ONLY
    ├── password-reset.blade.php
    ├── order-confirmation.blade.php
    └── ...
```

### Django/Wagtail

```
templates/emails/
├── layouts/
│   └── base.html               # Master layout (extends with blocks)
├── components/
│   ├── header.html             # Reusable branded header (logo, navigation)
│   ├── footer.html             # Reusable footer (legal, unsubscribe, social)
│   └── styles.html             # Shared CSS/inline styles
├── partials/
│   ├── button.html             # Branded CTA button component
│   ├── card.html               # Content card component
│   └── alert.html              # Alert/warning box component
└── notifications/
    ├── welcome.html            # Body content ONLY (extends base.html)
    ├── password_reset.html
    ├── order_confirmation.html
    └── ...
```

### React/Next.js

```
emails/
├── components/
│   ├── Layout.tsx              # Master layout component
│   ├── Header.tsx              # Reusable branded header
│   ├── Footer.tsx              # Reusable footer
│   ├── Button.tsx              # Branded CTA button component
│   ├── Card.tsx                # Content card component
│   └── Alert.tsx               # Alert/warning box component
├── templates/
│   ├── Welcome.tsx             # Welcome email template
│   ├── PasswordReset.tsx       # Password reset email template
│   ├── OrderConfirmation.tsx   # Order confirmation email template
│   └── ...
├── utils/
│   ├── render.ts               # Email rendering utilities
│   └── config.ts               # Brand configuration
└── index.ts                    # Export all templates
```

### React Native

React Native applications do not send emails directly. Email functionality is handled by the backend API (Laravel, Django, or Next.js API routes). The mobile app makes API requests to trigger email notifications, which are then sent using the backend's email templates and service.

**Backend Integration:**
- Use GraphQL mutations or REST API endpoints to trigger email notifications
- Pass user data and email type to the backend
- Backend handles template rendering and email delivery
- Mobile app receives confirmation of email dispatch

---

## TALL Stack (Laravel 12.x / PHP 8.4)

---

## Base Layout - Laravel

### resources/views/emails/layouts/base.blade.php

```blade
{{--
  Base email layout template for Laravel 12.x / PHP 8.4

  This template provides the master layout structure for all email notifications.
  It includes the shared header, footer, and styles components, and yields a
  content section for individual email templates to populate.

  @yields content - Main email body content populated by child templates
--}}
<!DOCTYPE html>
<html>
<head>
  @include('emails.components.styles')
</head>
<body>
  <div class="email-wrapper">
    @include('emails.components.header')

    <main class="email-body">
      @yield('content')
    </main>

    @include('emails.components.footer')
  </div>
</body>
</html>
```

---

## Shared Header - Laravel

### resources/views/emails/components/header.blade.php

```blade
{{--
  Shared email header component

  Displays the branded logo with a link to the application homepage.
  Consistent across all email notifications.

  @uses config('app.url') - Application URL for logo link
  @uses config('brand.logo_url') - Brand logo image URL
  @uses config('brand.name') - Brand name for alt text
--}}
<header class="email-header">
  <a href="{{ config('app.url') }}">
    <img src="{{ config('brand.logo_url') }}" alt="{{ config('brand.name') }}" class="logo" />
  </a>
</header>
```

---

## Shared Footer - Laravel

### resources/views/emails/components/footer.blade.php

```blade
{{--
  Shared email footer component

  Includes social media links, copyright notice, legal links, physical address,
  and optional unsubscribe link. Consistent across all email notifications.

  @uses config('brand.social_links') - Array of social media platform URLs
  @uses config('brand.name') - Brand name for copyright
  @uses config('brand.privacy_url') - Privacy policy URL
  @uses config('brand.terms_url') - Terms of service URL
  @uses config('brand.address') - Physical mailing address
  @var string|null $unsubscribeUrl - Optional unsubscribe URL passed from parent template
--}}
<footer class="email-footer">
  <div class="social-links">
    @foreach(config('brand.social_links') as $platform => $url)
      @if($url)
        <a href="{{ $url }}">{{ ucfirst($platform) }}</a>
      @endif
    @endforeach
  </div>

  <p class="legal">
    &copy; {{ date('Y') }} {{ config('brand.name') }}. All rights reserved.
  </p>

  <div class="footer-links">
    <a href="{{ config('brand.privacy_url') }}">Privacy Policy</a>
    <a href="{{ config('brand.terms_url') }}">Terms of Service</a>
    @if(isset($unsubscribeUrl))
      <a href="{{ $unsubscribeUrl }}">Unsubscribe</a>
    @endif
  </div>

  <p class="address">
    {{ config('brand.address') }}
  </p>
</footer>
```

---

## Shared Styles - Laravel

### resources/views/emails/components/styles.blade.php

```blade
{{--
  Shared email styles component

  Provides all CSS styling for email templates using inline styles and CSS variables
  for brand customisation. Includes reset styles, layout styles, and reusable
  component styles.

  @uses config('brand.primary_color') - Primary brand colour
  @uses config('brand.secondary_color') - Secondary brand colour
  @uses config('brand.font_family') - Brand font family
--}}
<style>
  /* Brand Variables */
  :root {
    --brand-primary: {{ config('brand.primary_color', '#1a73e8') }};
    --brand-secondary: {{ config('brand.secondary_color', '#4285f4') }};
    --brand-font: {{ config('brand.font_family', 'Inter, Arial, sans-serif') }};
    --text-color: #333333;
    --muted-color: #666666;
    --background: #f5f5f5;
  }

  /* Email Reset & Base Styles */
  body {
    margin: 0;
    padding: 0;
    font-family: var(--brand-font);
    background-color: var(--background);
  }

  .email-wrapper {
    max-width: 600px;
    margin: 0 auto;
    background-color: #ffffff;
  }

  .email-header {
    padding: 24px;
    text-align: centre;
    border-bottom: 1px solid #eeeeee;
  }

  .email-header .logo {
    max-height: 48px;
    width: auto;
  }

  .email-body {
    padding: 32px 24px;
  }

  .email-footer {
    padding: 24px;
    background-color: #f9f9f9;
    text-align: centre;
    font-size: 12px;
    colour: var(--muted-color);
  }

  /* Reusable Components */
  .btn-primary {
    display: inline-block;
    padding: 12px 24px;
    background-color: var(--brand-primary);
    colour: #ffffff !important;
    text-decoration: none;
    border-radius: 4px;
    font-weight: 600;
  }

  .btn-primary:hover {
    background-color: var(--brand-secondary);
  }

  .content-card {
    background-color: #f9f9f9;
    border-radius: 8px;
    padding: 16px;
    margin: 16px 0;
  }

  .alert-box {
    padding: 12px 16px;
    border-radius: 4px;
    margin: 16px 0;
  }

  .alert-warning {
    background-color: #fff3cd;
    border-left: 4px solid #ffc107;
  }

  .alert-info {
    background-color: #e7f3ff;
    border-left: 4px solid #2196f3;
  }

  .alert-success {
    background-color: #d4edda;
    border-left: 4px solid #28a745;
  }

  /* Typography */
  h1 {
    colour: var(--text-color);
    font-size: 24px;
    margin-bottom: 16px;
  }

  h2 {
    colour: var(--text-color);
    font-size: 20px;
    margin-bottom: 12px;
  }

  p {
    colour: var(--text-color);
    line-height: 1.6;
    margin-bottom: 12px;
  }

  a {
    colour: var(--brand-primary);
    text-decoration: none;
  }
</style>
```

---

## Partials - Laravel

### resources/views/emails/partials/button.blade.php

```blade
{{--
  Reusable button partial for email templates

  @param string $url - The URL the button should link to
  @param string $text - The button text to display
  @param string $variant - Button style variant (primary, secondary) [default: primary]
--}}
@php
  $class = $variant === 'secondary' ? 'btn-secondary' : 'btn-primary';
@endphp

<a href="{{ $url }}" class="{{ $class }}">{{ $text }}</a>
```

### resources/views/emails/partials/card.blade.php

```blade
{{--
  Reusable content card partial for email templates

  @param string $title - Card title
  @param string $content - Card body content
--}}
<div class="content-card">
  <h3>{{ $title }}</h3>
  <p>{{ $content }}</p>
</div>
```

### resources/views/emails/partials/alert.blade.php

```blade
{{--
  Reusable alert box partial for email templates

  @param string $message - Alert message content
  @param string $type - Alert type (warning, info, success) [default: info]
--}}
@php
  $class = match($type ?? 'info') {
    'warning' => 'alert-box alert-warning',
    'success' => 'alert-box alert-success',
    default => 'alert-box alert-info'
  };
@endphp

<div class="{{ $class }}">
  {!! $message !!}
</div>
```

---

## Example Notifications - Laravel

### resources/views/emails/notifications/welcome.blade.php

```blade
{{--
  Welcome email notification template

  Sent to new users upon successful registration to welcome them and guide
  them towards completing their profile and exploring the dashboard.

  @var object $user - User model instance with name and email
  @var string $dashboardUrl - URL to the user's dashboard
--}}
@extends('emails.layouts.base')

@section('content')
  <h1>Welcome to {{ config('brand.name') }}, {{ $user->name }}!</h1>

  <p>We're excited to have you on board. Here's what you can do next:</p>

  @include('emails.partials.card', [
    'title' => 'Complete Your Profile',
    'content' => 'Add your details to get personalised recommendations and make the most of your account.'
  ])

  @include('emails.partials.card', [
    'title' => 'Explore the Dashboard',
    'content' => 'Discover all the features and tools available to help you achieve your goals.'
  ])

  <p style="text-align: centre; margin-top: 24px;">
    @include('emails.partials.button', [
      'url' => $dashboardUrl,
      'text' => 'Go to Dashboard'
    ])
  </p>

  <p style="colour: #666; font-size: 14px; margin-top: 24px;">
    If you have any questions, feel free to reply to this email or visit our help centre.
  </p>
@endsection
```

### resources/views/emails/notifications/password-reset.blade.php

```blade
{{--
  Password reset email notification template

  Sent when a user requests a password reset. Includes a secure reset link
  that expires after 60 minutes.

  @var object $user - User model instance with name and email
  @var string $resetUrl - Secure password reset URL with token
  @var int $expiryMinutes - Number of minutes until the reset link expires [default: 60]
--}}
@extends('emails.layouts.base')

@section('content')
  <h1>Reset Your Password</h1>

  <p>Hi {{ $user->name }},</p>

  <p>We received a request to reset your password. Click the button below to create a new password:</p>

  <p style="text-align: centre; margin: 24px 0;">
    @include('emails.partials.button', [
      'url' => $resetUrl,
      'text' => 'Reset Password'
    ])
  </p>

  @include('emails.partials.alert', [
    'type' => 'warning',
    'message' => '<strong>This link expires in ' . ($expiryMinutes ?? 60) . ' minutes.</strong><br>If you didn\'t request this, you can safely ignore this email.'
  ])

  <p style="font-size: 12px; colour: #666;">
    If the button doesn't work, copy and paste this URL into your browser:<br>
    <a href="{{ $resetUrl }}">{{ $resetUrl }}</a>
  </p>
@endsection
```

### resources/views/emails/notifications/order-confirmation.blade.php

```blade
{{--
  Order confirmation email notification template

  Sent after a successful order to provide the customer with order details
  and tracking information.

  @var object $user - User model instance with name and email
  @var object $order - Order model instance with order details
  @var string $trackingUrl - URL to track the order
--}}
@extends('emails.layouts.base')

@section('content')
  <h1>Order Confirmation</h1>

  <p>Hi {{ $user->name }},</p>

  <p>Thank you for your order! We're processing it now and will send you an update when it ships.</p>

  @include('emails.partials.alert', [
    'type' => 'success',
    'message' => '<strong>Order Number:</strong> #{{ $order->order_number }}'
  ])

  <div class="content-card">
    <h3>Order Summary</h3>
    @foreach($order->items as $item)
      <p><strong>{{ $item->name }}</strong> x{{ $item->quantity }} - £{{ number_format($item->total, 2) }}</p>
    @endforeach
    <hr>
    <p><strong>Total:</strong> £{{ number_format($order->total, 2) }}</p>
  </div>

  <p style="text-align: centre; margin-top: 24px;">
    @include('emails.partials.button', [
      'url' => $trackingUrl,
      'text' => 'Track Your Order'
    ])
  </p>

  <p style="colour: #666; font-size: 14px; margin-top: 24px;">
    We'll send you another email when your order ships. If you have any questions, please contact our support team.
  </p>
@endsection
```

---

## Mailable Class - Laravel

### app/Mail/WelcomeEmail.php

```php
<?php

namespace App\Mail;

use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

/**
 * Welcome email mailable class
 *
 * Sends a welcome email to newly registered users with a link to their dashboard.
 * Uses the welcome email template and passes user data and dashboard URL.
 *
 * @package App\Mail
 */
class WelcomeEmail extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     *
     * @param User $user The user to send the welcome email to
     */
    public function __construct(
        public User $user,
    ) {}

    /**
     * Get the message envelope.
     *
     * Defines the email subject line.
     *
     * @return \Illuminate\Mail\Mailables\Envelope
     */
    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Welcome to ' . config('brand.name'),
        );
    }

    /**
     * Get the message content definition.
     *
     * Specifies the Blade template and passes required data.
     *
     * @return \Illuminate\Mail\Mailables\Content
     */
    public function content(): Content
    {
        return new Content(
            view: 'emails.notifications.welcome',
            with: [
                'user' => $this->user,
                'dashboardUrl' => route('dashboard'),
            ],
        );
    }

    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [];
    }
}
```

---

## Brand Configuration - Laravel

### templates/emails/layouts/base.html

```html
<!DOCTYPE html>
<html>
<head>
  {% include "emails/components/styles.html" %}
</head>
<body>
  <div class="email-wrapper">
    {% include "emails/components/header.html" %}

    <main class="email-body">
      {% block content %}{% endblock %}
    </main>

    {% include "emails/components/footer.html" with unsubscribe_url=unsubscribe_url %}
  </div>
</body>
</html>
```

---

### config/brand.php

```php
<?php

return [
    'name' => env('BRAND_NAME', 'Company Name'),
    'logo_url' => env('BRAND_LOGO_URL'),
    'primary_color' => env('BRAND_PRIMARY_COLOR', '#1a73e8'),
    'secondary_color' => env('BRAND_SECONDARY_COLOR', '#4285f4'),
    'font_family' => env('BRAND_FONT', 'Inter, Arial, sans-serif'),

    'from_name' => env('MAIL_FROM_NAME'),
    'from_email' => env('MAIL_FROM_ADDRESS'),
    'reply_to' => env('MAIL_REPLY_TO'),

    'privacy_url' => env('BRAND_PRIVACY_URL'),
    'terms_url' => env('BRAND_TERMS_URL'),
    'address' => env('BRAND_ADDRESS'),

    'social_links' => [
        'twitter' => env('BRAND_TWITTER_URL'),
        'linkedin' => env('BRAND_LINKEDIN_URL'),
        'facebook' => env('BRAND_FACEBOOK_URL'),
    ],
];
```
