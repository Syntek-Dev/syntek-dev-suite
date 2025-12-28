# Report Services

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

---

## Table of Contents

- [Metadata](#metadata)
- [Framework Versions Tested](#framework-versions-tested)
- [Table of Contents](#table-of-contents)
- [Overview](#overview)
- [Laravel (TALL Stack)](#laravel-tall-stack)
  - [Report Generation Service](#report-generation-service)
  - [Report Filters DTO](#report-filters-dto)
  - [Sales Report Example](#sales-report-example)
- [Django/Wagtail](#djangowagtail)
  - [Report Service Base Class](#report-service-base-class)
  - [Report Filters Dataclass](#report-filters-dataclass)
  - [Sales Report Example](#sales-report-example-1)
- [Node.js/TypeScript (NestJS)](#nodejstypescript-nestjs)
  - [Report Filters DTO](#report-filters-dto-1)

---

## Overview

Base report data services provide common methods for:
- Date range filtering
- Pagination
- Currency formatting
- Percentage change calculations

---

## Laravel (TALL Stack)

### Report Generation Service

```php
<?php
/**
 * ReportDataService.php
 *
 * Base service with common methods for report data queries.
 * Provides reusable functionality for generating reports in Laravel applications.
 */

namespace App\Services\Reports;

use App\Services\Reports\DTOs\ReportFilters;
use App\Services\Reports\DTOs\ReportResult;
use Illuminate\Support\Facades\DB;
use Illuminate\Database\Query\Builder;

abstract class ReportDataService
{
    /**
     * Applies date range filter to a query builder.
     *
     * Filters the query by a specified date column within the given date range.
     * If only start date is provided, filters from that date onwards.
     * If only end date is provided, filters up to that date.
     *
     * @param Builder $query The query builder instance to filter
     * @param ReportFilters $filters Report filter parameters containing date range
     * @param string $dateColumn The date column to filter on (defaults to 'created_at')
     * @return Builder The filtered query builder instance
     */
    protected function applyDateRange(Builder $query, ReportFilters $filters, string $dateColumn = 'created_at'): Builder
    {
        if ($filters->startDate) {
            $query->where($dateColumn, '>=', $filters->startDate);
        }
        if ($filters->endDate) {
            $query->where($dateColumn, '<=', $filters->endDate);
        }
        return $query;
    }

    /**
     * Applies pagination to a query builder.
     *
     * Limits the number of results and sets the offset for pagination.
     * Useful for implementing paginated report views.
     *
     * @param Builder $query The query builder instance to paginate
     * @param ReportFilters $filters Report filter parameters containing limit and offset
     * @return Builder The paginated query builder instance
     */
    protected function applyPagination(Builder $query, ReportFilters $filters): Builder
    {
        if ($filters->limit) {
            $query->limit($filters->limit);
        }
        if ($filters->offset) {
            $query->offset($filters->offset);
        }
        return $query;
    }

    /**
     * Formats a number as GBP currency string.
     *
     * Formats the amount with pound sign, thousands separator, and two decimal places.
     * Example: 1234.56 becomes "£1,234.56"
     *
     * @param float $amount The amount to format
     * @return string The formatted currency string with £ symbol
     */
    protected function formatCurrency(float $amount): string
    {
        return '£' . number_format($amount, 2, '.', ',');
    }

    /**
     * Calculates percentage change between two values.
     *
     * Computes the percentage difference from previous to current value.
     * Handles edge case where previous value is zero to avoid division by zero.
     * Returns positive percentage for increases, negative for decreases.
     *
     * @param float $current The current value
     * @param float $previous The previous value to compare against
     * @return float The percentage change rounded to 2 decimal places
     */
    protected function calculatePercentageChange(float $current, float $previous): float
    {
        if ($previous == 0) {
            return $current > 0 ? 100.0 : 0.0;
        }

        return round((($current - $previous) / $previous) * 100, 2);
    }

    /**
     * Generates report data based on provided filters.
     *
     * Abstract method that must be implemented by concrete report services.
     * Each implementation should query the relevant data and return a ReportResult.
     *
     * @param ReportFilters $filters Filter parameters for the report
     * @return ReportResult The generated report data
     */
    abstract public function getData(ReportFilters $filters): ReportResult;
}
```

### Report Filters DTO

```php
<?php
/**
 * ReportFilters.php
 *
 * Data Transfer Object for report filter parameters.
 * Used to pass filtering criteria to report generation services.
 */

namespace App\Services\Reports\DTOs;

class ReportFilters
{
    /**
     * Creates a new ReportFilters instance.
     *
     * @param string|null $startDate Start date for filtering (format: Y-m-d)
     * @param string|null $endDate End date for filtering (format: Y-m-d)
     * @param int|null $limit Maximum number of results to return
     * @param int|null $offset Number of results to skip (for pagination)
     * @param string|null $groupBy Field to group results by (e.g., 'date', 'category')
     */
    public function __construct(
        public ?string $startDate = null,
        public ?string $endDate = null,
        public ?int $limit = null,
        public ?int $offset = null,
        public ?string $groupBy = null,
    ) {}
}
```

### Sales Report Example

```php
<?php
/**
 * SalesReportService.php
 *
 * Concrete implementation of ReportDataService for sales reporting.
 * Generates revenue, order count, and growth statistics.
 */

namespace App\Services\Reports;

use App\Models\Order;
use App\Services\Reports\DTOs\ReportFilters;
use App\Services\Reports\DTOs\ReportResult;
use Illuminate\Support\Facades\DB;

class SalesReportService extends ReportDataService
{
    /**
     * Generates sales report data for the specified date range.
     *
     * Retrieves total revenue, order count, average order value,
     * and calculates growth compared to the previous period.
     *
     * @param ReportFilters $filters Filter parameters for the report
     * @return ReportResult Sales report data with metrics and comparisons
     */
    public function getData(ReportFilters $filters): ReportResult
    {
        $query = Order::query()
            ->select([
                DB::raw('DATE(created_at) as date'),
                DB::raw('SUM(total_amount) as revenue'),
                DB::raw('COUNT(*) as order_count'),
                DB::raw('AVG(total_amount) as average_order_value'),
            ])
            ->where('status', 'completed');

        $query = $this->applyDateRange($query, $filters);

        if ($filters->groupBy === 'date') {
            $query->groupBy('date')->orderBy('date', 'desc');
        }

        $query = $this->applyPagination($query, $filters);

        $results = $query->get();

        // Calculate totals
        $totalRevenue = $results->sum('revenue');
        $totalOrders = $results->sum('order_count');

        // Get previous period data for comparison
        $previousResults = $this->getPreviousPeriodData($filters);
        $previousRevenue = $previousResults->sum('revenue');

        return new ReportResult(
            data: $results->map(function ($row) {
                return [
                    'date' => $row->date,
                    'revenue' => $this->formatCurrency($row->revenue),
                    'revenue_raw' => $row->revenue,
                    'order_count' => $row->order_count,
                    'average_order_value' => $this->formatCurrency($row->average_order_value),
                ];
            })->toArray(),
            metadata: [
                'total_revenue' => $this->formatCurrency($totalRevenue),
                'total_orders' => $totalOrders,
                'growth_percentage' => $this->calculatePercentageChange($totalRevenue, $previousRevenue),
            ]
        );
    }

    /**
     * Retrieves data from the previous period for comparison.
     *
     * @param ReportFilters $filters Original filter parameters
     * @return \Illuminate\Support\Collection Previous period results
     */
    private function getPreviousPeriodData(ReportFilters $filters)
    {
        // Implementation to fetch previous period data
        // (calculate date range and query)
        return collect([]);
    }
}
```

---

## Django/Wagtail

### Report Service Base Class

```python
"""
services/report_service.py

Base report data service with common query methods.
Provides reusable functionality for generating reports in Django/Wagtail applications.
"""

from abc import ABC, abstractmethod
from datetime import datetime, timedelta
from decimal import Decimal
from typing import Optional, Any

from django.db.models import QuerySet

from .dataclasses import ReportFilters, ReportResult


class ReportDataService(ABC):
    """
    Abstract base class for report data services.

    Provides common methods for filtering, pagination, and formatting
    that can be reused across different report types.
    """

    def apply_date_range(
        self,
        queryset: QuerySet,
        filters: ReportFilters,
        date_field: str = 'created_at',
    ) -> QuerySet:
        """
        Applies date range filter to queryset.

        Filters the queryset by a specified date field within the given date range.
        If only start date is provided, filters from that date onwards.
        If only end date is provided, filters up to that date.

        Args:
            queryset: The queryset to filter
            filters: Report filter parameters containing date range
            date_field: The date field to filter on (defaults to 'created_at')

        Returns:
            QuerySet: The filtered queryset
        """
        if filters.start_date:
            queryset = queryset.filter(**{f'{date_field}__gte': filters.start_date})
        if filters.end_date:
            queryset = queryset.filter(**{f'{date_field}__lte': filters.end_date})
        return queryset

    def apply_pagination(
        self,
        queryset: QuerySet,
        filters: ReportFilters,
    ) -> QuerySet:
        """
        Applies pagination to queryset.

        Limits the number of results and sets the offset for pagination.
        Useful for implementing paginated report views.

        Args:
            queryset: The queryset to paginate
            filters: Report filter parameters containing limit and offset

        Returns:
            QuerySet: The paginated queryset
        """
        if filters.offset:
            queryset = queryset[filters.offset:]
        if filters.limit:
            queryset = queryset[:filters.limit]
        return queryset

    def format_currency(self, amount: Decimal) -> str:
        """
        Formats a decimal amount as GBP currency string.

        Formats the amount with pound sign, thousands separator, and two decimal places.
        Example: Decimal('1234.56') becomes "£1,234.56"

        Args:
            amount: The amount to format

        Returns:
            str: The formatted currency string with £ symbol
        """
        return f'£{amount:,.2f}'

    def calculate_percentage_change(
        self,
        current: float,
        previous: float,
    ) -> float:
        """
        Calculates percentage change between two values.

        Computes the percentage difference from previous to current value.
        Handles edge case where previous value is zero to avoid division by zero.
        Returns positive percentage for increases, negative for decreases.

        Args:
            current: The current value
            previous: The previous value to compare against

        Returns:
            float: The percentage change rounded to 2 decimal places
        """
        if previous == 0:
            return 100.0 if current > 0 else 0.0
        return round(((current - previous) / previous) * 100, 2)

    @abstractmethod
    def get_data(self, filters: ReportFilters) -> ReportResult:
        """
        Generates report data based on filters.

        Abstract method that must be implemented by concrete report services.
        Each implementation should query the relevant data and return a ReportResult.

        Args:
            filters: Report filter parameters

        Returns:
            ReportResult: The report data
        """
        pass
```

### Report Filters Dataclass

```python
"""
dataclasses/report_filters.py

Filter parameters for report queries.
Data class for passing filtering criteria to report generation services.
"""

from dataclasses import dataclass
from datetime import date
from typing import Optional


@dataclass
class ReportFilters:
    """
    Filter parameters for report queries.

    Attributes:
        start_date: Start date for filtering (inclusive)
        end_date: End date for filtering (inclusive)
        limit: Maximum number of results to return
        offset: Number of results to skip (for pagination)
        group_by: Field to group results by (e.g., 'date', 'category')
    """
    start_date: Optional[date] = None
    end_date: Optional[date] = None
    limit: Optional[int] = None
    offset: Optional[int] = None
    group_by: Optional[str] = None
```

### Sales Report Example

```python
"""
services/sales_report_service.py

Concrete implementation of ReportDataService for sales reporting.
Generates revenue, order count, and growth statistics.
"""

from datetime import timedelta
from decimal import Decimal

from django.db.models import Sum, Count, Avg, F
from django.db.models.functions import TruncDate

from apps.orders.models import Order
from .report_service import ReportDataService
from .dataclasses import ReportFilters, ReportResult


class SalesReportService(ReportDataService):
    """
    Sales report service for generating revenue and order statistics.

    Provides methods to generate sales reports with revenue metrics,
    order counts, and period-over-period comparisons.
    """

    def get_data(self, filters: ReportFilters) -> ReportResult:
        """
        Generates sales report data for the specified date range.

        Retrieves total revenue, order count, average order value,
        and calculates growth compared to the previous period.

        Args:
            filters: Filter parameters for the report

        Returns:
            ReportResult: Sales report data with metrics and comparisons
        """
        queryset = Order.objects.filter(status='completed')
        queryset = self.apply_date_range(queryset, filters)

        if filters.group_by == 'date':
            queryset = queryset.annotate(
                date=TruncDate('created_at')
            ).values('date').annotate(
                revenue=Sum('total_amount'),
                order_count=Count('id'),
                average_order_value=Avg('total_amount'),
            ).order_by('-date')
        else:
            queryset = queryset.aggregate(
                revenue=Sum('total_amount'),
                order_count=Count('id'),
                average_order_value=Avg('total_amount'),
            )

        queryset = self.apply_pagination(queryset, filters)

        # Calculate totals
        if filters.group_by == 'date':
            results = list(queryset)
            total_revenue = sum(r['revenue'] for r in results if r['revenue'])
            total_orders = sum(r['order_count'] for r in results)
        else:
            results = [queryset]
            total_revenue = queryset.get('revenue', Decimal('0'))
            total_orders = queryset.get('order_count', 0)

        # Get previous period data for comparison
        previous_results = self._get_previous_period_data(filters)
        previous_revenue = previous_results.get('revenue', Decimal('0'))

        # Format data for output
        formatted_data = []
        for row in results:
            formatted_data.append({
                'date': row.get('date', None),
                'revenue': self.format_currency(row.get('revenue', Decimal('0'))),
                'revenue_raw': float(row.get('revenue', Decimal('0'))),
                'order_count': row.get('order_count', 0),
                'average_order_value': self.format_currency(
                    row.get('average_order_value', Decimal('0'))
                ),
            })

        return ReportResult(
            data=formatted_data,
            metadata={
                'total_revenue': self.format_currency(total_revenue),
                'total_orders': total_orders,
                'growth_percentage': self.calculate_percentage_change(
                    float(total_revenue),
                    float(previous_revenue)
                ),
            }
        )

    def _get_previous_period_data(self, filters: ReportFilters) -> dict:
        """
        Retrieves data from the previous period for comparison.

        Calculates the previous period date range based on the current filters
        and queries the same metrics for comparison.

        Args:
            filters: Original filter parameters

        Returns:
            dict: Previous period aggregated data
        """
        if not filters.start_date or not filters.end_date:
            return {'revenue': Decimal('0'), 'order_count': 0}

        # Calculate previous period date range
        period_length = (filters.end_date - filters.start_date).days
        previous_start = filters.start_date - timedelta(days=period_length + 1)
        previous_end = filters.end_date - timedelta(days=period_length + 1)

        # Query previous period data
        return Order.objects.filter(
            status='completed',
            created_at__gte=previous_start,
            created_at__lte=previous_end,
        ).aggregate(
            revenue=Sum('total_amount'),
            order_count=Count('id'),
        )
```

---

## Node.js/TypeScript (NestJS)

```typescript
/**
 * report.service.ts
 *
 * Base report data service with common query methods.
 */

import { Injectable } from '@nestjs/common';
import { ReportFilters } from '../dto/report-filters.dto';
import { ReportResult } from '../dto/report-result.dto';
import { SelectQueryBuilder } from 'typeorm';

@Injectable()
export abstract class ReportDataService {
  /**
   * Applies date range filter to query builder.
   *
   * @param queryBuilder - The query builder to filter
   * @param filters - Report filter parameters
   * @param dateColumn - The date column to filter on
   * @returns The filtered query builder
   */
  protected applyDateRange<T>(
    queryBuilder: SelectQueryBuilder<T>,
    filters: ReportFilters,
    dateColumn = 'created_at',
  ): SelectQueryBuilder<T> {
    if (filters.startDate) {
      queryBuilder.andWhere(`${dateColumn} >= :startDate`, {
        startDate: filters.startDate,
      });
    }
    if (filters.endDate) {
      queryBuilder.andWhere(`${dateColumn} <= :endDate`, {
        endDate: filters.endDate,
      });
    }
    return queryBuilder;
  }

  /**
   * Applies pagination to query builder.
   *
   * @param queryBuilder - The query builder to paginate
   * @param filters - Report filter parameters
   * @returns The paginated query builder
   */
  protected applyPagination<T>(
    queryBuilder: SelectQueryBuilder<T>,
    filters: ReportFilters,
  ): SelectQueryBuilder<T> {
    if (filters.limit) {
      queryBuilder.take(filters.limit);
    }
    if (filters.offset) {
      queryBuilder.skip(filters.offset);
    }
    return queryBuilder;
  }

  /**
   * Formats a number as currency string.
   *
   * @param amount - The amount to format
   * @returns The formatted currency string
   */
  protected formatCurrency(amount: number): string {
    return amount.toLocaleString('en-GB', {
      minimumFractionDigits: 2,
      maximumFractionDigits: 2,
    });
  }

  /**
   * Calculates percentage change between two values.
   *
   * @param current - The current value
   * @param previous - The previous value
   * @returns The percentage change
   */
  protected calculatePercentageChange(current: number, previous: number): number {
    if (previous === 0) {
      return current > 0 ? 100.0 : 0.0;
    }
    return Math.round(((current - previous) / previous) * 100 * 100) / 100;
  }

  /**
   * Generates report data based on filters.
   *
   * @param filters - Report filter parameters
   * @returns The report data
   */
  abstract getData(filters: ReportFilters): Promise<ReportResult>;
}
```

### Report Filters DTO

```typescript
/**
 * report-filters.dto.ts
 *
 * Filter parameters for report queries.
 */

import { IsOptional, IsDateString, IsInt, Min, IsString } from 'class-validator';

export class ReportFilters {
  @IsOptional()
  @IsDateString()
  startDate?: string;

  @IsOptional()
  @IsDateString()
  endDate?: string;

  @IsOptional()
  @IsInt()
  @Min(1)
  limit?: number;

  @IsOptional()
  @IsInt()
  @Min(0)
  offset?: number;

  @IsOptional()
  @IsString()
  groupBy?: string;
}
```