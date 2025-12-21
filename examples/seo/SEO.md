# SEO Examples

## Overview

SEO implementation examples for server-rendered and client-side applications.

---

## Table of Contents

- [Overview](#overview)
- [TALL Stack (Laravel 12)](#tall-stack-laravel-12)
- [Django/Wagtail Stack](#djangowagtail-stack)
- [React/Next.js Stack](#reactnextjs-stack)
- [React Native Stack](#react-native-stack)

## TALL Stack (Laravel 12)

### Meta Tag Service

```php
<?php
// app/Services/SeoService.php

namespace App\Services;

use Illuminate\Support\HtmlString;

final class SeoService
{
    private string $title = '';
    private string $description = '';
    private string $canonicalUrl = '';
    private array $openGraph = [];
    private array $twitter = [];
    private array $jsonLd = [];

    public function title(string $title): self
    {
        $this->title = $title . ' | ' . config('app.name');
        return $this;
    }

    public function description(string $description): self
    {
        $this->description = substr($description, 0, 160);
        return $this;
    }

    public function canonical(string $url): self
    {
        $this->canonicalUrl = $url;
        return $this;
    }

    public function openGraph(array $data): self
    {
        $this->openGraph = array_merge([
            'type' => 'website',
            'locale' => 'en_GB',
            'site_name' => config('app.name'),
        ], $data);
        return $this;
    }

    public function jsonLd(array $data): self
    {
        $this->jsonLd = $data;
        return $this;
    }

    public function render(): HtmlString
    {
        $html = [];

        if ($this->title) {
            $html[] = "<title>{$this->title}</title>";
            $html[] = "<meta property=\"og:title\" content=\"{$this->title}\">";
        }

        if ($this->description) {
            $html[] = "<meta name=\"description\" content=\"{$this->description}\">";
            $html[] = "<meta property=\"og:description\" content=\"{$this->description}\">";
        }

        if ($this->canonicalUrl) {
            $html[] = "<link rel=\"canonical\" href=\"{$this->canonicalUrl}\">";
            $html[] = "<meta property=\"og:url\" content=\"{$this->canonicalUrl}\">";
        }

        foreach ($this->openGraph as $property => $content) {
            $html[] = "<meta property=\"og:{$property}\" content=\"{$content}\">";
        }

        if (!empty($this->jsonLd)) {
            $json = json_encode($this->jsonLd, JSON_UNESCAPED_SLASHES);
            $html[] = "<script type=\"application/ld+json\">{$json}</script>";
        }

        return new HtmlString(implode("\n", $html));
    }
}
```

### Sitemap Generation

```php
<?php
// app/Console/Commands/GenerateSitemap.php

namespace App\Console\Commands;

use App\Models\Product;
use App\Models\Page;
use Illuminate\Console\Command;
use Spatie\Sitemap\Sitemap;
use Spatie\Sitemap\Tags\Url;

class GenerateSitemap extends Command
{
    protected $signature = 'sitemap:generate';
    protected $description = 'Generate the sitemap';

    public function handle(): int
    {
        $sitemap = Sitemap::create();

        // Static pages
        $sitemap->add(Url::create('/')->setPriority(1.0)->setChangeFrequency('daily'));
        $sitemap->add(Url::create('/about')->setPriority(0.8));
        $sitemap->add(Url::create('/contact')->setPriority(0.8));

        // Dynamic pages
        Page::published()->each(function (Page $page) use ($sitemap) {
            $sitemap->add(
                Url::create("/pages/{$page->slug}")
                    ->setLastModificationDate($page->updated_at)
                    ->setPriority(0.7)
            );
        });

        // Products
        Product::active()->each(function (Product $product) use ($sitemap) {
            $sitemap->add(
                Url::create("/products/{$product->slug}")
                    ->setLastModificationDate($product->updated_at)
                    ->setPriority(0.9)
            );
        });

        $sitemap->writeToFile(public_path('sitemap.xml'));

        $this->info('Sitemap generated successfully!');
        return self::SUCCESS;
    }
}
```

---

## Django/Wagtail Stack

### SEO Mixin

```python
# apps/core/seo.py

from django.utils.html import format_html
from django.utils.safestring import mark_safe
import json


class SEOMixin:
    """Mixin for adding SEO capabilities to views."""

    seo_title: str = ''
    seo_description: str = ''

    def get_seo_title(self) -> str:
        return f'{self.seo_title} | My Site'

    def get_seo_description(self) -> str:
        return self.seo_description[:160]

    def get_canonical_url(self) -> str:
        return self.request.build_absolute_uri()

    def get_open_graph(self) -> dict:
        return {
            'type': 'website',
            'locale': 'en_GB',
            'site_name': 'My Site',
            'title': self.get_seo_title(),
            'description': self.get_seo_description(),
            'url': self.get_canonical_url(),
        }

    def get_json_ld(self) -> dict | None:
        return None

    def get_seo_context(self) -> dict:
        return {
            'seo_title': self.get_seo_title(),
            'seo_description': self.get_seo_description(),
            'canonical_url': self.get_canonical_url(),
            'open_graph': self.get_open_graph(),
            'json_ld': self.get_json_ld(),
        }

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context.update(self.get_seo_context())
        return context
```

### Wagtail SEO Panel

```python
# apps/cms/models.py

from wagtail.models import Page
from wagtail.admin.panels import FieldPanel, MultiFieldPanel


class SEOFields:
    """SEO fields mixin for Wagtail pages."""

    seo_title = models.CharField(max_length=70, blank=True)
    seo_description = models.CharField(max_length=160, blank=True)
    og_image = models.ForeignKey(
        'wagtailimages.Image',
        null=True,
        blank=True,
        on_delete=models.SET_NULL,
        related_name='+',
    )

    seo_panels = [
        MultiFieldPanel([
            FieldPanel('seo_title'),
            FieldPanel('seo_description'),
            FieldPanel('og_image'),
        ], heading='SEO'),
    ]

    def get_meta_title(self) -> str:
        return self.seo_title or self.title

    def get_meta_description(self) -> str:
        return self.seo_description or ''


class ArticlePage(SEOFields, Page):
    """Article page with SEO support."""

    content_panels = Page.content_panels + [
        # ... content fields
    ]

    promote_panels = Page.promote_panels + SEOFields.seo_panels
```

---

## React/Next.js Stack

### Metadata API

```tsx
// app/products/[slug]/page.tsx

import { Metadata } from 'next';
import { getProduct } from '@/lib/api';

interface Props {
  params: Promise<{ slug: string }>;
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const product = await getProduct(slug);

  return {
    title: `${product.name} | My Store`,
    description: product.description.slice(0, 160),
    openGraph: {
      title: product.name,
      description: product.description,
      images: [{ url: product.imageUrl, width: 1200, height: 630 }],
      type: 'product',
      locale: 'en_GB',
    },
    twitter: {
      card: 'summary_large_image',
      title: product.name,
      description: product.description,
      images: [product.imageUrl],
    },
    alternates: {
      canonical: `https://mystore.com/products/${slug}`,
    },
  };
}

export default async function ProductPage({ params }: Props) {
  const { slug } = await params;
  const product = await getProduct(slug);

  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    description: product.description,
    image: product.imageUrl,
    offers: {
      '@type': 'Offer',
      price: (product.price / 100).toFixed(2),
      priceCurrency: 'GBP',
      availability: product.inStock
        ? 'https://schema.org/InStock'
        : 'https://schema.org/OutOfStock',
    },
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      {/* Page content */}
    </>
  );
}
```

### Sitemap Generation

```tsx
// app/sitemap.ts

import { MetadataRoute } from 'next';
import { getProducts, getPages } from '@/lib/api';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = 'https://mystore.com';

  const products = await getProducts();
  const pages = await getPages();

  const productUrls = products.map((product) => ({
    url: `${baseUrl}/products/${product.slug}`,
    lastModified: product.updatedAt,
    changeFrequency: 'weekly' as const,
    priority: 0.8,
  }));

  const pageUrls = pages.map((page) => ({
    url: `${baseUrl}/${page.slug}`,
    lastModified: page.updatedAt,
    changeFrequency: 'monthly' as const,
    priority: 0.6,
  }));

  return [
    {
      url: baseUrl,
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 1,
    },
    ...productUrls,
    ...pageUrls,
  ];
}
```

---

## React Native Stack

### App Store Optimisation

```tsx
// app.config.ts

import { ExpoConfig } from 'expo/config';

const config: ExpoConfig = {
  name: 'My App',
  slug: 'my-app',
  description: 'A brief, keyword-rich description for app stores',
  ios: {
    bundleIdentifier: 'com.example.myapp',
    buildNumber: '1.0.0',
    infoPlist: {
      // App Tracking Transparency
      NSUserTrackingUsageDescription:
        'This identifier will be used to deliver personalised ads to you.',
    },
  },
  android: {
    package: 'com.example.myapp',
    versionCode: 1,
  },
  extra: {
    // Keywords for discoverability
    keywords: ['shopping', 'ecommerce', 'deals'],
  },
};

export default config;
```

### Deep Linking for SEO

```tsx
// app.config.ts

export default {
  // ... other config
  scheme: 'myapp',
  web: {
    bundler: 'metro',
  },
  plugins: [
    [
      'expo-router',
      {
        origin: 'https://myapp.com',
        asyncRoutes: {
          web: true,
          native: false,
        },
      },
    ],
  ],
};
```

```tsx
// app/products/[slug].tsx

import { useLocalSearchParams } from 'expo-router';
import Head from 'expo-router/head';

export default function ProductScreen() {
  const { slug } = useLocalSearchParams<{ slug: string }>();
  const product = useProduct(slug);

  return (
    <>
      <Head>
        <title>{product?.name} | My App</title>
        <meta name="description" content={product?.description} />
      </Head>
      {/* Screen content */}
    </>
  );
}
```
