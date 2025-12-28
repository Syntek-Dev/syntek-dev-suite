# Logging Examples

## Overview

Comprehensive logging examples for application monitoring, debugging, and audit trails across all supported technology stacks.

## Technology Stacks

| Stack          | Version                 | Logging Library       |
| -------------- | ----------------------- | --------------------- |
| TALL Stack     | Laravel 12.x, PHP 8.4   | Monolog 3.x           |
| Django/Wagtail | Django 6.x, Python 3.14 | structlog 24.x        |
| React/Next.js  | Next.js 16.x, Node 24.x | Pino 9.x              |
| React Native   | RN 0.83.x, TS 5.9       | react-native-logs 5.x |

---

## Table of Contents

- [Overview](#overview)
- [Technology Stacks](#technology-stacks)
- [Table of Contents](#table-of-contents)
- [TALL Stack (Laravel 12)](#tall-stack-laravel-12)
  - [Laravel Configuration](#laravel-configuration)
  - [Custom Log Channels](#custom-log-channels)
  - [Contextual Logging](#contextual-logging)
  - [Log Viewer](#log-viewer)
- [Django/Wagtail Stack](#djangowagtail-stack)
  - [Django Configuration](#django-configuration)
  - [Custom Handlers](#custom-handlers)
  - [Structured Logging](#structured-logging)
- [React/Next.js Stack](#reactnextjs-stack)
  - [Server-Side Logging](#server-side-logging)
  - [Client-Side Logging](#client-side-logging)
  - [API Route Logging](#api-route-logging)
- [React Native Stack](#react-native-stack)
  - [Development Logging](#development-logging)
  - [Production Logging](#production-logging)
  - [Crash Reporting](#crash-reporting)



## TALL Stack (Laravel 12)

### Laravel Configuration

```php
<?php
// config/logging.php

return [
    'default' => env('LOG_CHANNEL', 'stack'),

    'deprecations' => [
        'channel' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),
        'trace' => false,
    ],

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['daily', 'slack'],
            'ignore_exceptions' => false,
        ],

        'daily' => [
            'driver' => 'daily',
            'path' => storage_path('logs/laravel.log'),
            'level' => env('LOG_LEVEL', 'debug'),
            'days' => 14,
            'replace_placeholders' => true,
        ],

        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Laravel Log',
            'emoji' => ':boom:',
            'level' => env('LOG_SLACK_LEVEL', 'critical'),
        ],

        'papertrail' => [
            'driver' => 'monolog',
            'level' => env('LOG_LEVEL', 'debug'),
            'handler' => \Monolog\Handler\SyslogUdpHandler::class,
            'handler_with' => [
                'host' => env('PAPERTRAIL_URL'),
                'port' => env('PAPERTRAIL_PORT'),
                'connectionString' => 'tls://'.env('PAPERTRAIL_URL').':'.env('PAPERTRAIL_PORT'),
            ],
        ],

        'audit' => [
            'driver' => 'daily',
            'path' => storage_path('logs/audit.log'),
            'level' => 'info',
            'days' => 90,
        ],

        'security' => [
            'driver' => 'daily',
            'path' => storage_path('logs/security.log'),
            'level' => 'warning',
            'days' => 365,
        ],
    ],
];
```

### Custom Log Channels

```php
<?php
// app/Logging/CustomFormatter.php

namespace App\Logging;

use Monolog\Formatter\JsonFormatter;
use Monolog\LogRecord;

class CustomFormatter extends JsonFormatter
{
    public function format(LogRecord $record): string
    {
        $data = [
            'timestamp' => $record->datetime->format('Y-m-d\TH:i:s.uP'),
            'level' => $record->level->getName(),
            'message' => $record->message,
            'context' => $record->context,
            'extra' => array_merge($record->extra, [
                'environment' => app()->environment(),
                'app_version' => config('app.version'),
            ]),
        ];

        return $this->toJson($data) . "\n";
    }
}
```

```php
<?php
// app/Logging/CreateCustomLogger.php

namespace App\Logging;

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Processor\WebProcessor;
use Monolog\Processor\MemoryUsageProcessor;
use Monolog\Processor\IntrospectionProcessor;

class CreateCustomLogger
{
    public function __invoke(array $config): Logger
    {
        $logger = new Logger('custom');

        $handler = new StreamHandler(
            $config['path'] ?? storage_path('logs/custom.log'),
            $config['level'] ?? Logger::DEBUG
        );

        $handler->setFormatter(new CustomFormatter());

        $logger->pushHandler($handler);
        $logger->pushProcessor(new WebProcessor());
        $logger->pushProcessor(new MemoryUsageProcessor());
        $logger->pushProcessor(new IntrospectionProcessor());

        return $logger;
    }
}
```

### Contextual Logging

```php
<?php
// app/Services/LoggingService.php

namespace App\Services;

use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Auth;

class LoggingService
{
    public function logUserAction(string $action, array $data = []): void
    {
        Log::channel('audit')->info($action, [
            'user_id' => Auth::id(),
            'user_email' => Auth::user()?->email,
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'url' => request()->fullUrl(),
            'method' => request()->method(),
            'data' => $this->maskSensitiveData($data),
        ]);
    }

    public function logSecurityEvent(string $event, array $context = []): void
    {
        Log::channel('security')->warning($event, [
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'timestamp' => now()->toIso8601String(),
            ...$context,
        ]);
    }

    public function logException(\Throwable $exception, array $context = []): void
    {
        Log::error($exception->getMessage(), [
            'exception' => get_class($exception),
            'file' => $exception->getFile(),
            'line' => $exception->getLine(),
            'trace' => $exception->getTraceAsString(),
            'user_id' => Auth::id(),
            ...$context,
        ]);
    }

    private function maskSensitiveData(array $data): array
    {
        $sensitiveKeys = ['password', 'token', 'secret', 'credit_card', 'cvv'];

        return collect($data)->map(function ($value, $key) use ($sensitiveKeys) {
            if (in_array(strtolower($key), $sensitiveKeys)) {
                return '***REDACTED***';
            }
            return $value;
        })->toArray();
    }
}
```

### Log Viewer

```php
<?php
// app/Livewire/LogViewer.php

namespace App\Livewire;

use Livewire\Component;
use Livewire\WithPagination;
use Illuminate\Support\Facades\File;

class LogViewer extends Component
{
    use WithPagination;

    public string $selectedChannel = 'laravel';
    public string $levelFilter = 'all';
    public string $searchTerm = '';
    public int $perPage = 50;

    public function render()
    {
        $logs = $this->parseLogs();

        return view('livewire.log-viewer', [
            'logs' => $logs,
            'channels' => $this->getAvailableChannels(),
        ]);
    }

    private function parseLogs(): array
    {
        $logFile = storage_path("logs/{$this->selectedChannel}.log");

        if (!File::exists($logFile)) {
            return [];
        }

        $content = File::get($logFile);
        $lines = explode("\n", $content);

        return collect($lines)
            ->filter(fn($line) => !empty(trim($line)))
            ->map(fn($line) => $this->parseLogLine($line))
            ->filter(fn($log) => $this->matchesFilters($log))
            ->reverse()
            ->take($this->perPage)
            ->values()
            ->toArray();
    }

    private function parseLogLine(string $line): ?array
    {
        $pattern = '/\[(\d{4}-\d{2}-\d{2}\s\d{2}:\d{2}:\d{2})\]\s(\w+)\.(\w+):\s(.+)/';

        if (preg_match($pattern, $line, $matches)) {
            return [
                'timestamp' => $matches[1],
                'environment' => $matches[2],
                'level' => strtolower($matches[3]),
                'message' => $matches[4],
            ];
        }

        return null;
    }

    private function matchesFilters(?array $log): bool
    {
        if (!$log) return false;

        if ($this->levelFilter !== 'all' && $log['level'] !== $this->levelFilter) {
            return false;
        }

        if ($this->searchTerm && !str_contains(strtolower($log['message']), strtolower($this->searchTerm))) {
            return false;
        }

        return true;
    }

    private function getAvailableChannels(): array
    {
        return collect(File::files(storage_path('logs')))
            ->map(fn($file) => pathinfo($file, PATHINFO_FILENAME))
            ->filter(fn($name) => !str_starts_with($name, '.'))
            ->values()
            ->toArray();
    }
}
```

---

## Django/Wagtail Stack

### Django Configuration

```python
# config/settings/base.py

import structlog

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'json': {
            '()': structlog.stdlib.ProcessorFormatter,
            'processor': structlog.processors.JSONRenderer(),
        },
        'console': {
            '()': structlog.stdlib.ProcessorFormatter,
            'processor': structlog.dev.ConsoleRenderer(),
        },
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'formatter': 'console',
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': BASE_DIR / 'logs' / 'application.log',
            'maxBytes': 10485760,  # 10MB
            'backupCount': 10,
            'formatter': 'json',
        },
        'audit': {
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': BASE_DIR / 'logs' / 'audit.log',
            'maxBytes': 52428800,  # 50MB
            'backupCount': 20,
            'formatter': 'json',
        },
        'security': {
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': BASE_DIR / 'logs' / 'security.log',
            'maxBytes': 52428800,  # 50MB
            'backupCount': 50,
            'formatter': 'json',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'level': 'INFO',
            'propagate': True,
        },
        'django.security': {
            'handlers': ['security'],
            'level': 'WARNING',
            'propagate': False,
        },
        'audit': {
            'handlers': ['audit'],
            'level': 'INFO',
            'propagate': False,
        },
        'app': {
            'handlers': ['console', 'file'],
            'level': 'DEBUG',
            'propagate': False,
        },
    },
}

# Structlog configuration
structlog.configure(
    processors=[
        structlog.contextvars.merge_contextvars,
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.stdlib.PositionalArgumentsFormatter(),
        structlog.processors.TimeStamper(fmt='iso'),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.UnicodeDecoder(),
        structlog.stdlib.ProcessorFormatter.wrap_for_formatter,
    ],
    logger_factory=structlog.stdlib.LoggerFactory(),
    wrapper_class=structlog.stdlib.BoundLogger,
    cache_logger_on_first_use=True,
)
```

### Custom Handlers

```python
# apps/core/logging/handlers.py

import logging
import json
from datetime import datetime
from typing import Any
import httpx


class SlackHandler(logging.Handler):
    """Send critical logs to Slack."""

    def __init__(self, webhook_url: str, channel: str = '#alerts'):
        super().__init__()
        self.webhook_url = webhook_url
        self.channel = channel

    def emit(self, record: logging.LogRecord) -> None:
        try:
            log_entry = self.format(record)
            colour = self._get_colour(record.levelno)

            payload = {
                'channel': self.channel,
                'attachments': [{
                    'color': colour,
                    'title': f'{record.levelname}: {record.name}',
                    'text': log_entry,
                    'ts': datetime.now().timestamp(),
                    'fields': [
                        {'title': 'Environment', 'value': record.__dict__.get('environment', 'unknown'), 'short': True},
                        {'title': 'Module', 'value': record.module, 'short': True},
                    ],
                }],
            }

            httpx.post(self.webhook_url, json=payload, timeout=5.0)
        except Exception:
            self.handleError(record)

    def _get_colour(self, level: int) -> str:
        colours = {
            logging.DEBUG: '#808080',
            logging.INFO: '#36a64f',
            logging.WARNING: '#ff9800',
            logging.ERROR: '#f44336',
            logging.CRITICAL: '#9c27b0',
        }
        return colours.get(level, '#808080')


class DatabaseHandler(logging.Handler):
    """Store logs in database for querying."""

    def emit(self, record: logging.LogRecord) -> None:
        from apps.core.models import LogEntry

        try:
            LogEntry.objects.create(
                level=record.levelname,
                logger_name=record.name,
                message=self.format(record),
                module=record.module,
                func_name=record.funcName,
                line_no=record.lineno,
                exception_info=record.exc_text if record.exc_info else None,
                extra_data=getattr(record, 'extra', {}),
            )
        except Exception:
            self.handleError(record)
```

### Structured Logging

```python
# apps/core/logging/service.py

import structlog
from functools import wraps
from typing import Any, Callable
from django.http import HttpRequest


logger = structlog.get_logger('app')
audit_logger = structlog.get_logger('audit')
security_logger = structlog.get_logger('django.security')


class LoggingService:
    """Centralised logging service with structured output."""

    @staticmethod
    def bind_request_context(request: HttpRequest) -> None:
        """Bind request context to all subsequent log calls."""
        structlog.contextvars.clear_contextvars()
        structlog.contextvars.bind_contextvars(
            request_id=getattr(request, 'id', None),
            user_id=request.user.id if request.user.is_authenticated else None,
            ip_address=LoggingService._get_client_ip(request),
            user_agent=request.META.get('HTTP_USER_AGENT', ''),
            path=request.path,
            method=request.method,
        )

    @staticmethod
    def log_user_action(action: str, **kwargs: Any) -> None:
        """Log user actions for audit trail."""
        audit_logger.info(
            action,
            **LoggingService._mask_sensitive_data(kwargs),
        )

    @staticmethod
    def log_security_event(event: str, severity: str = 'warning', **kwargs: Any) -> None:
        """Log security-related events."""
        log_method = getattr(security_logger, severity, security_logger.warning)
        log_method(event, **kwargs)

    @staticmethod
    def log_exception(exc: Exception, **kwargs: Any) -> None:
        """Log exception with full context."""
        logger.exception(
            str(exc),
            exception_type=type(exc).__name__,
            **kwargs,
        )

    @staticmethod
    def _get_client_ip(request: HttpRequest) -> str:
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            return x_forwarded_for.split(',')[0].strip()
        return request.META.get('REMOTE_ADDR', '')

    @staticmethod
    def _mask_sensitive_data(data: dict[str, Any]) -> dict[str, Any]:
        sensitive_keys = {'password', 'token', 'secret', 'credit_card', 'cvv', 'api_key'}
        return {
            k: '***REDACTED***' if k.lower() in sensitive_keys else v
            for k, v in data.items()
        }


def log_function_call(logger_name: str = 'app'):
    """Decorator to log function entry and exit."""
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            log = structlog.get_logger(logger_name)
            log.debug(f'Entering {func.__name__}', args_count=len(args), kwargs_keys=list(kwargs.keys()))

            try:
                result = func(*args, **kwargs)
                log.debug(f'Exiting {func.__name__}', success=True)
                return result
            except Exception as e:
                log.exception(f'Error in {func.__name__}', exception_type=type(e).__name__)
                raise

        return wrapper
    return decorator
```

```python
# apps/core/middleware/logging.py

import time
import uuid
import structlog
from django.http import HttpRequest, HttpResponse


class RequestLoggingMiddleware:
    """Log all HTTP requests with timing information."""

    def __init__(self, get_response):
        self.get_response = get_response
        self.logger = structlog.get_logger('django.request')

    def __call__(self, request: HttpRequest) -> HttpResponse:
        request.id = str(uuid.uuid4())
        start_time = time.perf_counter()

        # Bind context for this request
        structlog.contextvars.clear_contextvars()
        structlog.contextvars.bind_contextvars(
            request_id=request.id,
            method=request.method,
            path=request.path,
        )

        response = self.get_response(request)

        duration_ms = (time.perf_counter() - start_time) * 1000

        self.logger.info(
            'request_completed',
            status_code=response.status_code,
            duration_ms=round(duration_ms, 2),
            content_length=len(response.content) if hasattr(response, 'content') else 0,
        )

        response['X-Request-ID'] = request.id
        return response
```

---

## React/Next.js Stack

### Server-Side Logging

```typescript
// lib/logger/index.ts

import pino from 'pino';

const isProduction = process.env.NODE_ENV === 'production';

export const logger = pino({
  level: process.env.LOG_LEVEL || (isProduction ? 'info' : 'debug'),
  transport: isProduction
    ? undefined
    : {
        target: 'pino-pretty',
        options: {
          colorize: true,
          translateTime: 'SYS:standard',
          ignore: 'pid,hostname',
        },
      },
  base: {
    env: process.env.NODE_ENV,
    version: process.env.npm_package_version,
  },
  redact: {
    paths: ['password', 'token', 'secret', 'creditCard', 'cvv', 'apiKey'],
    censor: '***REDACTED***',
  },
  formatters: {
    level: (label: string) => ({ level: label }),
  },
});

export const auditLogger = logger.child({ logger: 'audit' });
export const securityLogger = logger.child({ logger: 'security' });

export type Logger = typeof logger;
```

```typescript
// lib/logger/request-logger.ts

import { NextRequest, NextResponse } from 'next/server';
import { logger } from './index';
import { v4 as uuidv4 } from 'uuid';

export interface RequestContext {
  requestId: string;
  method: string;
  path: string;
  userAgent: string;
  ip: string;
  userId?: string;
}

export function createRequestLogger(request: NextRequest): {
  logger: typeof logger;
  context: RequestContext;
} {
  const requestId = request.headers.get('x-request-id') || uuidv4();
  const context: RequestContext = {
    requestId,
    method: request.method,
    path: request.nextUrl.pathname,
    userAgent: request.headers.get('user-agent') || '',
    ip: request.headers.get('x-forwarded-for')?.split(',')[0] || 'unknown',
  };

  const childLogger = logger.child(context);

  return { logger: childLogger, context };
}

export function withRequestLogging(
  handler: (
    request: NextRequest,
    context: { params: Promise<Record<string, string>> }
  ) => Promise<NextResponse>
) {
  return async (
    request: NextRequest,
    context: { params: Promise<Record<string, string>> }
  ): Promise<NextResponse> => {
    const startTime = performance.now();
    const { logger: reqLogger, context: reqContext } = createRequestLogger(request);

    reqLogger.info('Request started');

    try {
      const response = await handler(request, context);
      const duration = Math.round(performance.now() - startTime);

      reqLogger.info({
        msg: 'Request completed',
        statusCode: response.status,
        durationMs: duration,
      });

      response.headers.set('x-request-id', reqContext.requestId);
      return response;
    } catch (error) {
      const duration = Math.round(performance.now() - startTime);

      reqLogger.error({
        msg: 'Request failed',
        error: error instanceof Error ? error.message : 'Unknown error',
        stack: error instanceof Error ? error.stack : undefined,
        durationMs: duration,
      });

      throw error;
    }
  };
}
```

### Client-Side Logging

```typescript
// lib/logger/client.ts

'use client';

type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface LogEntry {
  level: LogLevel;
  message: string;
  timestamp: string;
  context?: Record<string, unknown>;
  error?: {
    name: string;
    message: string;
    stack?: string;
  };
}

class ClientLogger {
  private queue: LogEntry[] = [];
  private flushInterval: NodeJS.Timeout | null = null;
  private readonly maxQueueSize = 50;
  private readonly flushIntervalMs = 5000;

  constructor() {
    if (typeof window !== 'undefined') {
      this.startFlushInterval();
      window.addEventListener('beforeunload', () => this.flush());
      window.addEventListener('error', (event) => {
        this.error('Uncaught error', {
          message: event.message,
          filename: event.filename,
          lineno: event.lineno,
          colno: event.colno,
        });
      });
    }
  }

  private log(level: LogLevel, message: string, context?: Record<string, unknown>): void {
    const entry: LogEntry = {
      level,
      message,
      timestamp: new Date().toISOString(),
      context: {
        url: typeof window !== 'undefined' ? window.location.href : undefined,
        userAgent: typeof navigator !== 'undefined' ? navigator.userAgent : undefined,
        ...context,
      },
    };

    if (process.env.NODE_ENV === 'development') {
      const consoleMethod = level === 'debug' ? 'log' : level;
      console[consoleMethod](`[${level.toUpperCase()}]`, message, context);
    }

    this.queue.push(entry);

    if (this.queue.length >= this.maxQueueSize) {
      this.flush();
    }
  }

  debug(message: string, context?: Record<string, unknown>): void {
    this.log('debug', message, context);
  }

  info(message: string, context?: Record<string, unknown>): void {
    this.log('info', message, context);
  }

  warn(message: string, context?: Record<string, unknown>): void {
    this.log('warn', message, context);
  }

  error(message: string, context?: Record<string, unknown>, error?: Error): void {
    const entry: LogEntry = {
      level: 'error',
      message,
      timestamp: new Date().toISOString(),
      context,
      error: error
        ? {
            name: error.name,
            message: error.message,
            stack: error.stack,
          }
        : undefined,
    };

    if (process.env.NODE_ENV === 'development') {
      console.error(`[ERROR]`, message, context, error);
    }

    this.queue.push(entry);
    this.flush(); // Flush immediately for errors
  }

  private startFlushInterval(): void {
    this.flushInterval = setInterval(() => this.flush(), this.flushIntervalMs);
  }

  async flush(): Promise<void> {
    if (this.queue.length === 0) return;

    const entries = [...this.queue];
    this.queue = [];

    try {
      await fetch('/api/logs', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ entries }),
      });
    } catch {
      // Re-queue failed entries
      this.queue.unshift(...entries);
    }
  }
}

export const clientLogger = new ClientLogger();
```

### API Route Logging

```typescript
// app/api/logs/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { logger } from '@/lib/logger';
import { z } from 'zod';

const LogEntrySchema = z.object({
  level: z.enum(['debug', 'info', 'warn', 'error']),
  message: z.string(),
  timestamp: z.string(),
  context: z.record(z.unknown()).optional(),
  error: z
    .object({
      name: z.string(),
      message: z.string(),
      stack: z.string().optional(),
    })
    .optional(),
});

const LogsPayloadSchema = z.object({
  entries: z.array(LogEntrySchema),
});

export async function POST(request: NextRequest): Promise<NextResponse> {
  try {
    const body = await request.json();
    const { entries } = LogsPayloadSchema.parse(body);

    const clientIp = request.headers.get('x-forwarded-for')?.split(',')[0] || 'unknown';

    for (const entry of entries) {
      const childLogger = logger.child({
        source: 'client',
        clientIp,
        ...entry.context,
      });

      switch (entry.level) {
        case 'debug':
          childLogger.debug(entry.message);
          break;
        case 'info':
          childLogger.info(entry.message);
          break;
        case 'warn':
          childLogger.warn(entry.message);
          break;
        case 'error':
          childLogger.error({
            msg: entry.message,
            err: entry.error,
          });
          break;
      }
    }

    return NextResponse.json({ success: true });
  } catch (error) {
    logger.error({ msg: 'Failed to process client logs', error });
    return NextResponse.json({ error: 'Invalid log format' }, { status: 400 });
  }
}
```

---

## React Native Stack

### Development Logging

```typescript
// src/lib/logger/index.ts

import { logger as rnLogger, consoleTransport, fileAsyncTransport } from 'react-native-logs';
import RNFS from 'react-native-fs';

const isDev = __DEV__;

const config = {
  levels: {
    debug: 0,
    info: 1,
    warn: 2,
    error: 3,
  },
  severity: isDev ? 'debug' : 'info',
  transport: isDev ? consoleTransport : fileAsyncTransport,
  transportOptions: {
    FS: RNFS,
    fileName: `app-{date}.log`,
    filePath: RNFS.DocumentDirectoryPath + '/logs',
  },
  async: true,
  dateFormat: 'iso',
  printLevel: true,
  printDate: true,
  fixedExtLvlLength: false,
  enabled: true,
};

export const logger = rnLogger.createLogger<'debug' | 'info' | 'warn' | 'error'>(config);

// Create child loggers for different modules
export const apiLogger = logger.extend('api');
export const authLogger = logger.extend('auth');
export const navigationLogger = logger.extend('navigation');
export const storageLogger = logger.extend('storage');
```

```typescript
// src/lib/logger/context.tsx

import React, { createContext, useContext, useMemo, ReactNode } from 'react';
import { logger as baseLogger } from './index';

interface LogContext {
  userId?: string;
  sessionId?: string;
  screenName?: string;
}

interface LoggerContextValue {
  logger: typeof baseLogger;
  setContext: (context: Partial<LogContext>) => void;
}

const LoggerContext = createContext<LoggerContextValue | null>(null);

export function LoggerProvider({ children }: { children: ReactNode }) {
  const [context, setContextState] = React.useState<LogContext>({});

  const value = useMemo(() => {
    const contextualLogger = {
      debug: (message: string, ...args: unknown[]) => {
        baseLogger.debug(`[${context.screenName || 'app'}] ${message}`, { ...context, ...args[0] });
      },
      info: (message: string, ...args: unknown[]) => {
        baseLogger.info(`[${context.screenName || 'app'}] ${message}`, { ...context, ...args[0] });
      },
      warn: (message: string, ...args: unknown[]) => {
        baseLogger.warn(`[${context.screenName || 'app'}] ${message}`, { ...context, ...args[0] });
      },
      error: (message: string, ...args: unknown[]) => {
        baseLogger.error(`[${context.screenName || 'app'}] ${message}`, { ...context, ...args[0] });
      },
    };

    return {
      logger: contextualLogger as typeof baseLogger,
      setContext: (newContext: Partial<LogContext>) => {
        setContextState((prev) => ({ ...prev, ...newContext }));
      },
    };
  }, [context]);

  return <LoggerContext.Provider value={value}>{children}</LoggerContext.Provider>;
}

export function useLogger() {
  const context = useContext(LoggerContext);
  if (!context) {
    throw new Error('useLogger must be used within LoggerProvider');
  }
  return context;
}
```

### Production Logging

```typescript
// src/lib/logger/remote-transport.ts

import { transportFunctionType } from 'react-native-logs';
import NetInfo from '@react-native-community/netinfo';

interface LogEntry {
  level: string;
  message: string;
  timestamp: string;
  context?: Record<string, unknown>;
}

const logQueue: LogEntry[] = [];
const MAX_QUEUE_SIZE = 100;
const FLUSH_INTERVAL = 30000; // 30 seconds

export const remoteTransport: transportFunctionType = (props) => {
  const { msg, level, extension, options } = props;

  const entry: LogEntry = {
    level: level.text,
    message: typeof msg === 'string' ? msg : JSON.stringify(msg),
    timestamp: new Date().toISOString(),
    context: {
      extension,
      ...options?.context,
    },
  };

  logQueue.push(entry);

  if (logQueue.length >= MAX_QUEUE_SIZE) {
    flushLogs(options?.endpoint);
  }
};

async function flushLogs(endpoint?: string): Promise<void> {
  if (logQueue.length === 0 || !endpoint) return;

  const networkState = await NetInfo.fetch();
  if (!networkState.isConnected) return;

  const entries = [...logQueue];
  logQueue.length = 0;

  try {
    await fetch(endpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ entries }),
    });
  } catch {
    // Re-queue on failure
    logQueue.unshift(...entries.slice(0, MAX_QUEUE_SIZE - logQueue.length));
  }
}

// Start periodic flush
setInterval(() => {
  flushLogs(process.env.EXPO_PUBLIC_LOG_ENDPOINT);
}, FLUSH_INTERVAL);
```

### Crash Reporting

```typescript
// src/lib/logger/crash-reporter.ts

import * as Sentry from '@sentry/react-native';
import { logger } from './index';

interface UserContext {
  id: string;
  email?: string;
  username?: string;
}

export function initialiseCrashReporting(): void {
  if (__DEV__) {
    logger.info('Crash reporting disabled in development');
    return;
  }

  Sentry.init({
    dsn: process.env.EXPO_PUBLIC_SENTRY_DSN,
    environment: process.env.EXPO_PUBLIC_APP_ENV || 'production',
    tracesSampleRate: 0.2,
    enableAutoSessionTracking: true,
    attachStacktrace: true,
    beforeSend(event) {
      // Scrub sensitive data
      if (event.request?.headers) {
        delete event.request.headers['Authorization'];
        delete event.request.headers['Cookie'];
      }
      return event;
    },
  });

  logger.info('Crash reporting initialised');
}

export function setUserContext(user: UserContext | null): void {
  if (user) {
    Sentry.setUser({
      id: user.id,
      email: user.email,
      username: user.username,
    });
  } else {
    Sentry.setUser(null);
  }
}

export function captureException(error: Error, context?: Record<string, unknown>): void {
  logger.error('Exception captured', { error: error.message, ...context });

  if (!__DEV__) {
    Sentry.captureException(error, {
      extra: context,
    });
  }
}

export function captureMessage(message: string, level: Sentry.SeverityLevel = 'info'): void {
  logger.info('Message captured', { message, level });

  if (!__DEV__) {
    Sentry.captureMessage(message, level);
  }
}

export function addBreadcrumb(
  category: string,
  message: string,
  data?: Record<string, unknown>
): void {
  Sentry.addBreadcrumb({
    category,
    message,
    data,
    level: 'info',
  });
}
```

```typescript
// src/components/ErrorBoundary.tsx

import React, { Component, ErrorInfo, ReactNode } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { captureException, addBreadcrumb } from '@/lib/logger/crash-reporter';
import { logger } from '@/lib/logger';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    logger.error('React error boundary caught error', {
      error: error.message,
      componentStack: errorInfo.componentStack,
    });

    addBreadcrumb('error-boundary', 'Component error caught', {
      componentStack: errorInfo.componentStack,
    });

    captureException(error, {
      componentStack: errorInfo.componentStack,
    });
  }

  handleRetry = (): void => {
    this.setState({ hasError: false, error: undefined });
  };

  render(): ReactNode {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <View style={styles.container}>
          <Text style={styles.title}>Something went wrong</Text>
          <Text style={styles.message}>
            We've been notified and are working on a fix.
          </Text>
          <TouchableOpacity style={styles.button} onPress={this.handleRetry}>
            <Text style={styles.buttonText}>Try Again</Text>
          </TouchableOpacity>
        </View>
      );
    }

    return this.props.children;
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333',
  },
  message: {
    fontSize: 16,
    textAlign: 'center',
    marginBottom: 20,
    color: '#666',
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 8,
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600',
  },
});
```
