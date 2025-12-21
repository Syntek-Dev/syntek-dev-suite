# CSV Formatter

## Metadata

| Property | Value |
|----------|-------|
| **Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
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

---

## Table of Contents

- [Metadata](#metadata)
- [Framework Versions Tested](#framework-versions-tested)
- [Overview](#overview)
- [Laravel (TALL Stack)](#laravel-tall-stack)
- [Django/Wagtail](#djangowagtail)
- [Node.js/TypeScript](#nodejstypescript)

---

## Overview

CSV formatters export data with:
- Configurable delimiters and quote characters
- Streaming support for large datasets
- Proper escaping of special characters

---

## Laravel (TALL Stack)

### CSV Export Service

```php
<?php
/**
 * CsvExportService.php
 *
 * CSV export service with streaming support for large datasets.
 * Compatible with Laravel 12.x and PHP 8.4.
 */

namespace App\Services\Export;

use Illuminate\Support\Collection;
use Illuminate\Database\Eloquent\Builder;
use Symfony\Component\HttpFoundation\StreamedResponse;

class CsvExportService
{
    /**
     * Initialises the CSV export service with formatting options.
     *
     * @param string $delimiter Field separator character
     * @param string $enclosure Quote character for fields containing special characters
     * @param string $escape Escape character for special characters
     */
    public function __construct(
        private string $delimiter = ',',
        private string $enclosure = '"',
        private string $escape = '\\'
    ) {}

    /**
     * Formats data as a CSV string.
     *
     * Converts a collection or array of data into a complete CSV string.
     * Automatically detects headers from the first row if not provided.
     *
     * @param Collection|array $data Data to export
     * @param array|null $headers Optional column headers
     * @return string The formatted CSV content
     */
    public function format(Collection|array $data, ?array $headers = null): string
    {
        $data = collect($data);
        $output = fopen('php://temp', 'r+');

        // Write headers
        if ($headers) {
            fputcsv($output, $headers, $this->delimiter, $this->enclosure, $this->escape);
        } elseif ($data->isNotEmpty()) {
            $firstRow = $data->first();
            $headerKeys = is_array($firstRow) ? array_keys($firstRow) : array_keys((array) $firstRow);
            fputcsv($output, $headerKeys, $this->delimiter, $this->enclosure, $this->escape);
        }

        // Write data rows
        foreach ($data as $row) {
            fputcsv($output, array_values((array) $row), $this->delimiter, $this->enclosure, $this->escape);
        }

        rewind($output);
        $content = stream_get_contents($output);
        fclose($output);

        return $content;
    }

    /**
     * Streams data as CSV for large datasets.
     *
     * Uses a generator to yield CSV rows one at a time, preventing memory
     * exhaustion when exporting large datasets.
     *
     * @param iterable $data Data iterator
     * @param array|null $headers Optional column headers
     * @return \Generator CSV rows
     */
    public function stream(iterable $data, ?array $headers = null): \Generator
    {
        if ($headers) {
            yield $this->formatRow($headers);
        }

        $firstRow = true;
        foreach ($data as $row) {
            if ($firstRow && !$headers) {
                $headerKeys = is_array($row) ? array_keys($row) : array_keys((array) $row);
                yield $this->formatRow($headerKeys);
                $firstRow = false;
            }
            yield $this->formatRow(array_values((array) $row));
        }
    }

    /**
     * Creates a streamed HTTP response for CSV download.
     *
     * Generates a streaming response that outputs CSV data directly to the
     * client, ideal for large exports that shouldn't be loaded into memory.
     *
     * @param Builder $query Eloquent query builder
     * @param string $filename Download filename
     * @param array|null $headers Optional column headers
     * @return StreamedResponse
     */
    public function streamResponse(Builder $query, string $filename, ?array $headers = null): StreamedResponse
    {
        $callback = function () use ($query, $headers) {
            $output = fopen('php://output', 'w');

            // Write headers
            if ($headers) {
                fputcsv($output, $headers, $this->delimiter, $this->enclosure, $this->escape);
            }

            // Stream query results in chunks
            $query->chunk(1000, function ($records) use ($output, &$headers) {
                foreach ($records as $record) {
                    // Auto-detect headers from first record
                    if (!$headers) {
                        $headers = array_keys($record->toArray());
                        fputcsv($output, $headers, $this->delimiter, $this->enclosure, $this->escape);
                    }

                    fputcsv($output, array_values($record->toArray()), $this->delimiter, $this->enclosure, $this->escape);
                }
            });

            fclose($output);
        };

        return response()->stream($callback, 200, [
            'Content-Type' => 'text/csv',
            'Content-Disposition' => "attachment; filename=\"{$filename}\"",
            'Cache-Control' => 'no-cache, no-store, must-revalidate',
            'Pragma' => 'no-cache',
            'Expires' => '0',
        ]);
    }

    /**
     * Formats a single row as CSV.
     *
     * @param array $row Row data to format
     * @return string The formatted CSV row with newline
     */
    private function formatRow(array $row): string
    {
        $output = fopen('php://temp', 'r+');
        fputcsv($output, $row, $this->delimiter, $this->enclosure, $this->escape);
        rewind($output);
        $line = stream_get_contents($output);
        fclose($output);
        return $line;
    }
}
```

### Controller Example

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use App\Services\Export\CsvExportService;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\StreamedResponse;

class ExportController extends Controller
{
    /**
     * Initialises the controller with the CSV export service.
     */
    public function __construct(
        private CsvExportService $csvExporter
    ) {}

    /**
     * Exports users as a streaming CSV download.
     *
     * Streams all users from the database as a CSV file, preventing
     * memory issues with large datasets.
     *
     * @return StreamedResponse
     */
    public function exportUsers(): StreamedResponse
    {
        $query = User::query()
            ->select('id', 'name', 'email', 'created_at')
            ->orderBy('created_at', 'desc');

        return $this->csvExporter->streamResponse(
            $query,
            'users-export-' . now()->format('Y-m-d') . '.csv',
            ['ID', 'Name', 'Email', 'Created At']
        );
    }

    /**
     * Exports filtered users based on request parameters.
     *
     * @param Request $request
     * @return StreamedResponse
     */
    public function exportFilteredUsers(Request $request): StreamedResponse
    {
        $query = User::query()
            ->select('id', 'name', 'email', 'created_at');

        // Apply filters
        if ($request->has('search')) {
            $query->where('name', 'like', '%' . $request->search . '%');
        }

        if ($request->has('from_date')) {
            $query->where('created_at', '>=', $request->from_date);
        }

        return $this->csvExporter->streamResponse(
            $query,
            'users-filtered-' . now()->format('Y-m-d') . '.csv',
            ['ID', 'Name', 'Email', 'Registration Date']
        );
    }
}
```

---

## Django/Wagtail

### CSV Formatter Service

```python
"""
services/export/csv_export_service.py

CSV export service with streaming support for large datasets.
Compatible with Django 6.x and Python 3.14.
"""

import csv
import io
from typing import Iterator, List, Dict, Any, Optional, Union
from django.db.models import QuerySet


class CsvExportService:
    """
    Formats data as CSV with configurable delimiters and streaming support.

    This service provides methods for both in-memory CSV generation and
    memory-efficient streaming for large datasets.
    """

    def __init__(
        self,
        delimiter: str = ',',
        quotechar: str = '"',
        quoting: int = csv.QUOTE_MINIMAL,
    ):
        """
        Initialises the CSV export service with formatting options.

        Args:
            delimiter: Field separator character
            quotechar: Quote character for fields containing special characters
            quoting: Quote behaviour (csv.QUOTE_MINIMAL, csv.QUOTE_ALL, etc.)
        """
        self.delimiter = delimiter
        self.quotechar = quotechar
        self.quoting = quoting

    def format(
        self,
        data: Union[List[Dict[str, Any]], QuerySet],
        headers: Optional[List[str]] = None,
    ) -> str:
        """
        Formats data as a complete CSV string.

        Converts a list of dictionaries or Django QuerySet into a CSV string.
        Automatically detects headers from the first row if not provided.

        Args:
            data: List of dictionaries or QuerySet to export
            headers: Optional list of column headers

        Returns:
            str: The formatted CSV content
        """
        output = io.StringIO()
        writer = csv.writer(
            output,
            delimiter=self.delimiter,
            quotechar=self.quotechar,
            quoting=self.quoting,
        )

        # Convert QuerySet to list if needed
        if hasattr(data, 'values'):
            data = list(data.values())

        # Write headers
        if headers:
            writer.writerow(headers)
        elif data:
            writer.writerow(data[0].keys())

        # Write data rows
        for row in data:
            writer.writerow(row.values())

        return output.getvalue()

    def stream(
        self,
        data: Union[Iterator[Dict[str, Any]], QuerySet],
        headers: Optional[List[str]] = None,
        chunk_size: int = 1000,
    ) -> Iterator[str]:
        """
        Streams data as CSV rows for large datasets.

        Uses an iterator to yield CSV rows one at a time, preventing memory
        exhaustion when exporting large datasets from the database.

        Args:
            data: Iterator of dictionaries or QuerySet to export
            headers: Optional list of column headers
            chunk_size: Number of records to fetch at once (for QuerySets)

        Yields:
            str: Each formatted CSV row
        """
        # Handle QuerySet with chunking
        if hasattr(data, 'iterator'):
            data = data.iterator(chunk_size=chunk_size)

        if headers:
            yield self._format_row(headers)

        first_row = True
        for row in data:
            # Convert model instance to dict if needed
            if hasattr(row, '__dict__'):
                row = {k: v for k, v in row.__dict__.items() if not k.startswith('_')}

            if first_row and not headers:
                yield self._format_row(list(row.keys()))
                first_row = False
            yield self._format_row(list(row.values()))

    def stream_queryset(
        self,
        queryset: QuerySet,
        fields: List[str],
        headers: Optional[List[str]] = None,
        chunk_size: int = 1000,
    ) -> Iterator[str]:
        """
        Streams a Django QuerySet as CSV rows with specific fields.

        Efficiently streams large QuerySets by using .iterator() and .only()
        to minimise memory usage.

        Args:
            queryset: Django QuerySet to export
            fields: List of field names to include
            headers: Optional list of column headers
            chunk_size: Number of records to fetch at once

        Yields:
            str: Each formatted CSV row
        """
        # Write headers
        if headers:
            yield self._format_row(headers)
        else:
            yield self._format_row(fields)

        # Stream data in chunks
        for record in queryset.only(*fields).iterator(chunk_size=chunk_size):
            row_data = [getattr(record, field) for field in fields]
            yield self._format_row(row_data)

    def _format_row(self, row: List[Any]) -> str:
        """
        Formats a single row as CSV.

        Args:
            row: List of values to format

        Returns:
            str: The formatted CSV row with newline
        """
        output = io.StringIO()
        writer = csv.writer(
            output,
            delimiter=self.delimiter,
            quotechar=self.quotechar,
            quoting=self.quoting,
        )
        writer.writerow(row)
        return output.getvalue()
```

### Django Streaming Response

```python
"""
views/export_views.py

Django views for CSV export with streaming responses.
"""

from django.http import StreamingHttpResponse
from django.contrib.auth.decorators import login_required
from django.utils import timezone
from django.contrib.auth import get_user_model

from services.export.csv_export_service import CsvExportService

User = get_user_model()


@login_required
def export_users_csv(request):
    """
    Exports all users as a streaming CSV download.

    Streams user data directly to the client to prevent memory issues
    with large datasets. Requires user authentication.

    Args:
        request: Django HTTP request

    Returns:
        StreamingHttpResponse: CSV file download
    """
    formatter = CsvExportService()

    def generate():
        """Generator function for streaming CSV rows."""
        users = User.objects.all().values('id', 'username', 'email', 'date_joined')
        yield from formatter.stream(
            users,
            headers=['ID', 'Username', 'Email', 'Registration Date']
        )

    timestamp = timezone.now().strftime('%Y-%m-%d')
    filename = f'users-export-{timestamp}.csv'

    response = StreamingHttpResponse(generate(), content_type='text/csv')
    response['Content-Disposition'] = f'attachment; filename="{filename}"'
    response['Cache-Control'] = 'no-cache, no-store, must-revalidate'
    response['Pragma'] = 'no-cache'
    response['Expires'] = '0'

    return response


@login_required
def export_filtered_users_csv(request):
    """
    Exports filtered users based on query parameters.

    Supports filtering by search term and date range. Uses the more
    efficient stream_queryset method for better performance.

    Args:
        request: Django HTTP request with optional query parameters:
            - search: Username or email search term
            - from_date: Start date filter (YYYY-MM-DD)
            - to_date: End date filter (YYYY-MM-DD)

    Returns:
        StreamingHttpResponse: Filtered CSV file download
    """
    formatter = CsvExportService()

    # Build filtered query
    queryset = User.objects.all()

    if search := request.GET.get('search'):
        queryset = queryset.filter(
            username__icontains=search
        ) | queryset.filter(
            email__icontains=search
        )

    if from_date := request.GET.get('from_date'):
        queryset = queryset.filter(date_joined__gte=from_date)

    if to_date := request.GET.get('to_date'):
        queryset = queryset.filter(date_joined__lte=to_date)

    queryset = queryset.order_by('-date_joined')

    def generate():
        """Generator function for streaming filtered CSV rows."""
        yield from formatter.stream_queryset(
            queryset,
            fields=['id', 'username', 'email', 'date_joined'],
            headers=['ID', 'Username', 'Email', 'Registration Date'],
            chunk_size=1000
        )

    timestamp = timezone.now().strftime('%Y-%m-%d')
    filename = f'users-filtered-{timestamp}.csv'

    response = StreamingHttpResponse(generate(), content_type='text/csv')
    response['Content-Disposition'] = f'attachment; filename="{filename}"'
    response['Cache-Control'] = 'no-cache, no-store, must-revalidate'
    response['Pragma'] = 'no-cache'
    response['Expires'] = '0'

    return response
```

---

## Node.js/TypeScript

```typescript
/**
 * csv.formatter.ts
 *
 * CSV export formatter with streaming support for large datasets.
 */

import { Transform } from 'stream';

export interface CsvFormatterOptions {
  delimiter?: string;
  enclosure?: string;
  escape?: string;
}

export class CsvFormatter {
  private delimiter: string;
  private enclosure: string;
  private escape: string;

  constructor(options: CsvFormatterOptions = {}) {
    this.delimiter = options.delimiter ?? ',';
    this.enclosure = options.enclosure ?? '"';
    this.escape = options.escape ?? '\\';
  }

  /**
   * Formats data as a CSV string.
   *
   * @param data - Array of objects to export
   * @param headers - Optional array of column headers
   * @returns The formatted CSV content
   */
  format(data: Record<string, unknown>[], headers?: string[]): string {
    const lines: string[] = [];

    // Write headers
    if (headers) {
      lines.push(this.formatRow(headers));
    } else if (data.length > 0) {
      lines.push(this.formatRow(Object.keys(data[0])));
    }

    // Write data rows
    for (const row of data) {
      lines.push(this.formatRow(Object.values(row)));
    }

    return lines.join('');
  }

  /**
   * Creates a streaming transform for large datasets.
   *
   * @param headers - Optional array of column headers
   * @returns A Transform stream that outputs CSV rows
   */
  createStream(headers?: string[]): Transform {
    let headersSent = false;
    const self = this;

    return new Transform({
      objectMode: true,
      transform(chunk: Record<string, unknown>, encoding, callback) {
        try {
          if (!headersSent) {
            const headerRow = headers ?? Object.keys(chunk);
            this.push(self.formatRow(headerRow));
            headersSent = true;
          }
          this.push(self.formatRow(Object.values(chunk)));
          callback();
        } catch (error) {
          callback(error as Error);
        }
      },
    });
  }

  /**
   * Formats a single row as CSV.
   *
   * @param values - Array of values to format
   * @returns The formatted CSV row with newline
   */
  private formatRow(values: unknown[]): string {
    const escaped = values.map((value) => {
      const stringValue = String(value ?? '');
      if (
        stringValue.includes(this.delimiter) ||
        stringValue.includes(this.enclosure) ||
        stringValue.includes('\n')
      ) {
        const escapedValue = stringValue.replace(
          new RegExp(this.enclosure, 'g'),
          this.escape + this.enclosure,
        );
        return `${this.enclosure}${escapedValue}${this.enclosure}`;
      }
      return stringValue;
    });
    return escaped.join(this.delimiter) + '\n';
  }
}
```

### NestJS Streaming Response

```typescript
import { Controller, Get, Res } from '@nestjs/common';
import { Response } from 'express';
import { CsvFormatter } from './formatters/csv.formatter';

@Controller('export')
export class ExportController {
  @Get('users')
  async exportUsers(@Res() res: Response) {
    const formatter = new CsvFormatter();

    res.setHeader('Content-Type', 'text/csv');
    res.setHeader('Content-Disposition', 'attachment; filename="users.csv"');

    const csvStream = formatter.createStream(['ID', 'Username', 'Created']);
    csvStream.pipe(res);

    // Stream users from database
    const users = await this.userService.streamAll();
    for await (const user of users) {
      csvStream.write(user);
    }
    csvStream.end();
  }
}
```