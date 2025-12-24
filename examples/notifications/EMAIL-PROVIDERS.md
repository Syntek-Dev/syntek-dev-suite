# Email Provider Integrations

## Overview

Integration examples for Postmark and Mailchimp Transactional (formerly Mandrill) email APIs. Both providers offer high deliverability, detailed analytics, and robust APIs for transactional email.

**Postmark**: Best for transactional emails with strict delivery guarantees. Separates transactional from marketing emails to maintain sender reputation.

**Mailchimp Transactional**: Best when also using Mailchimp for marketing. Unified platform for all email communications with powerful templates.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
| **Postmark API** | v3 |
| **Mailchimp Transactional API** | v1.0 |
| **Laravel** | 12.x |
| **PHP** | 8.4 |
| **Django** | 6.x |
| **Python** | 3.14 |
| **Next.js** | 16.x |
| **React** | 19.x |
| **Stacks** | TALL, Django/Wagtail, React/Next.js, React Native |

---

## Table of Contents

- [Email Provider Integrations](#email-provider-integrations)
  - [Overview](#overview)
  - [Metadata](#metadata)
  - [Table of Contents](#table-of-contents)
  - [Provider Comparison](#provider-comparison)
  - [TALL Stack - Laravel 12.x](#tall-stack---laravel-12x)
    - [Postmark Configuration - Laravel](#postmark-configuration---laravel)
      - [config/mail.php (partial)](#configmailphp-partial)
      - [.env.example](#envexample)
    - [Postmark Service - Laravel](#postmark-service---laravel)
      - [app/Services/Email/PostmarkService.php](#appservicesemailpostmarkservicephp)
    - [Mailchimp Transactional Configuration - Laravel](#mailchimp-transactional-configuration---laravel)
      - [config/services.php (partial)](#configservicesphp-partial)
      - [.env.example (addition)](#envexample-addition)
    - [Mailchimp Transactional Service - Laravel](#mailchimp-transactional-service---laravel)
      - [app/Services/Email/MailchimpTransactionalService.php](#appservicesemailmailchimptransactionalservicephp)
    - [Unified Mail Service - Laravel](#unified-mail-service---laravel)
      - [app/Services/Email/UnifiedMailService.php](#appservicesemailunifiedmailservicephp)
  - [Django/Wagtail Stack - Django 6.x](#djangowagtail-stack---django-6x)
    - [Postmark Backend - Django](#postmark-backend---django)
      - [apps/core/email/postmark\_backend.py](#appscoreemailpostmark_backendpy)
    - [Mailchimp Transactional Backend - Django](#mailchimp-transactional-backend---django)
      - [apps/core/email/mailchimp\_backend.py](#appscoreemailmailchimp_backendpy)
  - [React/Next.js Stack - Next.js 16.x](#reactnextjs-stack---nextjs-16x)
    - [Postmark API Client - Next.js](#postmark-api-client---nextjs)
      - [lib/email/postmark-client.ts](#libemailpostmark-clientts)
    - [Mailchimp Transactional API Client - Next.js](#mailchimp-transactional-api-client---nextjs)
      - [lib/email/mailchimp-client.ts](#libemailmailchimp-clientts)
    - [Email Service - Next.js](#email-service---nextjs)
      - [lib/email/email-service.ts](#libemailemail-servicets)
    - [API Routes - Next.js](#api-routes---nextjs)
      - [app/api/email/send/route.ts](#appapiemailsendroutets)
      - [app/api/webhooks/postmark/route.ts](#appapiwebhookspostmarkroutets)
  - [React Native Stack](#react-native-stack)
    - [Email Integration Note - React Native](#email-integration-note---react-native)
    - [GraphQL Email Mutations - React Native](#graphql-email-mutations---react-native)
      - [graphql/mutations/email.ts](#graphqlmutationsemailts)
      - [hooks/useEmail.ts](#hooksuseemailts)



## Provider Comparison

| Feature | Postmark | Mailchimp Transactional |
|---------|----------|------------------------|
| **Focus** | Transactional only | Transactional + Marketing |
| **Deliverability** | 99%+ inbox rate | High deliverability |
| **Pricing Model** | Per email | Per email |
| **Free Tier** | 100/month | None (pay as you go) |
| **Templates** | Server-side templates | Server-side templates |
| **Webhooks** | Yes (delivery, bounce, open, click) | Yes (all events) |
| **Analytics** | Detailed per-message | Comprehensive reporting |
| **Best For** | Critical transactional emails | Multi-channel marketing |

---

## TALL Stack - Laravel 12.x

### Postmark Configuration - Laravel

#### config/mail.php (partial)

```php
<?php

/**
 * Mail configuration with Postmark integration.
 *
 * @package Laravel 12.x / PHP 8.4
 * @version 2.0.0
 */

return [
    'default' => env('MAIL_MAILER', 'postmark'),

    'mailers' => [
        'postmark' => [
            'transport' => 'postmark',
            'message_stream_id' => env('POSTMARK_MESSAGE_STREAM', 'outbound'),
        ],
    ],

    'from' => [
        'address' => env('MAIL_FROM_ADDRESS', 'noreply@example.com'),
        'name' => env('MAIL_FROM_NAME', 'Your App'),
    ],
];
```

#### .env.example

```env
# Postmark Configuration
MAIL_MAILER=postmark
POSTMARK_TOKEN=your-postmark-server-api-token
POSTMARK_MESSAGE_STREAM=outbound
MAIL_FROM_ADDRESS=noreply@yourdomain.com
MAIL_FROM_NAME="Your App Name"
```

---

### Postmark Service - Laravel

#### app/Services/Email/PostmarkService.php

```php
<?php

/**
 * PostmarkService.php
 *
 * Wrapper service for Postmark API with template support, batch sending,
 * and webhook handling. Implements the EmailProviderInterface.
 *
 * @package App\Services\Email
 * @version Laravel 12.x / PHP 8.4
 * @see https://postmarkapp.com/developer
 */

namespace App\Services\Email;

use App\Contracts\EmailProviderInterface;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class PostmarkService implements EmailProviderInterface
{
    protected string $apiToken;
    protected string $baseUrl = 'https://api.postmarkapp.com';
    protected string $messageStream;

    /**
     * Creates a new Postmark service instance.
     *
     * @param string|null $apiToken Postmark server API token
     * @param string $messageStream Message stream ID (outbound, broadcast, etc.)
     */
    public function __construct(
        ?string $apiToken = null,
        string $messageStream = 'outbound'
    ) {
        $this->apiToken = $apiToken ?? config('services.postmark.token');
        $this->messageStream = $messageStream;
    }

    /**
     * Sends a single email using Postmark.
     *
     * @param string $to Recipient email address
     * @param string $subject Email subject
     * @param string $htmlBody HTML email body
     * @param string|null $textBody Plain text body (optional)
     * @param array $options Additional options (from, replyTo, attachments, headers)
     * @return array Response data including MessageID
     */
    public function send(
        string $to,
        string $subject,
        string $htmlBody,
        ?string $textBody = null,
        array $options = []
    ): array {
        $payload = [
            'From' => $options['from'] ?? config('mail.from.address'),
            'To' => $to,
            'Subject' => $subject,
            'HtmlBody' => $htmlBody,
            'TextBody' => $textBody ?? strip_tags($htmlBody),
            'MessageStream' => $this->messageStream,
        ];

        // Optional fields
        if (isset($options['replyTo'])) {
            $payload['ReplyTo'] = $options['replyTo'];
        }

        if (isset($options['tag'])) {
            $payload['Tag'] = $options['tag'];
        }

        if (isset($options['trackOpens'])) {
            $payload['TrackOpens'] = $options['trackOpens'];
        }

        if (isset($options['trackLinks'])) {
            $payload['TrackLinks'] = $options['trackLinks'];
        }

        if (isset($options['attachments'])) {
            $payload['Attachments'] = $this->formatAttachments($options['attachments']);
        }

        if (isset($options['metadata'])) {
            $payload['Metadata'] = $options['metadata'];
        }

        $response = $this->request('POST', '/email', $payload);

        Log::info('Postmark email sent', [
            'to' => $to,
            'subject' => $subject,
            'message_id' => $response['MessageID'] ?? null,
        ]);

        return $response;
    }

    /**
     * Sends an email using a Postmark template.
     *
     * @param string $to Recipient email address
     * @param string $templateAlias Template alias or ID
     * @param array $templateModel Template variables
     * @param array $options Additional options
     * @return array Response data
     */
    public function sendWithTemplate(
        string $to,
        string $templateAlias,
        array $templateModel = [],
        array $options = []
    ): array {
        $payload = [
            'From' => $options['from'] ?? config('mail.from.address'),
            'To' => $to,
            'TemplateAlias' => $templateAlias,
            'TemplateModel' => $templateModel,
            'MessageStream' => $this->messageStream,
        ];

        if (isset($options['replyTo'])) {
            $payload['ReplyTo'] = $options['replyTo'];
        }

        if (isset($options['tag'])) {
            $payload['Tag'] = $options['tag'];
        }

        if (isset($options['metadata'])) {
            $payload['Metadata'] = $options['metadata'];
        }

        $response = $this->request('POST', '/email/withTemplate', $payload);

        Log::info('Postmark template email sent', [
            'to' => $to,
            'template' => $templateAlias,
            'message_id' => $response['MessageID'] ?? null,
        ]);

        return $response;
    }

    /**
     * Sends batch emails (up to 500 per request).
     *
     * @param array $messages Array of message payloads
     * @return array Array of responses
     */
    public function sendBatch(array $messages): array
    {
        // Postmark limit is 500 messages per batch
        $batches = array_chunk($messages, 500);
        $results = [];

        foreach ($batches as $batch) {
            $formattedBatch = array_map(function ($message) {
                return [
                    'From' => $message['from'] ?? config('mail.from.address'),
                    'To' => $message['to'],
                    'Subject' => $message['subject'],
                    'HtmlBody' => $message['htmlBody'],
                    'TextBody' => $message['textBody'] ?? strip_tags($message['htmlBody']),
                    'MessageStream' => $this->messageStream,
                    'Tag' => $message['tag'] ?? null,
                ];
            }, $batch);

            $response = $this->request('POST', '/email/batch', $formattedBatch);
            $results = array_merge($results, $response);
        }

        Log::info('Postmark batch sent', [
            'count' => count($messages),
        ]);

        return $results;
    }

    /**
     * Gets email delivery statistics.
     *
     * @param string|null $tag Optional tag to filter by
     * @param string|null $fromDate Start date (YYYY-MM-DD)
     * @param string|null $toDate End date (YYYY-MM-DD)
     * @return array Statistics data
     */
    public function getStats(
        ?string $tag = null,
        ?string $fromDate = null,
        ?string $toDate = null
    ): array {
        $params = [];

        if ($tag) {
            $params['tag'] = $tag;
        }
        if ($fromDate) {
            $params['fromdate'] = $fromDate;
        }
        if ($toDate) {
            $params['todate'] = $toDate;
        }

        return $this->request('GET', '/stats/outbound', $params);
    }

    /**
     * Gets bounce information for a message.
     *
     * @param string $messageId Postmark message ID
     * @return array|null Bounce data if exists
     */
    public function getBounce(string $messageId): ?array
    {
        try {
            return $this->request('GET', "/bounces/{$messageId}");
        } catch (\Exception $e) {
            return null;
        }
    }

    /**
     * Processes a webhook from Postmark.
     *
     * @param array $payload Webhook payload
     * @return void
     */
    public function processWebhook(array $payload): void
    {
        $recordType = $payload['RecordType'] ?? 'Unknown';

        match ($recordType) {
            'Delivery' => $this->handleDelivery($payload),
            'Bounce' => $this->handleBounce($payload),
            'SpamComplaint' => $this->handleSpamComplaint($payload),
            'Open' => $this->handleOpen($payload),
            'Click' => $this->handleClick($payload),
            default => Log::warning('Unknown Postmark webhook type', ['type' => $recordType]),
        };
    }

    /**
     * Makes an API request to Postmark.
     *
     * @param string $method HTTP method
     * @param string $endpoint API endpoint
     * @param array $data Request data
     * @return array Response data
     */
    protected function request(string $method, string $endpoint, array $data = []): array
    {
        $response = Http::withHeaders([
            'Accept' => 'application/json',
            'Content-Type' => 'application/json',
            'X-Postmark-Server-Token' => $this->apiToken,
        ])->$method($this->baseUrl . $endpoint, $data);

        if ($response->failed()) {
            Log::error('Postmark API error', [
                'endpoint' => $endpoint,
                'status' => $response->status(),
                'error' => $response->json('Message') ?? $response->body(),
            ]);

            throw new \Exception(
                $response->json('Message') ?? 'Postmark API request failed'
            );
        }

        return $response->json();
    }

    /**
     * Formats attachments for Postmark API.
     *
     * @param array $attachments Array of attachment data
     * @return array Formatted attachments
     */
    protected function formatAttachments(array $attachments): array
    {
        return array_map(function ($attachment) {
            return [
                'Name' => $attachment['name'],
                'Content' => base64_encode($attachment['content']),
                'ContentType' => $attachment['contentType'] ?? 'application/octet-stream',
            ];
        }, $attachments);
    }

    /**
     * Handles delivery webhook.
     */
    protected function handleDelivery(array $payload): void
    {
        Log::info('Email delivered', [
            'message_id' => $payload['MessageID'] ?? null,
            'recipient' => $payload['Recipient'] ?? null,
        ]);
    }

    /**
     * Handles bounce webhook.
     */
    protected function handleBounce(array $payload): void
    {
        Log::warning('Email bounced', [
            'message_id' => $payload['MessageID'] ?? null,
            'email' => $payload['Email'] ?? null,
            'type' => $payload['Type'] ?? null,
            'description' => $payload['Description'] ?? null,
        ]);

        // Mark email as bounced in database
        // event(new EmailBounced($payload['Email'], $payload['Type']));
    }

    /**
     * Handles spam complaint webhook.
     */
    protected function handleSpamComplaint(array $payload): void
    {
        Log::warning('Spam complaint received', [
            'email' => $payload['Email'] ?? null,
        ]);

        // Unsubscribe user
        // event(new SpamComplaint($payload['Email']));
    }

    /**
     * Handles open tracking webhook.
     */
    protected function handleOpen(array $payload): void
    {
        Log::info('Email opened', [
            'message_id' => $payload['MessageID'] ?? null,
            'recipient' => $payload['Recipient'] ?? null,
        ]);
    }

    /**
     * Handles click tracking webhook.
     */
    protected function handleClick(array $payload): void
    {
        Log::info('Email link clicked', [
            'message_id' => $payload['MessageID'] ?? null,
            'click_location' => $payload['ClickLocation'] ?? null,
        ]);
    }
}
```

---

### Mailchimp Transactional Configuration - Laravel

#### config/services.php (partial)

```php
<?php

/**
 * Mailchimp Transactional (Mandrill) configuration.
 *
 * @package Laravel 12.x / PHP 8.4
 * @version 2.0.0
 */

return [
    // ...other services

    'mailchimp' => [
        'transactional' => [
            'api_key' => env('MAILCHIMP_TRANSACTIONAL_API_KEY'),
            'from_email' => env('MAILCHIMP_FROM_EMAIL', 'noreply@example.com'),
            'from_name' => env('MAILCHIMP_FROM_NAME', 'Your App'),
        ],
    ],
];
```

#### .env.example (addition)

```env
# Mailchimp Transactional (Mandrill) Configuration
MAILCHIMP_TRANSACTIONAL_API_KEY=your-mandrill-api-key
MAILCHIMP_FROM_EMAIL=noreply@yourdomain.com
MAILCHIMP_FROM_NAME="Your App Name"
```

---

### Mailchimp Transactional Service - Laravel

#### app/Services/Email/MailchimpTransactionalService.php

```php
<?php

/**
 * MailchimpTransactionalService.php
 *
 * Wrapper service for Mailchimp Transactional (Mandrill) API with template support,
 * batch sending, and webhook handling. Implements the EmailProviderInterface.
 *
 * @package App\Services\Email
 * @version Laravel 12.x / PHP 8.4
 * @see https://mailchimp.com/developer/transactional/api/
 */

namespace App\Services\Email;

use App\Contracts\EmailProviderInterface;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class MailchimpTransactionalService implements EmailProviderInterface
{
    protected string $apiKey;
    protected string $baseUrl = 'https://mandrillapp.com/api/1.0';
    protected string $fromEmail;
    protected string $fromName;

    /**
     * Creates a new Mailchimp Transactional service instance.
     *
     * @param string|null $apiKey Mandrill API key
     */
    public function __construct(?string $apiKey = null)
    {
        $this->apiKey = $apiKey ?? config('services.mailchimp.transactional.api_key');
        $this->fromEmail = config('services.mailchimp.transactional.from_email');
        $this->fromName = config('services.mailchimp.transactional.from_name');
    }

    /**
     * Sends a single email using Mailchimp Transactional.
     *
     * @param string $to Recipient email address
     * @param string $subject Email subject
     * @param string $htmlBody HTML email body
     * @param string|null $textBody Plain text body (optional)
     * @param array $options Additional options
     * @return array Response data including _id
     */
    public function send(
        string $to,
        string $subject,
        string $htmlBody,
        ?string $textBody = null,
        array $options = []
    ): array {
        $message = [
            'from_email' => $options['from'] ?? $this->fromEmail,
            'from_name' => $options['fromName'] ?? $this->fromName,
            'to' => [
                ['email' => $to, 'type' => 'to'],
            ],
            'subject' => $subject,
            'html' => $htmlBody,
            'text' => $textBody ?? strip_tags($htmlBody),
        ];

        // Optional fields
        if (isset($options['replyTo'])) {
            $message['headers'] = ['Reply-To' => $options['replyTo']];
        }

        if (isset($options['tags'])) {
            $message['tags'] = $options['tags'];
        }

        if (isset($options['trackOpens'])) {
            $message['track_opens'] = $options['trackOpens'];
        }

        if (isset($options['trackClicks'])) {
            $message['track_clicks'] = $options['trackClicks'];
        }

        if (isset($options['attachments'])) {
            $message['attachments'] = $this->formatAttachments($options['attachments']);
        }

        if (isset($options['metadata'])) {
            $message['metadata'] = $options['metadata'];
        }

        if (isset($options['cc'])) {
            foreach ($options['cc'] as $ccEmail) {
                $message['to'][] = ['email' => $ccEmail, 'type' => 'cc'];
            }
        }

        if (isset($options['bcc'])) {
            foreach ($options['bcc'] as $bccEmail) {
                $message['to'][] = ['email' => $bccEmail, 'type' => 'bcc'];
            }
        }

        $response = $this->request('POST', '/messages/send.json', [
            'message' => $message,
            'async' => $options['async'] ?? false,
        ]);

        $result = $response[0] ?? [];

        Log::info('Mailchimp Transactional email sent', [
            'to' => $to,
            'subject' => $subject,
            'message_id' => $result['_id'] ?? null,
            'status' => $result['status'] ?? null,
        ]);

        return $result;
    }

    /**
     * Sends an email using a Mailchimp template.
     *
     * @param string $to Recipient email address
     * @param string $templateName Template name in Mailchimp
     * @param array $templateContent Merge variables for template
     * @param array $options Additional options
     * @return array Response data
     */
    public function sendWithTemplate(
        string $to,
        string $templateName,
        array $templateContent = [],
        array $options = []
    ): array {
        $message = [
            'from_email' => $options['from'] ?? $this->fromEmail,
            'from_name' => $options['fromName'] ?? $this->fromName,
            'to' => [
                ['email' => $to, 'type' => 'to'],
            ],
            'subject' => $options['subject'] ?? '',
        ];

        // Convert merge variables to Mailchimp format
        $mergeVars = [];
        foreach ($templateContent as $key => $value) {
            $mergeVars[] = [
                'name' => $key,
                'content' => $value,
            ];
        }

        if (isset($options['globalMergeVars'])) {
            $message['global_merge_vars'] = $options['globalMergeVars'];
        }

        if (isset($options['tags'])) {
            $message['tags'] = $options['tags'];
        }

        if (isset($options['metadata'])) {
            $message['metadata'] = $options['metadata'];
        }

        $response = $this->request('POST', '/messages/send-template.json', [
            'template_name' => $templateName,
            'template_content' => $mergeVars,
            'message' => $message,
            'async' => $options['async'] ?? false,
        ]);

        $result = $response[0] ?? [];

        Log::info('Mailchimp Transactional template email sent', [
            'to' => $to,
            'template' => $templateName,
            'message_id' => $result['_id'] ?? null,
            'status' => $result['status'] ?? null,
        ]);

        return $result;
    }

    /**
     * Sends batch emails.
     *
     * @param array $messages Array of message payloads
     * @return array Array of responses
     */
    public function sendBatch(array $messages): array
    {
        $formattedMessages = array_map(function ($message) {
            return [
                'from_email' => $message['from'] ?? $this->fromEmail,
                'from_name' => $message['fromName'] ?? $this->fromName,
                'to' => [
                    ['email' => $message['to'], 'type' => 'to'],
                ],
                'subject' => $message['subject'],
                'html' => $message['htmlBody'],
                'text' => $message['textBody'] ?? strip_tags($message['htmlBody']),
                'tags' => $message['tags'] ?? [],
            ];
        }, $messages);

        $results = [];

        foreach ($formattedMessages as $message) {
            $response = $this->request('POST', '/messages/send.json', [
                'message' => $message,
                'async' => true,
            ]);
            $results[] = $response[0] ?? [];
        }

        Log::info('Mailchimp Transactional batch sent', [
            'count' => count($messages),
        ]);

        return $results;
    }

    /**
     * Gets email delivery statistics.
     *
     * @return array Statistics data
     */
    public function getStats(): array
    {
        return $this->request('POST', '/users/info.json', []);
    }

    /**
     * Gets message information by ID.
     *
     * @param string $messageId Message ID
     * @return array Message info
     */
    public function getMessageInfo(string $messageId): array
    {
        return $this->request('POST', '/messages/info.json', [
            'id' => $messageId,
        ]);
    }

    /**
     * Searches sent messages.
     *
     * @param string $query Search query (email address, subject, etc.)
     * @param int $limit Maximum results (1-1000)
     * @return array Search results
     */
    public function searchMessages(string $query, int $limit = 100): array
    {
        return $this->request('POST', '/messages/search.json', [
            'query' => $query,
            'limit' => $limit,
        ]);
    }

    /**
     * Processes a webhook from Mailchimp Transactional.
     *
     * @param array $events Webhook events array
     * @return void
     */
    public function processWebhook(array $events): void
    {
        foreach ($events as $event) {
            $eventType = $event['event'] ?? 'unknown';

            match ($eventType) {
                'send' => $this->handleSend($event),
                'deferral' => $this->handleDeferral($event),
                'hard_bounce' => $this->handleHardBounce($event),
                'soft_bounce' => $this->handleSoftBounce($event),
                'open' => $this->handleOpen($event),
                'click' => $this->handleClick($event),
                'spam' => $this->handleSpam($event),
                'unsub' => $this->handleUnsubscribe($event),
                'reject' => $this->handleReject($event),
                default => Log::warning('Unknown Mailchimp webhook type', ['type' => $eventType]),
            };
        }
    }

    /**
     * Lists available templates.
     *
     * @return array Template list
     */
    public function listTemplates(): array
    {
        return $this->request('POST', '/templates/list.json', []);
    }

    /**
     * Makes an API request to Mailchimp Transactional.
     *
     * @param string $method HTTP method
     * @param string $endpoint API endpoint
     * @param array $data Request data
     * @return array Response data
     */
    protected function request(string $method, string $endpoint, array $data = []): array
    {
        $data['key'] = $this->apiKey;

        $response = Http::post($this->baseUrl . $endpoint, $data);

        if ($response->failed()) {
            $error = $response->json();
            Log::error('Mailchimp Transactional API error', [
                'endpoint' => $endpoint,
                'status' => $error['status'] ?? 'unknown',
                'message' => $error['message'] ?? $response->body(),
            ]);

            throw new \Exception(
                $error['message'] ?? 'Mailchimp Transactional API request failed'
            );
        }

        return $response->json();
    }

    /**
     * Formats attachments for Mailchimp API.
     *
     * @param array $attachments Array of attachment data
     * @return array Formatted attachments
     */
    protected function formatAttachments(array $attachments): array
    {
        return array_map(function ($attachment) {
            return [
                'name' => $attachment['name'],
                'content' => base64_encode($attachment['content']),
                'type' => $attachment['contentType'] ?? 'application/octet-stream',
            ];
        }, $attachments);
    }

    /**
     * Handles send confirmation webhook.
     */
    protected function handleSend(array $event): void
    {
        Log::info('Email sent via Mailchimp', [
            'message_id' => $event['_id'] ?? null,
            'email' => $event['msg']['email'] ?? null,
        ]);
    }

    /**
     * Handles deferral webhook.
     */
    protected function handleDeferral(array $event): void
    {
        Log::warning('Email deferred', [
            'message_id' => $event['_id'] ?? null,
            'email' => $event['msg']['email'] ?? null,
        ]);
    }

    /**
     * Handles hard bounce webhook.
     */
    protected function handleHardBounce(array $event): void
    {
        $email = $event['msg']['email'] ?? null;

        Log::warning('Hard bounce received', [
            'email' => $email,
            'bounce_description' => $event['msg']['bounce_description'] ?? null,
        ]);

        // Mark email as bounced
        // event(new EmailBounced($email, 'hard'));
    }

    /**
     * Handles soft bounce webhook.
     */
    protected function handleSoftBounce(array $event): void
    {
        Log::info('Soft bounce received', [
            'email' => $event['msg']['email'] ?? null,
        ]);
    }

    /**
     * Handles open tracking webhook.
     */
    protected function handleOpen(array $event): void
    {
        Log::info('Email opened', [
            'message_id' => $event['_id'] ?? null,
            'email' => $event['msg']['email'] ?? null,
        ]);
    }

    /**
     * Handles click tracking webhook.
     */
    protected function handleClick(array $event): void
    {
        Log::info('Email link clicked', [
            'message_id' => $event['_id'] ?? null,
            'url' => $event['url'] ?? null,
        ]);
    }

    /**
     * Handles spam complaint webhook.
     */
    protected function handleSpam(array $event): void
    {
        $email = $event['msg']['email'] ?? null;

        Log::warning('Spam complaint received', ['email' => $email]);

        // Unsubscribe user
        // event(new SpamComplaint($email));
    }

    /**
     * Handles unsubscribe webhook.
     */
    protected function handleUnsubscribe(array $event): void
    {
        $email = $event['msg']['email'] ?? null;

        Log::info('User unsubscribed', ['email' => $email]);

        // Handle unsubscription
        // event(new UserUnsubscribed($email));
    }

    /**
     * Handles rejection webhook.
     */
    protected function handleReject(array $event): void
    {
        Log::warning('Email rejected', [
            'email' => $event['msg']['email'] ?? null,
        ]);
    }
}
```

---

### Unified Mail Service - Laravel

#### app/Services/Email/UnifiedMailService.php

```php
<?php

/**
 * UnifiedMailService.php
 *
 * Facade for email sending that can switch between providers.
 * Supports failover between Postmark and Mailchimp Transactional.
 *
 * @package App\Services\Email
 * @version Laravel 12.x / PHP 8.4
 */

namespace App\Services\Email;

use App\Contracts\EmailProviderInterface;
use Illuminate\Support\Facades\Log;

class UnifiedMailService
{
    protected EmailProviderInterface $primary;
    protected ?EmailProviderInterface $fallback;

    /**
     * Creates a unified mail service with primary and optional fallback providers.
     *
     * @param string $primaryProvider Primary provider name ('postmark' or 'mailchimp')
     * @param string|null $fallbackProvider Fallback provider name
     */
    public function __construct(
        string $primaryProvider = 'postmark',
        ?string $fallbackProvider = null
    ) {
        $this->primary = $this->createProvider($primaryProvider);
        $this->fallback = $fallbackProvider ? $this->createProvider($fallbackProvider) : null;
    }

    /**
     * Sends an email with automatic failover.
     *
     * @param string $to Recipient email
     * @param string $subject Subject line
     * @param string $htmlBody HTML content
     * @param string|null $textBody Plain text content
     * @param array $options Additional options
     * @return array Response data
     */
    public function send(
        string $to,
        string $subject,
        string $htmlBody,
        ?string $textBody = null,
        array $options = []
    ): array {
        try {
            return $this->primary->send($to, $subject, $htmlBody, $textBody, $options);
        } catch (\Exception $e) {
            Log::error('Primary email provider failed', [
                'provider' => get_class($this->primary),
                'error' => $e->getMessage(),
            ]);

            if ($this->fallback) {
                Log::info('Falling back to secondary email provider');
                return $this->fallback->send($to, $subject, $htmlBody, $textBody, $options);
            }

            throw $e;
        }
    }

    /**
     * Sends an email using a template with automatic failover.
     *
     * @param string $to Recipient email
     * @param string $templateName Template identifier
     * @param array $templateData Template variables
     * @param array $options Additional options
     * @return array Response data
     */
    public function sendWithTemplate(
        string $to,
        string $templateName,
        array $templateData = [],
        array $options = []
    ): array {
        try {
            return $this->primary->sendWithTemplate($to, $templateName, $templateData, $options);
        } catch (\Exception $e) {
            Log::error('Primary email provider failed for template', [
                'provider' => get_class($this->primary),
                'template' => $templateName,
                'error' => $e->getMessage(),
            ]);

            if ($this->fallback) {
                Log::info('Falling back to secondary email provider for template');
                return $this->fallback->sendWithTemplate($to, $templateName, $templateData, $options);
            }

            throw $e;
        }
    }

    /**
     * Creates a provider instance by name.
     *
     * @param string $provider Provider name
     * @return EmailProviderInterface
     */
    protected function createProvider(string $provider): EmailProviderInterface
    {
        return match ($provider) {
            'postmark' => new PostmarkService(),
            'mailchimp' => new MailchimpTransactionalService(),
            default => throw new \InvalidArgumentException("Unknown email provider: {$provider}"),
        };
    }
}
```

---

## Django/Wagtail Stack - Django 6.x

### Postmark Backend - Django

#### apps/core/email/postmark_backend.py

```python
"""
Postmark email backend for Django 6.x.

Custom email backend that sends emails through the Postmark API.
Supports templates, batch sending, and webhook handling.

@package core.email
@version Django 6.x / Python 3.14
@see https://postmarkapp.com/developer
"""

import base64
import json
import logging
from typing import Any, Optional

import httpx
from django.conf import settings
from django.core.mail.backends.base import BaseEmailBackend
from django.core.mail import EmailMessage, EmailMultiAlternatives

logger = logging.getLogger(__name__)


class PostmarkBackend(BaseEmailBackend):
    """
    Django email backend for Postmark.

    Configuration in settings.py:
        EMAIL_BACKEND = 'apps.core.email.postmark_backend.PostmarkBackend'
        POSTMARK_API_TOKEN = 'your-server-token'
        POSTMARK_MESSAGE_STREAM = 'outbound'
    """

    API_URL = 'https://api.postmarkapp.com'

    def __init__(self, fail_silently: bool = False, **kwargs):
        """
        Initialises the Postmark backend.

        Args:
            fail_silently: If True, suppress exceptions
        """
        super().__init__(fail_silently=fail_silently)
        self.api_token = getattr(settings, 'POSTMARK_API_TOKEN', '')
        self.message_stream = getattr(settings, 'POSTMARK_MESSAGE_STREAM', 'outbound')
        self.client = httpx.Client(
            base_url=self.API_URL,
            headers={
                'Accept': 'application/json',
                'Content-Type': 'application/json',
                'X-Postmark-Server-Token': self.api_token,
            },
            timeout=30.0,
        )

    def send_messages(self, email_messages: list) -> int:
        """
        Sends one or more email messages.

        Args:
            email_messages: List of EmailMessage objects

        Returns:
            Number of successfully sent messages
        """
        if not email_messages:
            return 0

        num_sent = 0

        for message in email_messages:
            try:
                self._send_message(message)
                num_sent += 1
            except Exception as e:
                logger.error(f'Failed to send email via Postmark: {e}')
                if not self.fail_silently:
                    raise

        return num_sent

    def _send_message(self, message: EmailMessage) -> dict:
        """
        Sends a single email message.

        Args:
            message: EmailMessage object

        Returns:
            Postmark API response
        """
        payload = {
            'From': message.from_email,
            'To': ', '.join(message.to),
            'Subject': message.subject,
            'MessageStream': self.message_stream,
        }

        # Handle HTML content
        if isinstance(message, EmailMultiAlternatives):
            for content, mimetype in message.alternatives:
                if mimetype == 'text/html':
                    payload['HtmlBody'] = content
                    break
            payload['TextBody'] = message.body
        else:
            payload['TextBody'] = message.body

        # CC and BCC
        if message.cc:
            payload['Cc'] = ', '.join(message.cc)
        if message.bcc:
            payload['Bcc'] = ', '.join(message.bcc)

        # Reply-To
        if message.reply_to:
            payload['ReplyTo'] = ', '.join(message.reply_to)

        # Attachments
        if message.attachments:
            payload['Attachments'] = [
                {
                    'Name': name,
                    'Content': base64.b64encode(content).decode() if isinstance(content, bytes) else content,
                    'ContentType': mimetype or 'application/octet-stream',
                }
                for name, content, mimetype in message.attachments
            ]

        # Headers
        if message.extra_headers:
            payload['Headers'] = [
                {'Name': key, 'Value': value}
                for key, value in message.extra_headers.items()
            ]

        response = self.client.post('/email', json=payload)
        response.raise_for_status()

        result = response.json()
        logger.info(
            'Email sent via Postmark',
            extra={
                'to': message.to,
                'subject': message.subject,
                'message_id': result.get('MessageID'),
            }
        )

        return result

    def close(self):
        """Closes the HTTP client connection."""
        self.client.close()


class PostmarkService:
    """
    Postmark API service for template emails and advanced operations.

    Provides methods for sending template emails, batch sending,
    and handling webhooks.
    """

    API_URL = 'https://api.postmarkapp.com'

    def __init__(self):
        """Initialises the Postmark service."""
        self.api_token = getattr(settings, 'POSTMARK_API_TOKEN', '')
        self.message_stream = getattr(settings, 'POSTMARK_MESSAGE_STREAM', 'outbound')
        self.client = httpx.Client(
            base_url=self.API_URL,
            headers={
                'Accept': 'application/json',
                'Content-Type': 'application/json',
                'X-Postmark-Server-Token': self.api_token,
            },
            timeout=30.0,
        )

    def send_with_template(
        self,
        to: str,
        template_alias: str,
        template_model: dict,
        from_email: Optional[str] = None,
        **options,
    ) -> dict:
        """
        Sends an email using a Postmark template.

        Args:
            to: Recipient email address
            template_alias: Template alias or ID
            template_model: Template variables
            from_email: Sender email (optional)
            **options: Additional options (reply_to, tag, metadata)

        Returns:
            Postmark API response
        """
        payload = {
            'From': from_email or settings.DEFAULT_FROM_EMAIL,
            'To': to,
            'TemplateAlias': template_alias,
            'TemplateModel': template_model,
            'MessageStream': self.message_stream,
        }

        if options.get('reply_to'):
            payload['ReplyTo'] = options['reply_to']

        if options.get('tag'):
            payload['Tag'] = options['tag']

        if options.get('metadata'):
            payload['Metadata'] = options['metadata']

        response = self.client.post('/email/withTemplate', json=payload)
        response.raise_for_status()

        result = response.json()
        logger.info(
            'Template email sent via Postmark',
            extra={
                'to': to,
                'template': template_alias,
                'message_id': result.get('MessageID'),
            }
        )

        return result

    def send_batch(self, messages: list) -> list:
        """
        Sends batch emails (up to 500 per request).

        Args:
            messages: List of message dictionaries

        Returns:
            List of API responses
        """
        results = []

        # Postmark limit is 500 messages per batch
        for i in range(0, len(messages), 500):
            batch = messages[i:i + 500]
            formatted_batch = [
                {
                    'From': msg.get('from', settings.DEFAULT_FROM_EMAIL),
                    'To': msg['to'],
                    'Subject': msg['subject'],
                    'HtmlBody': msg.get('html_body'),
                    'TextBody': msg.get('text_body'),
                    'MessageStream': self.message_stream,
                    'Tag': msg.get('tag'),
                }
                for msg in batch
            ]

            response = self.client.post('/email/batch', json=formatted_batch)
            response.raise_for_status()
            results.extend(response.json())

        logger.info(f'Batch sent via Postmark: {len(messages)} emails')
        return results

    def get_stats(
        self,
        tag: Optional[str] = None,
        from_date: Optional[str] = None,
        to_date: Optional[str] = None,
    ) -> dict:
        """
        Gets email delivery statistics.

        Args:
            tag: Optional tag filter
            from_date: Start date (YYYY-MM-DD)
            to_date: End date (YYYY-MM-DD)

        Returns:
            Statistics data
        """
        params = {}
        if tag:
            params['tag'] = tag
        if from_date:
            params['fromdate'] = from_date
        if to_date:
            params['todate'] = to_date

        response = self.client.get('/stats/outbound', params=params)
        response.raise_for_status()
        return response.json()

    def process_webhook(self, payload: dict) -> None:
        """
        Processes a Postmark webhook.

        Args:
            payload: Webhook payload
        """
        record_type = payload.get('RecordType', 'Unknown')

        handlers = {
            'Delivery': self._handle_delivery,
            'Bounce': self._handle_bounce,
            'SpamComplaint': self._handle_spam,
            'Open': self._handle_open,
            'Click': self._handle_click,
        }

        handler = handlers.get(record_type)
        if handler:
            handler(payload)
        else:
            logger.warning(f'Unknown Postmark webhook type: {record_type}')

    def _handle_delivery(self, payload: dict) -> None:
        logger.info(
            'Email delivered',
            extra={
                'message_id': payload.get('MessageID'),
                'recipient': payload.get('Recipient'),
            }
        )

    def _handle_bounce(self, payload: dict) -> None:
        logger.warning(
            'Email bounced',
            extra={
                'email': payload.get('Email'),
                'type': payload.get('Type'),
            }
        )

    def _handle_spam(self, payload: dict) -> None:
        logger.warning('Spam complaint', extra={'email': payload.get('Email')})

    def _handle_open(self, payload: dict) -> None:
        logger.info('Email opened', extra={'message_id': payload.get('MessageID')})

    def _handle_click(self, payload: dict) -> None:
        logger.info('Email clicked', extra={'message_id': payload.get('MessageID')})

    def close(self):
        """Closes the HTTP client connection."""
        self.client.close()
```

---

### Mailchimp Transactional Backend - Django

#### apps/core/email/mailchimp_backend.py

```python
"""
Mailchimp Transactional (Mandrill) email backend for Django 6.x.

Custom email backend that sends emails through the Mailchimp Transactional API.
Supports templates, batch sending, and webhook handling.

@package core.email
@version Django 6.x / Python 3.14
@see https://mailchimp.com/developer/transactional/api/
"""

import base64
import logging
from typing import Any, Optional

import httpx
from django.conf import settings
from django.core.mail.backends.base import BaseEmailBackend
from django.core.mail import EmailMessage, EmailMultiAlternatives

logger = logging.getLogger(__name__)


class MailchimpTransactionalBackend(BaseEmailBackend):
    """
    Django email backend for Mailchimp Transactional (Mandrill).

    Configuration in settings.py:
        EMAIL_BACKEND = 'apps.core.email.mailchimp_backend.MailchimpTransactionalBackend'
        MAILCHIMP_TRANSACTIONAL_API_KEY = 'your-api-key'
    """

    API_URL = 'https://mandrillapp.com/api/1.0'

    def __init__(self, fail_silently: bool = False, **kwargs):
        """
        Initialises the Mailchimp Transactional backend.

        Args:
            fail_silently: If True, suppress exceptions
        """
        super().__init__(fail_silently=fail_silently)
        self.api_key = getattr(settings, 'MAILCHIMP_TRANSACTIONAL_API_KEY', '')
        self.client = httpx.Client(
            base_url=self.API_URL,
            timeout=30.0,
        )

    def send_messages(self, email_messages: list) -> int:
        """
        Sends one or more email messages.

        Args:
            email_messages: List of EmailMessage objects

        Returns:
            Number of successfully sent messages
        """
        if not email_messages:
            return 0

        num_sent = 0

        for message in email_messages:
            try:
                result = self._send_message(message)
                if result and result[0].get('status') in ('sent', 'queued'):
                    num_sent += 1
            except Exception as e:
                logger.error(f'Failed to send email via Mailchimp: {e}')
                if not self.fail_silently:
                    raise

        return num_sent

    def _send_message(self, message: EmailMessage) -> list:
        """
        Sends a single email message.

        Args:
            message: EmailMessage object

        Returns:
            Mailchimp API response
        """
        recipients = [{'email': addr, 'type': 'to'} for addr in message.to]

        if message.cc:
            recipients.extend([{'email': addr, 'type': 'cc'} for addr in message.cc])

        if message.bcc:
            recipients.extend([{'email': addr, 'type': 'bcc'} for addr in message.bcc])

        msg_payload = {
            'from_email': message.from_email,
            'to': recipients,
            'subject': message.subject,
        }

        # Handle HTML content
        if isinstance(message, EmailMultiAlternatives):
            for content, mimetype in message.alternatives:
                if mimetype == 'text/html':
                    msg_payload['html'] = content
                    break
            msg_payload['text'] = message.body
        else:
            msg_payload['text'] = message.body

        # Reply-To
        if message.reply_to:
            msg_payload['headers'] = {'Reply-To': ', '.join(message.reply_to)}

        # Attachments
        if message.attachments:
            msg_payload['attachments'] = [
                {
                    'name': name,
                    'content': base64.b64encode(content).decode() if isinstance(content, bytes) else content,
                    'type': mimetype or 'application/octet-stream',
                }
                for name, content, mimetype in message.attachments
            ]

        payload = {
            'key': self.api_key,
            'message': msg_payload,
        }

        response = self.client.post('/messages/send.json', json=payload)
        response.raise_for_status()

        result = response.json()
        logger.info(
            'Email sent via Mailchimp Transactional',
            extra={
                'to': message.to,
                'subject': message.subject,
                'message_id': result[0].get('_id') if result else None,
            }
        )

        return result

    def close(self):
        """Closes the HTTP client connection."""
        self.client.close()


class MailchimpTransactionalService:
    """
    Mailchimp Transactional API service for template emails and advanced operations.
    """

    API_URL = 'https://mandrillapp.com/api/1.0'

    def __init__(self):
        """Initialises the Mailchimp Transactional service."""
        self.api_key = getattr(settings, 'MAILCHIMP_TRANSACTIONAL_API_KEY', '')
        self.client = httpx.Client(base_url=self.API_URL, timeout=30.0)

    def send_with_template(
        self,
        to: str,
        template_name: str,
        template_content: dict,
        from_email: Optional[str] = None,
        subject: Optional[str] = None,
        **options,
    ) -> dict:
        """
        Sends an email using a Mailchimp template.

        Args:
            to: Recipient email address
            template_name: Template name in Mailchimp
            template_content: Merge variables for template
            from_email: Sender email (optional)
            subject: Email subject (optional)
            **options: Additional options (tags, metadata)

        Returns:
            API response
        """
        merge_vars = [
            {'name': key, 'content': value}
            for key, value in template_content.items()
        ]

        message = {
            'from_email': from_email or settings.DEFAULT_FROM_EMAIL,
            'to': [{'email': to, 'type': 'to'}],
            'subject': subject or '',
            'tags': options.get('tags', []),
            'metadata': options.get('metadata', {}),
        }

        payload = {
            'key': self.api_key,
            'template_name': template_name,
            'template_content': merge_vars,
            'message': message,
        }

        response = self.client.post('/messages/send-template.json', json=payload)
        response.raise_for_status()

        result = response.json()
        logger.info(
            'Template email sent via Mailchimp',
            extra={
                'to': to,
                'template': template_name,
                'message_id': result[0].get('_id') if result else None,
            }
        )

        return result[0] if result else {}

    def process_webhook(self, events: list) -> None:
        """
        Processes Mailchimp Transactional webhook events.

        Args:
            events: List of webhook events
        """
        for event in events:
            event_type = event.get('event', 'unknown')

            handlers = {
                'send': self._handle_send,
                'hard_bounce': self._handle_hard_bounce,
                'soft_bounce': self._handle_soft_bounce,
                'open': self._handle_open,
                'click': self._handle_click,
                'spam': self._handle_spam,
                'unsub': self._handle_unsub,
            }

            handler = handlers.get(event_type)
            if handler:
                handler(event)
            else:
                logger.warning(f'Unknown Mailchimp webhook type: {event_type}')

    def _handle_send(self, event: dict) -> None:
        logger.info('Email sent', extra={'message_id': event.get('_id')})

    def _handle_hard_bounce(self, event: dict) -> None:
        logger.warning('Hard bounce', extra={'email': event.get('msg', {}).get('email')})

    def _handle_soft_bounce(self, event: dict) -> None:
        logger.info('Soft bounce', extra={'email': event.get('msg', {}).get('email')})

    def _handle_open(self, event: dict) -> None:
        logger.info('Email opened', extra={'message_id': event.get('_id')})

    def _handle_click(self, event: dict) -> None:
        logger.info('Email clicked', extra={'url': event.get('url')})

    def _handle_spam(self, event: dict) -> None:
        logger.warning('Spam complaint', extra={'email': event.get('msg', {}).get('email')})

    def _handle_unsub(self, event: dict) -> None:
        logger.info('Unsubscribe', extra={'email': event.get('msg', {}).get('email')})

    def close(self):
        """Closes the HTTP client connection."""
        self.client.close()
```

---

## React/Next.js Stack - Next.js 16.x

### Postmark API Client - Next.js

#### lib/email/postmark-client.ts

```typescript
/**
 * postmark-client.ts
 *
 * Postmark API client for Next.js 16.x applications.
 * Provides type-safe email sending with template support.
 *
 * @package Next.js 16.x / React 19.x / TypeScript 5.9
 * @version 2.0.0
 * @see https://postmarkapp.com/developer
 */

const POSTMARK_API_URL = 'https://api.postmarkapp.com';

/**
 * Email send options.
 */
interface EmailOptions {
  from?: string;
  replyTo?: string;
  tag?: string;
  trackOpens?: boolean;
  trackLinks?: 'None' | 'HtmlAndText' | 'HtmlOnly' | 'TextOnly';
  metadata?: Record<string, string>;
  attachments?: Array<{
    name: string;
    content: string;
    contentType: string;
  }>;
}

/**
 * Postmark API response.
 */
interface PostmarkResponse {
  MessageID: string;
  SubmittedAt: string;
  To: string;
  ErrorCode: number;
  Message: string;
}

/**
 * Template model type.
 */
type TemplateModel = Record<string, unknown>;

/**
 * Postmark API client class.
 */
export class PostmarkClient {
  private apiToken: string;
  private messageStream: string;
  private defaultFrom: string;

  /**
   * Creates a new Postmark client instance.
   *
   * @param apiToken - Postmark server API token
   * @param messageStream - Message stream ID (default: 'outbound')
   */
  constructor(
    apiToken?: string,
    messageStream: string = 'outbound'
  ) {
    this.apiToken = apiToken || process.env.POSTMARK_API_TOKEN || '';
    this.messageStream = messageStream;
    this.defaultFrom = process.env.MAIL_FROM_ADDRESS || 'noreply@example.com';
  }

  /**
   * Sends a single email.
   *
   * @param to - Recipient email address
   * @param subject - Email subject
   * @param htmlBody - HTML email body
   * @param textBody - Plain text body (optional)
   * @param options - Additional options
   * @returns Postmark API response
   */
  async send(
    to: string,
    subject: string,
    htmlBody: string,
    textBody?: string,
    options: EmailOptions = {}
  ): Promise<PostmarkResponse> {
    const payload: Record<string, unknown> = {
      From: options.from || this.defaultFrom,
      To: to,
      Subject: subject,
      HtmlBody: htmlBody,
      TextBody: textBody || this.stripHtml(htmlBody),
      MessageStream: this.messageStream,
    };

    if (options.replyTo) payload.ReplyTo = options.replyTo;
    if (options.tag) payload.Tag = options.tag;
    if (options.trackOpens !== undefined) payload.TrackOpens = options.trackOpens;
    if (options.trackLinks) payload.TrackLinks = options.trackLinks;
    if (options.metadata) payload.Metadata = options.metadata;
    if (options.attachments) payload.Attachments = options.attachments;

    return this.request<PostmarkResponse>('/email', payload);
  }

  /**
   * Sends an email using a Postmark template.
   *
   * @param to - Recipient email address
   * @param templateAlias - Template alias or ID
   * @param templateModel - Template variables
   * @param options - Additional options
   * @returns Postmark API response
   */
  async sendWithTemplate(
    to: string,
    templateAlias: string,
    templateModel: TemplateModel,
    options: EmailOptions = {}
  ): Promise<PostmarkResponse> {
    const payload: Record<string, unknown> = {
      From: options.from || this.defaultFrom,
      To: to,
      TemplateAlias: templateAlias,
      TemplateModel: templateModel,
      MessageStream: this.messageStream,
    };

    if (options.replyTo) payload.ReplyTo = options.replyTo;
    if (options.tag) payload.Tag = options.tag;
    if (options.metadata) payload.Metadata = options.metadata;

    return this.request<PostmarkResponse>('/email/withTemplate', payload);
  }

  /**
   * Sends batch emails (up to 500 per request).
   *
   * @param messages - Array of message payloads
   * @returns Array of API responses
   */
  async sendBatch(
    messages: Array<{
      to: string;
      subject: string;
      htmlBody: string;
      textBody?: string;
      from?: string;
      tag?: string;
    }>
  ): Promise<PostmarkResponse[]> {
    const results: PostmarkResponse[] = [];

    // Postmark limit is 500 messages per batch
    for (let i = 0; i < messages.length; i += 500) {
      const batch = messages.slice(i, i + 500).map((msg) => ({
        From: msg.from || this.defaultFrom,
        To: msg.to,
        Subject: msg.subject,
        HtmlBody: msg.htmlBody,
        TextBody: msg.textBody || this.stripHtml(msg.htmlBody),
        MessageStream: this.messageStream,
        Tag: msg.tag,
      }));

      const response = await this.request<PostmarkResponse[]>('/email/batch', batch);
      results.push(...response);
    }

    return results;
  }

  /**
   * Makes an API request to Postmark.
   */
  private async request<T>(endpoint: string, payload: unknown): Promise<T> {
    const response = await fetch(`${POSTMARK_API_URL}${endpoint}`, {
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json',
        'X-Postmark-Server-Token': this.apiToken,
      },
      body: JSON.stringify(payload),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.Message || 'Postmark API request failed');
    }

    return response.json();
  }

  /**
   * Strips HTML tags from a string.
   */
  private stripHtml(html: string): string {
    return html.replace(/<[^>]*>/g, '').trim();
  }
}

/**
 * Default Postmark client instance.
 */
export const postmarkClient = new PostmarkClient();
```

---

### Mailchimp Transactional API Client - Next.js

#### lib/email/mailchimp-client.ts

```typescript
/**
 * mailchimp-client.ts
 *
 * Mailchimp Transactional (Mandrill) API client for Next.js 16.x applications.
 * Provides type-safe email sending with template support.
 *
 * @package Next.js 16.x / React 19.x / TypeScript 5.9
 * @version 2.0.0
 * @see https://mailchimp.com/developer/transactional/api/
 */

const MAILCHIMP_API_URL = 'https://mandrillapp.com/api/1.0';

/**
 * Email recipient.
 */
interface Recipient {
  email: string;
  name?: string;
  type: 'to' | 'cc' | 'bcc';
}

/**
 * Email send options.
 */
interface EmailOptions {
  from?: string;
  fromName?: string;
  replyTo?: string;
  tags?: string[];
  trackOpens?: boolean;
  trackClicks?: boolean;
  metadata?: Record<string, string>;
  attachments?: Array<{
    name: string;
    content: string;
    type: string;
  }>;
  cc?: string[];
  bcc?: string[];
}

/**
 * Mailchimp API response.
 */
interface MailchimpResponse {
  _id: string;
  status: 'sent' | 'queued' | 'rejected' | 'invalid';
  email: string;
  reject_reason?: string;
}

/**
 * Template content type.
 */
type TemplateContent = Record<string, string>;

/**
 * Mailchimp Transactional API client class.
 */
export class MailchimpTransactionalClient {
  private apiKey: string;
  private defaultFrom: string;
  private defaultFromName: string;

  /**
   * Creates a new Mailchimp Transactional client instance.
   *
   * @param apiKey - Mandrill API key
   */
  constructor(apiKey?: string) {
    this.apiKey = apiKey || process.env.MAILCHIMP_TRANSACTIONAL_API_KEY || '';
    this.defaultFrom = process.env.MAIL_FROM_ADDRESS || 'noreply@example.com';
    this.defaultFromName = process.env.MAIL_FROM_NAME || 'Your App';
  }

  /**
   * Sends a single email.
   *
   * @param to - Recipient email address
   * @param subject - Email subject
   * @param htmlBody - HTML email body
   * @param textBody - Plain text body (optional)
   * @param options - Additional options
   * @returns Mailchimp API response
   */
  async send(
    to: string,
    subject: string,
    htmlBody: string,
    textBody?: string,
    options: EmailOptions = {}
  ): Promise<MailchimpResponse> {
    const recipients: Recipient[] = [{ email: to, type: 'to' }];

    if (options.cc) {
      recipients.push(...options.cc.map((email) => ({ email, type: 'cc' as const })));
    }

    if (options.bcc) {
      recipients.push(...options.bcc.map((email) => ({ email, type: 'bcc' as const })));
    }

    const message: Record<string, unknown> = {
      from_email: options.from || this.defaultFrom,
      from_name: options.fromName || this.defaultFromName,
      to: recipients,
      subject,
      html: htmlBody,
      text: textBody || this.stripHtml(htmlBody),
    };

    if (options.replyTo) {
      message.headers = { 'Reply-To': options.replyTo };
    }

    if (options.tags) message.tags = options.tags;
    if (options.trackOpens !== undefined) message.track_opens = options.trackOpens;
    if (options.trackClicks !== undefined) message.track_clicks = options.trackClicks;
    if (options.metadata) message.metadata = options.metadata;
    if (options.attachments) message.attachments = options.attachments;

    const response = await this.request<MailchimpResponse[]>('/messages/send.json', {
      message,
      async: false,
    });

    return response[0];
  }

  /**
   * Sends an email using a Mailchimp template.
   *
   * @param to - Recipient email address
   * @param templateName - Template name in Mailchimp
   * @param templateContent - Merge variables for template
   * @param options - Additional options
   * @returns Mailchimp API response
   */
  async sendWithTemplate(
    to: string,
    templateName: string,
    templateContent: TemplateContent,
    options: EmailOptions & { subject?: string } = {}
  ): Promise<MailchimpResponse> {
    const mergeVars = Object.entries(templateContent).map(([name, content]) => ({
      name,
      content,
    }));

    const message: Record<string, unknown> = {
      from_email: options.from || this.defaultFrom,
      from_name: options.fromName || this.defaultFromName,
      to: [{ email: to, type: 'to' }],
      subject: options.subject || '',
    };

    if (options.tags) message.tags = options.tags;
    if (options.metadata) message.metadata = options.metadata;

    const response = await this.request<MailchimpResponse[]>('/messages/send-template.json', {
      template_name: templateName,
      template_content: mergeVars,
      message,
      async: false,
    });

    return response[0];
  }

  /**
   * Lists available templates.
   *
   * @returns Array of template information
   */
  async listTemplates(): Promise<Array<{ slug: string; name: string; publish_code: string }>> {
    return this.request('/templates/list.json', {});
  }

  /**
   * Gets information about a sent message.
   *
   * @param messageId - Message ID
   * @returns Message information
   */
  async getMessageInfo(messageId: string): Promise<Record<string, unknown>> {
    return this.request('/messages/info.json', { id: messageId });
  }

  /**
   * Makes an API request to Mailchimp Transactional.
   */
  private async request<T>(endpoint: string, payload: Record<string, unknown>): Promise<T> {
    const response = await fetch(`${MAILCHIMP_API_URL}${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ ...payload, key: this.apiKey }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message || 'Mailchimp API request failed');
    }

    return response.json();
  }

  /**
   * Strips HTML tags from a string.
   */
  private stripHtml(html: string): string {
    return html.replace(/<[^>]*>/g, '').trim();
  }
}

/**
 * Default Mailchimp Transactional client instance.
 */
export const mailchimpClient = new MailchimpTransactionalClient();
```

---

### Email Service - Next.js

#### lib/email/email-service.ts

```typescript
/**
 * email-service.ts
 *
 * Unified email service with provider abstraction and failover support.
 *
 * @package Next.js 16.x / React 19.x / TypeScript 5.9
 * @version 2.0.0
 */

import { postmarkClient, PostmarkClient } from './postmark-client';
import { mailchimpClient, MailchimpTransactionalClient } from './mailchimp-client';

type EmailProvider = 'postmark' | 'mailchimp';

interface EmailServiceConfig {
  provider: EmailProvider;
  fallbackProvider?: EmailProvider;
}

/**
 * Unified email service with failover support.
 */
export class EmailService {
  private config: EmailServiceConfig;

  constructor(config?: Partial<EmailServiceConfig>) {
    this.config = {
      provider: config?.provider || (process.env.EMAIL_PROVIDER as EmailProvider) || 'postmark',
      fallbackProvider: config?.fallbackProvider,
    };
  }

  /**
   * Sends an email with automatic failover.
   */
  async send(
    to: string,
    subject: string,
    htmlBody: string,
    textBody?: string,
    options: Record<string, unknown> = {}
  ): Promise<{ success: boolean; messageId?: string; provider: string }> {
    try {
      const result = await this.sendWithProvider(
        this.config.provider,
        to,
        subject,
        htmlBody,
        textBody,
        options
      );

      return {
        success: true,
        messageId: result.messageId,
        provider: this.config.provider,
      };
    } catch (error) {
      console.error(`Primary provider ${this.config.provider} failed:`, error);

      if (this.config.fallbackProvider) {
        console.log(`Falling back to ${this.config.fallbackProvider}`);

        const result = await this.sendWithProvider(
          this.config.fallbackProvider,
          to,
          subject,
          htmlBody,
          textBody,
          options
        );

        return {
          success: true,
          messageId: result.messageId,
          provider: this.config.fallbackProvider,
        };
      }

      throw error;
    }
  }

  /**
   * Sends an email using a template with automatic failover.
   */
  async sendWithTemplate(
    to: string,
    templateName: string,
    templateData: Record<string, unknown>,
    options: Record<string, unknown> = {}
  ): Promise<{ success: boolean; messageId?: string; provider: string }> {
    try {
      const result = await this.sendTemplateWithProvider(
        this.config.provider,
        to,
        templateName,
        templateData,
        options
      );

      return {
        success: true,
        messageId: result.messageId,
        provider: this.config.provider,
      };
    } catch (error) {
      console.error(`Primary provider ${this.config.provider} failed for template:`, error);

      if (this.config.fallbackProvider) {
        const result = await this.sendTemplateWithProvider(
          this.config.fallbackProvider,
          to,
          templateName,
          templateData,
          options
        );

        return {
          success: true,
          messageId: result.messageId,
          provider: this.config.fallbackProvider,
        };
      }

      throw error;
    }
  }

  private async sendWithProvider(
    provider: EmailProvider,
    to: string,
    subject: string,
    htmlBody: string,
    textBody?: string,
    options: Record<string, unknown> = {}
  ): Promise<{ messageId: string }> {
    if (provider === 'postmark') {
      const result = await postmarkClient.send(to, subject, htmlBody, textBody, options);
      return { messageId: result.MessageID };
    } else {
      const result = await mailchimpClient.send(to, subject, htmlBody, textBody, options);
      return { messageId: result._id };
    }
  }

  private async sendTemplateWithProvider(
    provider: EmailProvider,
    to: string,
    templateName: string,
    templateData: Record<string, unknown>,
    options: Record<string, unknown> = {}
  ): Promise<{ messageId: string }> {
    if (provider === 'postmark') {
      const result = await postmarkClient.sendWithTemplate(to, templateName, templateData, options);
      return { messageId: result.MessageID };
    } else {
      const result = await mailchimpClient.sendWithTemplate(
        to,
        templateName,
        templateData as Record<string, string>,
        options
      );
      return { messageId: result._id };
    }
  }
}

/**
 * Default email service instance.
 */
export const emailService = new EmailService();
```

---

### API Routes - Next.js

#### app/api/email/send/route.ts

```typescript
/**
 * Email send API route.
 *
 * @package Next.js 16.x
 * @version 2.0.0
 */

import { NextRequest, NextResponse } from 'next/server';
import { emailService } from '@/lib/email/email-service';
import { auth } from '@/lib/auth';

export async function POST(request: NextRequest) {
  const session = await auth();

  if (!session) {
    return NextResponse.json({ error: 'Unauthorised' }, { status: 401 });
  }

  try {
    const body = await request.json();
    const { to, subject, htmlBody, textBody, template, templateData, options } = body;

    let result;

    if (template) {
      result = await emailService.sendWithTemplate(
        to,
        template,
        templateData || {},
        options || {}
      );
    } else {
      result = await emailService.send(
        to,
        subject,
        htmlBody,
        textBody,
        options || {}
      );
    }

    return NextResponse.json(result);
  } catch (error) {
    console.error('Email send error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Failed to send email' },
      { status: 500 }
    );
  }
}
```

#### app/api/webhooks/postmark/route.ts

```typescript
/**
 * Postmark webhook handler.
 *
 * @package Next.js 16.x
 * @version 2.0.0
 */

import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const payload = await request.json();
    const recordType = payload.RecordType;

    console.log(`Postmark webhook received: ${recordType}`, {
      messageId: payload.MessageID,
      recipient: payload.Recipient || payload.Email,
    });

    // Handle different event types
    switch (recordType) {
      case 'Delivery':
        // Email delivered successfully
        break;
      case 'Bounce':
        // Handle bounce - mark email as invalid
        console.warn('Email bounced:', payload.Email);
        break;
      case 'SpamComplaint':
        // Handle spam complaint - unsubscribe user
        console.warn('Spam complaint:', payload.Email);
        break;
      case 'Open':
        // Track email open
        break;
      case 'Click':
        // Track link click
        break;
    }

    return NextResponse.json({ received: true });
  } catch (error) {
    console.error('Postmark webhook error:', error);
    return NextResponse.json({ error: 'Webhook processing failed' }, { status: 500 });
  }
}
```

---

## React Native Stack

### Email Integration Note - React Native

React Native applications should not send emails directly from the mobile device. Instead, they should call backend APIs (GraphQL or REST) to trigger email sending from the server.

This approach ensures:
- API keys are not exposed in the mobile app
- Consistent email formatting and branding
- Proper logging and tracking
- Rate limiting and abuse prevention

---

### GraphQL Email Mutations - React Native

#### graphql/mutations/email.ts

```typescript
/**
 * GraphQL mutations for email operations.
 *
 * @package React Native 0.83.x / TypeScript 5.9
 * @version 2.0.0
 */

import { gql } from '@apollo/client';

/**
 * Mutation to send a transactional email.
 */
export const SEND_EMAIL = gql`
  mutation SendEmail($input: SendEmailInput!) {
    sendEmail(input: $input) {
      success
      messageId
      provider
      error
    }
  }
`;

/**
 * Mutation to send a template-based email.
 */
export const SEND_TEMPLATE_EMAIL = gql`
  mutation SendTemplateEmail($input: SendTemplateEmailInput!) {
    sendTemplateEmail(input: $input) {
      success
      messageId
      provider
      error
    }
  }
`;

/**
 * Input type for email sending.
 */
export interface SendEmailInput {
  to: string;
  subject: string;
  htmlBody: string;
  textBody?: string;
  replyTo?: string;
  tags?: string[];
}

/**
 * Input type for template email sending.
 */
export interface SendTemplateEmailInput {
  to: string;
  template: string;
  templateData: Record<string, string>;
  replyTo?: string;
  tags?: string[];
}

/**
 * Email send result type.
 */
export interface EmailResult {
  success: boolean;
  messageId?: string;
  provider?: string;
  error?: string;
}
```

#### hooks/useEmail.ts

```typescript
/**
 * useEmail.ts
 *
 * React Native hook for email operations via GraphQL API.
 *
 * @package React Native 0.83.x / TypeScript 5.9
 * @version 2.0.0
 */

import { useMutation } from '@apollo/client';
import {
  SEND_EMAIL,
  SEND_TEMPLATE_EMAIL,
  SendEmailInput,
  SendTemplateEmailInput,
  EmailResult,
} from '@/graphql/mutations/email';

interface UseEmailReturn {
  sendEmail: (input: SendEmailInput) => Promise<EmailResult>;
  sendTemplateEmail: (input: SendTemplateEmailInput) => Promise<EmailResult>;
  loading: boolean;
  error: string | null;
}

/**
 * Hook for sending emails via the backend API.
 *
 * @example
 * const { sendTemplateEmail, loading } = useEmail();
 *
 * const handleSendInvite = async () => {
 *   const result = await sendTemplateEmail({
 *     to: 'friend@example.com',
 *     template: 'invite-friend',
 *     templateData: {
 *       inviterName: user.name,
 *       inviteLink: 'https://app.example.com/invite/abc123',
 *     },
 *   });
 *
 *   if (result.success) {
 *     Alert.alert('Success', 'Invitation sent!');
 *   }
 * };
 */
export function useEmail(): UseEmailReturn {
  const [sendEmailMutation, { loading: sendLoading }] = useMutation(SEND_EMAIL);
  const [sendTemplateMutation, { loading: templateLoading }] = useMutation(SEND_TEMPLATE_EMAIL);

  const sendEmail = async (input: SendEmailInput): Promise<EmailResult> => {
    try {
      const { data } = await sendEmailMutation({ variables: { input } });
      return data?.sendEmail || { success: false, error: 'No response' };
    } catch (error) {
      return {
        success: false,
        error: error instanceof Error ? error.message : 'Failed to send email',
      };
    }
  };

  const sendTemplateEmail = async (input: SendTemplateEmailInput): Promise<EmailResult> => {
    try {
      const { data } = await sendTemplateMutation({ variables: { input } });
      return data?.sendTemplateEmail || { success: false, error: 'No response' };
    } catch (error) {
      return {
        success: false,
        error: error instanceof Error ? error.message : 'Failed to send email',
      };
    }
  };

  return {
    sendEmail,
    sendTemplateEmail,
    loading: sendLoading || templateLoading,
    error: null,
  };
}
```
