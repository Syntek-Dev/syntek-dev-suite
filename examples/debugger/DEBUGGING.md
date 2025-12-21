# Debugging Examples

## Overview

Comprehensive debugging configurations and techniques for all supported technology stacks.

## Technology Stacks

| Stack | Version | Primary Debug Tool |
|-------|---------|-------------------|
| TALL Stack | Laravel 12.x, PHP 8.4 | Xdebug 3.4, Telescope |
| Django/Wagtail | Django 6.x, Python 3.14 | Django Debug Toolbar |
| React/Next.js | Next.js 16.x, Node 24.x | Chrome DevTools |
| React Native | RN 0.83.x, TS 5.9 | Flipper, React DevTools |

---

## Table of Contents

- [Overview](#overview)
- [Technology Stacks](#technology-stacks)
- [TALL Stack (Laravel 12)](#tall-stack-laravel-12)
- [Django/Wagtail Stack](#djangowagtail-stack)
- [React/Next.js Stack](#reactnextjs-stack)
- [React Native Stack](#react-native-stack)

## TALL Stack (Laravel 12)

### Xdebug Configuration

```ini
; php.ini or docker-php-ext-xdebug.ini

[xdebug]
zend_extension=xdebug

xdebug.mode=debug,develop,coverage
xdebug.start_with_request=trigger
xdebug.client_host=host.docker.internal
xdebug.client_port=9003
xdebug.idekey=VSCODE
xdebug.log=/var/log/xdebug.log
xdebug.log_level=1

; Performance settings
xdebug.max_nesting_level=512
xdebug.var_display_max_depth=10
xdebug.var_display_max_data=512
xdebug.var_display_max_children=256
```

```json
// .vscode/launch.json

{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for Xdebug",
      "type": "php",
      "request": "launch",
      "port": 9003,
      "pathMappings": {
        "/var/www/html": "${workspaceFolder}"
      },
      "ignore": [
        "**/vendor/**/*.php"
      ],
      "xdebugSettings": {
        "max_children": 128,
        "max_data": 512,
        "max_depth": 5
      }
    },
    {
      "name": "Debug Artisan Command",
      "type": "php",
      "request": "launch",
      "program": "${workspaceFolder}/artisan",
      "args": ["${input:artisanCommand}"],
      "cwd": "${workspaceFolder}",
      "port": 9003,
      "runtimeArgs": [
        "-dxdebug.mode=debug",
        "-dxdebug.start_with_request=yes"
      ]
    },
    {
      "name": "Debug PHPUnit Test",
      "type": "php",
      "request": "launch",
      "program": "${workspaceFolder}/vendor/bin/pest",
      "args": ["--filter", "${input:testName}"],
      "cwd": "${workspaceFolder}",
      "port": 9003,
      "runtimeArgs": [
        "-dxdebug.mode=debug",
        "-dxdebug.start_with_request=yes"
      ]
    }
  ],
  "inputs": [
    {
      "id": "artisanCommand",
      "type": "promptString",
      "description": "Artisan command to run"
    },
    {
      "id": "testName",
      "type": "promptString",
      "description": "Test name or filter"
    }
  ]
}
```

### Laravel Telescope

```php
<?php
// config/telescope.php

return [
    'domain' => env('TELESCOPE_DOMAIN'),
    'path' => 'telescope',
    'driver' => env('TELESCOPE_DRIVER', 'database'),
    'storage' => [
        'database' => [
            'connection' => env('DB_CONNECTION', 'mysql'),
            'chunk' => 1000,
        ],
    ],
    'enabled' => env('TELESCOPE_ENABLED', true),
    'middleware' => [
        'web',
        Authorize::class,
    ],
    'only_paths' => [],
    'ignore_paths' => [
        'nova-api*',
        'pulse*',
    ],
    'ignore_commands' => [],
    'watchers' => [
        Watchers\BatchWatcher::class => env('TELESCOPE_BATCH_WATCHER', true),
        Watchers\CacheWatcher::class => [
            'enabled' => env('TELESCOPE_CACHE_WATCHER', true),
            'hidden' => [],
        ],
        Watchers\CommandWatcher::class => [
            'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
            'ignore' => [],
        ],
        Watchers\DumpWatcher::class => [
            'enabled' => env('TELESCOPE_DUMP_WATCHER', true),
            'always' => env('TELESCOPE_DUMP_WATCHER_ALWAYS', false),
        ],
        Watchers\EventWatcher::class => [
            'enabled' => env('TELESCOPE_EVENT_WATCHER', true),
            'ignore' => [],
        ],
        Watchers\ExceptionWatcher::class => env('TELESCOPE_EXCEPTION_WATCHER', true),
        Watchers\GateWatcher::class => [
            'enabled' => env('TELESCOPE_GATE_WATCHER', true),
            'ignore_abilities' => [],
            'ignore_packages' => true,
            'ignore_paths' => [],
        ],
        Watchers\JobWatcher::class => env('TELESCOPE_JOB_WATCHER', true),
        Watchers\LogWatcher::class => [
            'enabled' => env('TELESCOPE_LOG_WATCHER', true),
            'level' => 'error',
        ],
        Watchers\MailWatcher::class => env('TELESCOPE_MAIL_WATCHER', true),
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.*'],
            'hydrations' => true,
        ],
        Watchers\NotificationWatcher::class => env('TELESCOPE_NOTIFICATION_WATCHER', true),
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'ignore_packages' => true,
            'ignore_paths' => [],
            'slow' => 100,
        ],
        Watchers\RedisWatcher::class => env('TELESCOPE_REDIS_WATCHER', true),
        Watchers\RequestWatcher::class => [
            'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
            'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
            'ignore_http_methods' => [],
            'ignore_status_codes' => [],
        ],
        Watchers\ScheduleWatcher::class => env('TELESCOPE_SCHEDULE_WATCHER', true),
        Watchers\ViewWatcher::class => env('TELESCOPE_VIEW_WATCHER', true),
    ],
];
```

### Query Debugging

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Database\Events\QueryExecuted;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        if (config('app.debug') && config('app.query_log')) {
            $this->enableQueryLogging();
        }
    }

    private function enableQueryLogging(): void
    {
        DB::listen(function (QueryExecuted $query) {
            $sql = $query->sql;
            $bindings = $query->bindings;
            $time = $query->time;

            // Format SQL with bindings
            foreach ($bindings as $binding) {
                $value = is_numeric($binding) ? $binding : "'" . addslashes($binding) . "'";
                $sql = preg_replace('/\?/', $value, $sql, 1);
            }

            // Log slow queries
            if ($time > 100) {
                Log::warning('Slow query detected', [
                    'sql' => $sql,
                    'time_ms' => $time,
                    'connection' => $query->connectionName,
                ]);
            }

            // Debug output
            if (config('app.query_log_console')) {
                dump([
                    'sql' => $sql,
                    'time' => $time . 'ms',
                    'slow' => $time > 100,
                ]);
            }
        });
    }
}
```

```php
<?php
// app/Http/Middleware/DebugBarMiddleware.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Symfony\Component\HttpFoundation\Response;

class DebugBarMiddleware
{
    private array $queries = [];
    private float $startTime;

    public function handle(Request $request, Closure $next): Response
    {
        if (!config('app.debug')) {
            return $next($request);
        }

        $this->startTime = microtime(true);
        $this->queries = [];

        DB::listen(function ($query) {
            $this->queries[] = [
                'sql' => $query->sql,
                'bindings' => $query->bindings,
                'time' => $query->time,
            ];
        });

        $response = $next($request);

        $this->addDebugHeaders($response);

        return $response;
    }

    private function addDebugHeaders(Response $response): void
    {
        $totalTime = round((microtime(true) - $this->startTime) * 1000, 2);
        $queryTime = array_sum(array_column($this->queries, 'time'));
        $queryCount = count($this->queries);
        $memoryUsage = round(memory_get_peak_usage(true) / 1024 / 1024, 2);

        $response->headers->set('X-Debug-Time', $totalTime . 'ms');
        $response->headers->set('X-Debug-Queries', $queryCount);
        $response->headers->set('X-Debug-Query-Time', round($queryTime, 2) . 'ms');
        $response->headers->set('X-Debug-Memory', $memoryUsage . 'MB');
    }
}
```

---

## Django/Wagtail Stack

### Django Debug Toolbar

```python
# config/settings/development.py

INSTALLED_APPS += [
    'debug_toolbar',
]

MIDDLEWARE = [
    'debug_toolbar.middleware.DebugToolbarMiddleware',
] + MIDDLEWARE

INTERNAL_IPS = [
    '127.0.0.1',
    'localhost',
]

# Docker support
import socket
hostname, _, ips = socket.gethostbyname_ex(socket.gethostname())
INTERNAL_IPS += [ip[:-1] + '1' for ip in ips]

DEBUG_TOOLBAR_CONFIG = {
    'SHOW_TOOLBAR_CALLBACK': lambda request: DEBUG,
    'SHOW_COLLAPSED': True,
    'SQL_WARNING_THRESHOLD': 100,  # ms
    'PROFILER_MAX_DEPTH': 25,
    'RESULTS_CACHE_SIZE': 100,
}

DEBUG_TOOLBAR_PANELS = [
    'debug_toolbar.panels.history.HistoryPanel',
    'debug_toolbar.panels.versions.VersionsPanel',
    'debug_toolbar.panels.timer.TimerPanel',
    'debug_toolbar.panels.settings.SettingsPanel',
    'debug_toolbar.panels.headers.HeadersPanel',
    'debug_toolbar.panels.request.RequestPanel',
    'debug_toolbar.panels.sql.SQLPanel',
    'debug_toolbar.panels.staticfiles.StaticFilesPanel',
    'debug_toolbar.panels.templates.TemplatesPanel',
    'debug_toolbar.panels.cache.CachePanel',
    'debug_toolbar.panels.signals.SignalsPanel',
    'debug_toolbar.panels.redirects.RedirectsPanel',
    'debug_toolbar.panels.profiling.ProfilingPanel',
]
```

```python
# config/urls.py

from django.conf import settings
from django.urls import include, path

urlpatterns = [
    # ... your urls
]

if settings.DEBUG:
    import debug_toolbar
    urlpatterns = [
        path('__debug__/', include(debug_toolbar.urls)),
    ] + urlpatterns
```

### pdb and ipdb

```python
# apps/core/debug.py

import functools
import time
from typing import Any, Callable, TypeVar
import logging

logger = logging.getLogger(__name__)

F = TypeVar('F', bound=Callable[..., Any])


def debug_breakpoint() -> None:
    """Insert a breakpoint that works in any environment."""
    try:
        import ipdb
        ipdb.set_trace()
    except ImportError:
        import pdb
        pdb.set_trace()


def debug_on_exception(func: F) -> F:
    """Decorator to drop into debugger on exception."""
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        try:
            return func(*args, **kwargs)
        except Exception as e:
            logger.exception(f'Exception in {func.__name__}: {e}')
            debug_breakpoint()
            raise
    return wrapper  # type: ignore


def timing_decorator(func: F) -> F:
    """Decorator to measure and log function execution time."""
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        start_time = time.perf_counter()
        result = func(*args, **kwargs)
        end_time = time.perf_counter()
        elapsed = (end_time - start_time) * 1000

        logger.debug(f'{func.__name__} executed in {elapsed:.2f}ms')

        if elapsed > 1000:
            logger.warning(f'Slow function: {func.__name__} took {elapsed:.2f}ms')

        return result
    return wrapper  # type: ignore


class DebugContext:
    """Context manager for debugging code blocks."""

    def __init__(self, name: str = 'Debug Block'):
        self.name = name
        self.start_time: float = 0

    def __enter__(self) -> 'DebugContext':
        self.start_time = time.perf_counter()
        logger.debug(f'Entering {self.name}')
        return self

    def __exit__(self, exc_type: Any, exc_val: Any, exc_tb: Any) -> bool:
        elapsed = (time.perf_counter() - self.start_time) * 1000
        logger.debug(f'Exiting {self.name} after {elapsed:.2f}ms')

        if exc_type is not None:
            logger.exception(f'Exception in {self.name}: {exc_val}')
            if logger.isEnabledFor(logging.DEBUG):
                debug_breakpoint()

        return False
```

### Django Query Debugging

```python
# apps/core/middleware/query_debug.py

import time
import logging
from django.db import connection, reset_queries
from django.conf import settings
from django.http import HttpRequest, HttpResponse

logger = logging.getLogger('django.db.backends')


class QueryDebugMiddleware:
    """Middleware to log and analyse database queries."""

    def __init__(self, get_response: callable):
        self.get_response = get_response

    def __call__(self, request: HttpRequest) -> HttpResponse:
        if not settings.DEBUG:
            return self.get_response(request)

        reset_queries()
        start_time = time.perf_counter()

        response = self.get_response(request)

        total_time = (time.perf_counter() - start_time) * 1000
        queries = connection.queries

        # Log query statistics
        query_count = len(queries)
        query_time = sum(float(q.get('time', 0)) for q in queries) * 1000

        if query_count > 0:
            logger.debug(
                f'Request: {request.method} {request.path} | '
                f'Queries: {query_count} | '
                f'Query time: {query_time:.2f}ms | '
                f'Total time: {total_time:.2f}ms'
            )

        # Warn about N+1 queries
        self._detect_n_plus_one(queries)

        # Add debug headers
        response['X-Query-Count'] = str(query_count)
        response['X-Query-Time'] = f'{query_time:.2f}ms'
        response['X-Total-Time'] = f'{total_time:.2f}ms'

        return response

    def _detect_n_plus_one(self, queries: list[dict]) -> None:
        """Detect potential N+1 query patterns."""
        query_patterns: dict[str, int] = {}

        for query in queries:
            sql = query.get('sql', '')
            # Normalise query by removing specific IDs
            import re
            normalised = re.sub(r'\b\d+\b', 'N', sql)
            normalised = re.sub(r"'[^']*'", "'X'", normalised)

            query_patterns[normalised] = query_patterns.get(normalised, 0) + 1

        for pattern, count in query_patterns.items():
            if count > 5:
                logger.warning(
                    f'Potential N+1 query detected ({count} similar queries): '
                    f'{pattern[:100]}...'
                )
```

```python
# apps/core/debug/query_explainer.py

from django.db import connection
from django.db.models import QuerySet
from typing import Any
import logging

logger = logging.getLogger(__name__)


def explain_query(queryset: QuerySet) -> dict[str, Any]:
    """Get EXPLAIN output for a queryset."""
    sql, params = queryset.query.sql_with_params()

    with connection.cursor() as cursor:
        cursor.execute(f'EXPLAIN ANALYSE {sql}', params)
        explain_output = cursor.fetchall()

    return {
        'sql': sql,
        'params': params,
        'explain': explain_output,
    }


def log_slow_queries(queryset: QuerySet, threshold_ms: float = 100) -> QuerySet:
    """Log queryset execution if it exceeds threshold."""
    import time

    sql = str(queryset.query)
    start = time.perf_counter()

    # Force evaluation
    result = list(queryset)

    elapsed = (time.perf_counter() - start) * 1000

    if elapsed > threshold_ms:
        logger.warning(
            f'Slow query ({elapsed:.2f}ms): {sql[:200]}...'
        )

    return queryset.model.objects.filter(pk__in=[r.pk for r in result])
```

---

## React/Next.js Stack

### React DevTools

```typescript
// lib/debug/react-debug.ts

export function useRenderCount(componentName: string): void {
  const renderCount = React.useRef(0);
  renderCount.current += 1;

  if (process.env.NODE_ENV === 'development') {
    console.log(`[${componentName}] Render count: ${renderCount.current}`);
  }
}

export function useWhyDidYouUpdate<T extends Record<string, unknown>>(
  componentName: string,
  props: T
): void {
  const previousProps = React.useRef<T | undefined>(undefined);

  React.useEffect(() => {
    if (previousProps.current && process.env.NODE_ENV === 'development') {
      const changedProps: Partial<Record<keyof T, { from: unknown; to: unknown }>> = {};

      for (const key of Object.keys(props) as Array<keyof T>) {
        if (previousProps.current[key] !== props[key]) {
          changedProps[key] = {
            from: previousProps.current[key],
            to: props[key],
          };
        }
      }

      if (Object.keys(changedProps).length > 0) {
        console.log(`[${componentName}] Changed props:`, changedProps);
      }
    }

    previousProps.current = props;
  });
}

export function useDebugValue<T>(label: string, value: T): void {
  React.useDebugValue(`${label}: ${JSON.stringify(value)}`);
}
```

### Next.js Debugging

```json
// .vscode/launch.json

{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Next.js: debug server-side",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev",
      "cwd": "${workspaceFolder}",
      "serverReadyAction": {
        "pattern": "started server on .+, url: (https?://.+)",
        "uriFormat": "%s",
        "action": "debugWithChrome"
      }
    },
    {
      "name": "Next.js: debug client-side",
      "type": "chrome",
      "request": "launch",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}",
      "sourceMapPathOverrides": {
        "webpack://_N_E/*": "${webRoot}/*"
      }
    },
    {
      "name": "Next.js: debug full stack",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev",
      "cwd": "${workspaceFolder}",
      "serverReadyAction": {
        "pattern": "started server on .+, url: (https?://.+)",
        "uriFormat": "%s",
        "action": "startDebugging",
        "name": "Next.js: debug client-side"
      }
    }
  ]
}
```

```typescript
// lib/debug/api-debug.ts

import { NextRequest, NextResponse } from 'next/server';

interface DebugInfo {
  requestId: string;
  method: string;
  path: string;
  duration: number;
  statusCode: number;
  headers: Record<string, string>;
  body?: unknown;
}

export function withApiDebug<T extends (...args: unknown[]) => Promise<NextResponse>>(
  handler: T
): T {
  if (process.env.NODE_ENV !== 'development') {
    return handler;
  }

  return (async (...args: Parameters<T>): Promise<NextResponse> => {
    const request = args[0] as NextRequest;
    const startTime = performance.now();
    const requestId = crypto.randomUUID().slice(0, 8);

    console.log(`[API ${requestId}] ${request.method} ${request.nextUrl.pathname}`);

    try {
      const response = await handler(...args);
      const duration = Math.round(performance.now() - startTime);

      const debugInfo: DebugInfo = {
        requestId,
        method: request.method,
        path: request.nextUrl.pathname,
        duration,
        statusCode: response.status,
        headers: Object.fromEntries(request.headers.entries()),
      };

      console.log(`[API ${requestId}] Completed in ${duration}ms - Status: ${response.status}`);
      console.debug('[API Debug]', debugInfo);

      return response;
    } catch (error) {
      const duration = Math.round(performance.now() - startTime);
      console.error(`[API ${requestId}] Failed after ${duration}ms:`, error);
      throw error;
    }
  }) as T;
}
```

### VS Code Configuration

```json
// .vscode/settings.json

{
  "debug.javascript.autoAttachFilter": "smart",
  "debug.javascript.terminalOptions": {
    "skipFiles": [
      "<node_internals>/**",
      "**/node_modules/**"
    ]
  },
  "typescript.tsdk": "node_modules/typescript/lib",
  "editor.formatOnSave": true,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

---

## React Native Stack

### Flipper Setup

```javascript
// index.js or App.tsx

if (__DEV__) {
  require('./src/lib/debug/flipper-setup');
}
```

```typescript
// src/lib/debug/flipper-setup.ts

import { addPlugin } from 'react-native-flipper';

// Network debugging
addPlugin({
  getId: () => 'network',
  onConnect: (connection) => {
    console.log('[Flipper] Network plugin connected');
  },
  onDisconnect: () => {
    console.log('[Flipper] Network plugin disconnected');
  },
  runInBackground: () => true,
});

// Redux debugging (if using Redux)
export function setupReduxFlipper(store: any): void {
  if (__DEV__) {
    const createDebugger = require('redux-flipper').default;
    createDebugger(store);
  }
}

// Custom debug plugin
export function logToFlipper(category: string, message: string, data?: object): void {
  if (__DEV__) {
    console.log(`[${category}] ${message}`, data);
  }
}
```

### React Native Debugger

```typescript
// src/lib/debug/index.ts

import { InteractionManager } from 'react-native';

export function enableDebugging(): void {
  if (!__DEV__) return;

  // Enable network debugging
  global.XMLHttpRequest = global.originalXMLHttpRequest || global.XMLHttpRequest;

  // Enable console time tracking
  const originalLog = console.log;
  console.log = (...args: unknown[]) => {
    const timestamp = new Date().toISOString().slice(11, 23);
    originalLog(`[${timestamp}]`, ...args);
  };

  console.log('Debugging enabled');
}

export function usePerformanceMonitor(componentName: string): void {
  const mountTime = React.useRef<number>(0);
  const renderCount = React.useRef<number>(0);

  React.useEffect(() => {
    mountTime.current = performance.now();

    return () => {
      const lifetime = performance.now() - mountTime.current;
      console.log(
        `[Perf] ${componentName} unmounted after ${lifetime.toFixed(0)}ms, ` +
        `${renderCount.current} renders`
      );
    };
  }, [componentName]);

  React.useEffect(() => {
    renderCount.current += 1;
    const renderTime = performance.now();

    InteractionManager.runAfterInteractions(() => {
      const interactiveTime = performance.now() - renderTime;
      if (interactiveTime > 100) {
        console.warn(
          `[Perf] ${componentName} took ${interactiveTime.toFixed(0)}ms to become interactive`
        );
      }
    });
  });
}

export function measureAsync<T>(
  label: string,
  fn: () => Promise<T>
): Promise<T> {
  const start = performance.now();

  return fn().then(
    (result) => {
      const duration = performance.now() - start;
      console.log(`[Measure] ${label}: ${duration.toFixed(2)}ms`);
      return result;
    },
    (error) => {
      const duration = performance.now() - start;
      console.error(`[Measure] ${label} failed after ${duration.toFixed(2)}ms:`, error);
      throw error;
    }
  );
}
```

### Performance Debugging

```typescript
// src/lib/debug/performance.ts

import { PerformanceObserver, performance } from 'react-native-performance';

export function setupPerformanceMonitoring(): void {
  if (!__DEV__) return;

  const observer = new PerformanceObserver((list) => {
    list.getEntries().forEach((entry) => {
      if (entry.duration > 16.67) {
        // Longer than one frame at 60fps
        console.warn(
          `[Perf Warning] Slow operation: ${entry.name} took ${entry.duration.toFixed(2)}ms`
        );
      }
    });
  });

  observer.observe({ entryTypes: ['measure', 'mark'] });
}

export function markStart(label: string): void {
  if (__DEV__) {
    performance.mark(`${label}-start`);
  }
}

export function markEnd(label: string): void {
  if (__DEV__) {
    performance.mark(`${label}-end`);
    performance.measure(label, `${label}-start`, `${label}-end`);
  }
}

export function withPerformanceMark<T extends (...args: any[]) => any>(
  label: string,
  fn: T
): T {
  if (!__DEV__) return fn;

  return ((...args: Parameters<T>): ReturnType<T> => {
    markStart(label);
    const result = fn(...args);

    if (result instanceof Promise) {
      return result.finally(() => markEnd(label)) as ReturnType<T>;
    }

    markEnd(label);
    return result;
  }) as T;
}
```

```typescript
// src/lib/debug/memory.ts

import { NativeModules } from 'react-native';

interface MemoryInfo {
  usedJSHeapSize: number;
  totalJSHeapSize: number;
}

export async function getMemoryUsage(): Promise<MemoryInfo | null> {
  if (!__DEV__) return null;

  try {
    // This requires a native module - example implementation
    const { MemoryModule } = NativeModules;
    if (MemoryModule?.getMemoryInfo) {
      return await MemoryModule.getMemoryInfo();
    }
  } catch {
    // Fallback for when native module isn't available
  }

  return null;
}

export function useMemoryMonitor(intervalMs: number = 5000): void {
  React.useEffect(() => {
    if (!__DEV__) return;

    const interval = setInterval(async () => {
      const memory = await getMemoryUsage();
      if (memory) {
        const usedMB = (memory.usedJSHeapSize / 1024 / 1024).toFixed(2);
        const totalMB = (memory.totalJSHeapSize / 1024 / 1024).toFixed(2);
        console.log(`[Memory] Used: ${usedMB}MB / Total: ${totalMB}MB`);
      }
    }, intervalMs);

    return () => clearInterval(interval);
  }, [intervalMs]);
}

export function logComponentTree(component: React.ComponentType): void {
  if (!__DEV__) return;

  console.log(`[Component Tree] ${component.displayName || component.name}`);
  // Additional tree logging would require React DevTools integration
}
```
