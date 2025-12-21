# Data Analysis Examples

## Overview

Data analysis, visualisation, and reporting examples for all technology stacks.

## Table of Contents

- [TALL Stack (Laravel 12)](#tall-stack-laravel-12)
- [Django/Wagtail Stack](#djangowagtail-stack)
- [React/Next.js Stack](#reactnextjs-stack)
- [React Native Stack](#react-native-stack)

---

## TALL Stack (Laravel 12)

### Analytics Service

```php
<?php
// app/Services/AnalyticsService.php

namespace App\Services;

use App\Models\Order;
use Carbon\Carbon;
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;

final readonly class AnalyticsService
{
    public function getSalesOverview(Carbon $from, Carbon $to): array
    {
        return [
            'total_revenue' => $this->getTotalRevenue($from, $to),
            'order_count' => $this->getOrderCount($from, $to),
            'average_order_value' => $this->getAverageOrderValue($from, $to),
            'daily_sales' => $this->getDailySales($from, $to),
            'top_products' => $this->getTopProducts($from, $to, 10),
        ];
    }

    public function getDailySales(Carbon $from, Carbon $to): Collection
    {
        return Order::query()
            ->whereBetween('created_at', [$from, $to])
            ->where('status', 'completed')
            ->select(
                DB::raw('DATE(created_at) as date'),
                DB::raw('SUM(total) as revenue'),
                DB::raw('COUNT(*) as orders')
            )
            ->groupBy('date')
            ->orderBy('date')
            ->get();
    }

    public function getTopProducts(Carbon $from, Carbon $to, int $limit = 10): Collection
    {
        return DB::table('order_items')
            ->join('orders', 'orders.id', '=', 'order_items.order_id')
            ->join('products', 'products.id', '=', 'order_items.product_id')
            ->whereBetween('orders.created_at', [$from, $to])
            ->where('orders.status', 'completed')
            ->select(
                'products.id',
                'products.name',
                DB::raw('SUM(order_items.quantity) as units_sold'),
                DB::raw('SUM(order_items.total) as revenue')
            )
            ->groupBy('products.id', 'products.name')
            ->orderByDesc('revenue')
            ->limit($limit)
            ->get();
    }

    private function getTotalRevenue(Carbon $from, Carbon $to): int
    {
        return Order::query()
            ->whereBetween('created_at', [$from, $to])
            ->where('status', 'completed')
            ->sum('total');
    }

    private function getOrderCount(Carbon $from, Carbon $to): int
    {
        return Order::query()
            ->whereBetween('created_at', [$from, $to])
            ->where('status', 'completed')
            ->count();
    }

    private function getAverageOrderValue(Carbon $from, Carbon $to): float
    {
        return Order::query()
            ->whereBetween('created_at', [$from, $to])
            ->where('status', 'completed')
            ->avg('total') ?? 0;
    }
}
```

### Dashboard Component (Livewire)

```php
<?php
// app/Livewire/AnalyticsDashboard.php

namespace App\Livewire;

use App\Services\AnalyticsService;
use Carbon\Carbon;
use Livewire\Component;

class AnalyticsDashboard extends Component
{
    public string $period = '30';
    public array $salesData = [];

    public function mount(AnalyticsService $analytics): void
    {
        $this->loadData($analytics);
    }

    public function updatedPeriod(AnalyticsService $analytics): void
    {
        $this->loadData($analytics);
    }

    private function loadData(AnalyticsService $analytics): void
    {
        $from = now()->subDays((int) $this->period);
        $to = now();

        $this->salesData = $analytics->getSalesOverview($from, $to);
    }

    public function render()
    {
        return view('livewire.analytics-dashboard');
    }
}
```

---

## Django/Wagtail Stack

### Analytics with Pandas

```python
# apps/analytics/services.py

from datetime import date, timedelta
from decimal import Decimal
import pandas as pd
from django.db.models import Sum, Count, Avg
from apps.orders.models import Order, OrderItem


class AnalyticsService:
    """Service for generating analytics and reports."""

    def get_sales_overview(
        self,
        from_date: date,
        to_date: date,
    ) -> dict:
        """Get comprehensive sales overview."""
        orders = Order.objects.filter(
            created_at__date__range=[from_date, to_date],
            status='completed',
        )

        return {
            'total_revenue': orders.aggregate(Sum('total'))['total__sum'] or 0,
            'order_count': orders.count(),
            'average_order_value': orders.aggregate(Avg('total'))['total__avg'] or 0,
            'daily_sales': self.get_daily_sales(from_date, to_date),
            'top_products': self.get_top_products(from_date, to_date),
        }

    def get_daily_sales(
        self,
        from_date: date,
        to_date: date,
    ) -> pd.DataFrame:
        """Get daily sales data as DataFrame."""
        orders = Order.objects.filter(
            created_at__date__range=[from_date, to_date],
            status='completed',
        ).values('created_at__date').annotate(
            revenue=Sum('total'),
            orders=Count('id'),
        ).order_by('created_at__date')

        df = pd.DataFrame(list(orders))

        if df.empty:
            return pd.DataFrame(columns=['date', 'revenue', 'orders'])

        df = df.rename(columns={'created_at__date': 'date'})

        # Fill missing dates with zeros
        date_range = pd.date_range(from_date, to_date)
        df = df.set_index('date').reindex(date_range, fill_value=0).reset_index()
        df = df.rename(columns={'index': 'date'})

        return df

    def get_top_products(
        self,
        from_date: date,
        to_date: date,
        limit: int = 10,
    ) -> pd.DataFrame:
        """Get top selling products."""
        items = OrderItem.objects.filter(
            order__created_at__date__range=[from_date, to_date],
            order__status='completed',
        ).values(
            'product__id',
            'product__name',
        ).annotate(
            units_sold=Sum('quantity'),
            revenue=Sum('total'),
        ).order_by('-revenue')[:limit]

        return pd.DataFrame(list(items))

    def export_to_csv(self, df: pd.DataFrame, filename: str) -> str:
        """Export DataFrame to CSV file."""
        filepath = f'/tmp/{filename}.csv'
        df.to_csv(filepath, index=False)
        return filepath
```

### GraphQL Analytics

```python
# apps/analytics/schema.py

import strawberry
from datetime import date
from typing import List
from .services import AnalyticsService


@strawberry.type
class DailySales:
    date: date
    revenue: float
    orders: int


@strawberry.type
class TopProduct:
    id: str
    name: str
    units_sold: int
    revenue: float


@strawberry.type
class SalesOverview:
    total_revenue: float
    order_count: int
    average_order_value: float
    daily_sales: List[DailySales]
    top_products: List[TopProduct]


@strawberry.type
class AnalyticsQuery:
    @strawberry.field
    def sales_overview(
        self,
        from_date: date,
        to_date: date,
    ) -> SalesOverview:
        service = AnalyticsService()
        data = service.get_sales_overview(from_date, to_date)

        return SalesOverview(
            total_revenue=float(data['total_revenue']),
            order_count=data['order_count'],
            average_order_value=float(data['average_order_value']),
            daily_sales=[
                DailySales(
                    date=row['date'],
                    revenue=float(row['revenue']),
                    orders=row['orders'],
                )
                for _, row in data['daily_sales'].iterrows()
            ],
            top_products=[
                TopProduct(
                    id=str(row['product__id']),
                    name=row['product__name'],
                    units_sold=row['units_sold'],
                    revenue=float(row['revenue']),
                )
                for _, row in data['top_products'].iterrows()
            ],
        )
```

---

## React/Next.js Stack

### Analytics Dashboard

```tsx
// app/dashboard/analytics/page.tsx

import { Suspense } from 'react';
import { getSalesOverview } from '@/lib/api/analytics';
import { SalesChart } from '@/components/charts/SalesChart';
import { TopProductsTable } from '@/components/tables/TopProductsTable';
import { MetricCard } from '@/components/ui/MetricCard';

interface SearchParams {
  period?: string;
}

export default async function AnalyticsPage({
  searchParams,
}: {
  searchParams: Promise<SearchParams>;
}) {
  const { period = '30' } = await searchParams;
  const data = await getSalesOverview(parseInt(period));

  return (
    <div className="space-y-6">
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <MetricCard
          title="Total Revenue"
          value={formatCurrency(data.totalRevenue)}
          change={data.revenueChange}
        />
        <MetricCard
          title="Orders"
          value={data.orderCount.toLocaleString()}
          change={data.ordersChange}
        />
        <MetricCard
          title="Average Order Value"
          value={formatCurrency(data.averageOrderValue)}
          change={data.aovChange}
        />
      </div>

      <Suspense fallback={<ChartSkeleton />}>
        <SalesChart data={data.dailySales} />
      </Suspense>

      <Suspense fallback={<TableSkeleton />}>
        <TopProductsTable products={data.topProducts} />
      </Suspense>
    </div>
  );
}

function formatCurrency(pence: number): string {
  return new Intl.NumberFormat('en-GB', {
    style: 'currency',
    currency: 'GBP',
  }).format(pence / 100);
}
```

### Chart Component

```tsx
// components/charts/SalesChart.tsx

'use client';

import { useMemo } from 'react';
import {
  LineChart,
  Line,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
} from 'recharts';

interface DailySale {
  date: string;
  revenue: number;
  orders: number;
}

interface SalesChartProps {
  data: DailySale[];
}

export function SalesChart({ data }: SalesChartProps) {
  const formattedData = useMemo(
    () =>
      data.map((item) => ({
        ...item,
        date: new Date(item.date).toLocaleDateString('en-GB', {
          day: 'numeric',
          month: 'short',
        }),
        revenue: item.revenue / 100,
      })),
    [data]
  );

  return (
    <div className="bg-white p-6 rounded-lg shadow">
      <h3 className="text-lg font-semibold mb-4">Daily Sales</h3>
      <ResponsiveContainer width="100%" height={300}>
        <LineChart data={formattedData}>
          <CartesianGrid strokeDasharray="3 3" />
          <XAxis dataKey="date" />
          <YAxis
            tickFormatter={(value) =>
              new Intl.NumberFormat('en-GB', {
                style: 'currency',
                currency: 'GBP',
                notation: 'compact',
              }).format(value)
            }
          />
          <Tooltip
            formatter={(value: number) =>
              new Intl.NumberFormat('en-GB', {
                style: 'currency',
                currency: 'GBP',
              }).format(value)
            }
          />
          <Line
            type="monotone"
            dataKey="revenue"
            stroke="#2563eb"
            strokeWidth={2}
          />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
}
```

---

## React Native Stack

### Analytics Screen

```tsx
// src/screens/AnalyticsScreen.tsx

import React, { useState } from 'react';
import { View, Text, ScrollView, StyleSheet, Dimensions } from 'react-native';
import { LineChart } from 'react-native-chart-kit';
import { useAnalytics } from '@/hooks/useAnalytics';
import { MetricCard } from '@/components/MetricCard';
import { PeriodSelector } from '@/components/PeriodSelector';
import { LoadingSpinner } from '@/components/ui/LoadingSpinner';

const screenWidth = Dimensions.get('window').width;

export function AnalyticsScreen() {
  const [period, setPeriod] = useState(30);
  const { data, isLoading, error } = useAnalytics(period);

  if (isLoading) return <LoadingSpinner />;
  if (error) return <Text>Error loading analytics</Text>;

  const chartData = {
    labels: data.dailySales.slice(-7).map((d) =>
      new Date(d.date).toLocaleDateString('en-GB', { day: 'numeric' })
    ),
    datasets: [
      {
        data: data.dailySales.slice(-7).map((d) => d.revenue / 100),
      },
    ],
  };

  return (
    <ScrollView style={styles.container}>
      <PeriodSelector value={period} onChange={setPeriod} />

      <View style={styles.metrics}>
        <MetricCard
          title="Revenue"
          value={formatCurrency(data.totalRevenue)}
          change={data.revenueChange}
        />
        <MetricCard
          title="Orders"
          value={data.orderCount.toString()}
          change={data.ordersChange}
        />
      </View>

      <View style={styles.chartContainer}>
        <Text style={styles.chartTitle}>Daily Revenue</Text>
        <LineChart
          data={chartData}
          width={screenWidth - 32}
          height={220}
          chartConfig={{
            backgroundColor: '#ffffff',
            backgroundGradientFrom: '#ffffff',
            backgroundGradientTo: '#ffffff',
            decimalPlaces: 0,
            color: (opacity = 1) => `rgba(37, 99, 235, ${opacity})`,
            style: { borderRadius: 16 },
          }}
          bezier
          style={styles.chart}
        />
      </View>
    </ScrollView>
  );
}

function formatCurrency(pence: number): string {
  return new Intl.NumberFormat('en-GB', {
    style: 'currency',
    currency: 'GBP',
  }).format(pence / 100);
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  metrics: {
    flexDirection: 'row',
    padding: 16,
    gap: 12,
  },
  chartContainer: {
    backgroundColor: '#ffffff',
    margin: 16,
    padding: 16,
    borderRadius: 12,
  },
  chartTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 12,
  },
  chart: {
    borderRadius: 16,
  },
});
```

### Analytics Hook

```tsx
// src/hooks/useAnalytics.ts

import { useQuery } from '@tanstack/react-query';
import { api } from '@/lib/api';

interface DailySale {
  date: string;
  revenue: number;
  orders: number;
}

interface AnalyticsData {
  totalRevenue: number;
  orderCount: number;
  averageOrderValue: number;
  revenueChange: number;
  ordersChange: number;
  dailySales: DailySale[];
}

export function useAnalytics(days: number) {
  return useQuery({
    queryKey: ['analytics', days],
    queryFn: () => api.get<AnalyticsData>(`/analytics?days=${days}`),
    staleTime: 5 * 60 * 1000,
  });
}
```
