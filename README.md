<div align="center">

<img src="https://avatars.githubusercontent.com/u/244616883?s=200&v=4" alt="BlackRoad OS Logo" width="120" />

# `blackroad-web-scraper`

**Enterprise-grade, headless web scraping for Node.js — powered by BlackRoad OS, Inc.**

[![npm version](https://img.shields.io/npm/v/blackroad-web-scraper?color=black&label=npm)](https://www.npmjs.com/package/blackroad-web-scraper)
[![License: Proprietary](https://img.shields.io/badge/license-Proprietary-red)](./LICENSE)
[![Node.js ≥ 18](https://img.shields.io/badge/node-%E2%89%A518-brightgreen)](https://nodejs.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue)](https://www.typescriptlang.org/)
[![Stripe Billing](https://img.shields.io/badge/billing-Stripe-635BFF)](https://stripe.com)
[![BlackRoad OS](https://img.shields.io/badge/by-BlackRoad%20OS-000000)](https://blackroad.os)

> Reliable, scalable, production-hardened web data extraction with first-class TypeScript support,  
> Stripe-based subscription billing, and an ergonomic developer API.

</div>

---

## Table of Contents

1. [Overview](#1-overview)
2. [Key Features](#2-key-features)
3. [Architecture](#3-architecture)
4. [Requirements](#4-requirements)
5. [Installation](#5-installation)
6. [Quick Start](#6-quick-start)
7. [Configuration Reference](#7-configuration-reference)
8. [API Reference](#8-api-reference)
   - 8.1 [Scraper](#81-scraper)
   - 8.2 [Session](#82-session)
   - 8.3 [Pipeline](#83-pipeline)
   - 8.4 [Extractors](#84-extractors)
   - 8.5 [Transformers](#85-transformers)
   - 8.6 [Storage Adapters](#86-storage-adapters)
9. [Stripe Billing Integration](#9-stripe-billing-integration)
   - 9.1 [Plans & Usage Tiers](#91-plans--usage-tiers)
   - 9.2 [Setup](#92-setup)
   - 9.3 [Webhook Handling](#93-webhook-handling)
   - 9.4 [Metered Billing](#94-metered-billing)
10. [Authentication & Security](#10-authentication--security)
11. [Proxy & Rate Limiting](#11-proxy--rate-limiting)
12. [Error Handling](#12-error-handling)
13. [Logging & Observability](#13-logging--observability)
14. [End-to-End Testing](#14-end-to-end-testing)
15. [CLI Reference](#15-cli-reference)
16. [TypeScript Support](#16-typescript-support)
17. [Changelog](#17-changelog)
18. [Support](#18-support)
19. [License](#19-license)

---

## 1. Overview

`blackroad-web-scraper` is the official BlackRoad OS web-data extraction SDK.
It provides a declarative, pipeline-based interface for crawling, scraping, transforming, and
persisting web data at any scale — from rapid prototypes to enterprise production workloads
processing millions of pages per day.

Billing is handled transparently via **Stripe** subscriptions and metered usage, so teams are
charged only for what they consume. A lightweight Stripe integration layer ships inside the
package itself, requiring zero external billing infrastructure to get started.

---

## 2. Key Features

| Feature | Description |
|---|---|
| **Headless Browser Support** | Chromium/Firefox via Playwright for JavaScript-heavy pages |
| **Static HTML Parsing** | Cheerio-powered fast extraction for static content |
| **Pipeline Architecture** | Composable extract → transform → load (ETL) stages |
| **Stripe Billing** | Built-in metered billing, plan gates, and webhook handlers |
| **Proxy Rotation** | First-class support for residential and datacenter proxy pools |
| **Rate Limiting** | Configurable per-domain throttling to respect `robots.txt` |
| **TypeScript-First** | 100% typed public API with full generics support |
| **Retry & Resilience** | Exponential back-off, jitter, and circuit breakers out of the box |
| **Storage Adapters** | JSON, CSV, SQLite, PostgreSQL, S3, and custom adapters |
| **CLI** | `brscrape` CLI for interactive and scriptable scraping tasks |
| **Observability** | OpenTelemetry traces, structured logs, and Prometheus metrics |

---

## 3. Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                        blackroad-web-scraper                       │
│                                                                   │
│  ┌─────────┐   ┌──────────┐   ┌─────────────┐   ┌─────────────┐ │
│  │ Scraper │──▶│ Session  │──▶│  Extractor  │──▶│ Transformer │ │
│  │  Core   │   │ Manager  │   │ (HTML/JS)   │   │  Pipeline   │ │
│  └─────────┘   └──────────┘   └─────────────┘   └──────┬──────┘ │
│                                                          │        │
│  ┌───────────────────┐   ┌──────────────────────────────▼──────┐ │
│  │  Stripe Billing   │   │          Storage Adapter            │ │
│  │  (metered usage)  │   │  JSON / CSV / SQLite / PG / S3      │ │
│  └───────────────────┘   └─────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────────┘
```

---

## 4. Requirements

| Dependency | Minimum Version |
|---|---|
| Node.js | 18 LTS |
| npm | 9 |
| TypeScript _(optional)_ | 5.0 |
| Stripe Account _(for billing)_ | Any paid plan |

---

## 5. Installation

### npm

```bash
npm install blackroad-web-scraper
```

### yarn

```bash
yarn add blackroad-web-scraper
```

### pnpm

```bash
pnpm add blackroad-web-scraper
```

> **Note** — To use headless-browser extraction, also install the Playwright browser binaries:
>
> ```bash
> npx playwright install chromium
> ```

---

## 6. Quick Start

### CommonJS

```js
const { Scraper } = require('blackroad-web-scraper');

const scraper = new Scraper({ apiKey: process.env.BLACKROAD_API_KEY });

(async () => {
  const result = await scraper.fetch('https://example.com');
  console.log(result.title);
})();
```

### ES Modules / TypeScript

```ts
import { Scraper, type ScrapeResult } from 'blackroad-web-scraper';

const scraper = new Scraper({ apiKey: process.env.BLACKROAD_API_KEY });

const result: ScrapeResult = await scraper.fetch('https://example.com');
console.log(result.title);
```

### Pipeline (recommended for production)

```ts
import { Pipeline, HtmlExtractor, JsonStorage } from 'blackroad-web-scraper';

const pipeline = new Pipeline()
  .extract(new HtmlExtractor({ selector: 'article' }))
  .transform((html) => ({ body: html.trim() }))
  .load(new JsonStorage({ path: './output/articles.json' }));

await pipeline.run(['https://example.com/blog/1', 'https://example.com/blog/2']);
```

---

## 7. Configuration Reference

Create a `blackroad.config.ts` (or `.js`) file at your project root, or pass options directly
to the constructor.

```ts
import { defineConfig } from 'blackroad-web-scraper';

export default defineConfig({
  // Authentication
  apiKey: process.env.BLACKROAD_API_KEY,          // required

  // Stripe billing
  stripe: {
    secretKey: process.env.STRIPE_SECRET_KEY,
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET,
    plan: 'growth',                               // 'starter' | 'growth' | 'enterprise'
  },

  // Browser settings
  browser: {
    engine: 'chromium',                           // 'chromium' | 'firefox' | 'webkit'
    headless: true,
    timeout: 30_000,                              // ms
    viewport: { width: 1280, height: 720 },
  },

  // HTTP settings (static pages)
  http: {
    timeout: 10_000,                              // ms
    headers: { 'Accept-Language': 'en-US,en;q=0.9' },
    followRedirects: true,
    maxRedirects: 5,
  },

  // Proxy
  proxy: {
    enabled: false,
    pool: [],                                     // ['http://user:pass@host:port', ...]
    rotation: 'round-robin',                      // 'round-robin' | 'random' | 'sticky'
  },

  // Rate limiting
  rateLimit: {
    requestsPerSecond: 2,
    respectRobotsTxt: true,
    crawlDelay: 1000,                             // ms between requests to the same domain
  },

  // Retry policy
  retry: {
    maxAttempts: 3,
    backoff: 'exponential',                       // 'linear' | 'exponential'
    jitter: true,
  },

  // Storage
  storage: {
    adapter: 'json',                              // 'json' | 'csv' | 'sqlite' | 'postgres' | 's3'
    outputDir: './output',
  },

  // Observability
  observability: {
    logging: { level: 'info' },                  // 'debug' | 'info' | 'warn' | 'error'
    metrics: { enabled: false },
    tracing: { enabled: false },
  },
});
```

---

## 8. API Reference

### 8.1 `Scraper`

The top-level class for single-URL and batch-URL fetching.

```ts
class Scraper {
  constructor(options: ScraperOptions);

  /** Fetch a single URL and return a ScrapeResult. */
  fetch(url: string, options?: FetchOptions): Promise<ScrapeResult>;

  /** Fetch multiple URLs concurrently (honours concurrency limits). */
  fetchAll(urls: string[], options?: FetchAllOptions): Promise<ScrapeResult[]>;

  /** Stream results as they resolve — useful for large batches. */
  stream(urls: string[], options?: FetchAllOptions): AsyncGenerator<ScrapeResult>;

  /** Destroy the underlying browser/HTTP pool. */
  close(): Promise<void>;
}
```

#### `ScrapeResult`

```ts
interface ScrapeResult {
  url: string;
  status: number;
  title: string | null;
  html: string;
  text: string;
  links: string[];
  metadata: Record<string, string>;
  timing: { fetchMs: number; parseMs: number; totalMs: number };
  usage: { requestUnits: number };
}
```

---

### 8.2 `Session`

Manages cookies, authentication state, and shared browser contexts across requests.

```ts
class Session {
  constructor(options?: SessionOptions);

  /** Log in via a form submission and persist cookies. */
  login(url: string, credentials: Credentials): Promise<void>;

  /** Return all cookies in the current session. */
  cookies(): Promise<Cookie[]>;

  /** Attach this session to a Scraper instance. */
  attach(scraper: Scraper): Scraper;

  close(): Promise<void>;
}
```

---

### 8.3 `Pipeline`

Composable ETL pipeline with built-in concurrency control.

```ts
class Pipeline<TIn = string, TOut = unknown> {
  extract<T>(extractor: Extractor<T>): Pipeline<TIn, T>;
  transform<T>(fn: (input: TOut) => T | Promise<T>): Pipeline<TIn, T>;
  load(storage: StorageAdapter<TOut>): Pipeline<TIn, TOut>;
  run(inputs: TIn[], options?: PipelineRunOptions): Promise<PipelineReport>;
}
```

---

### 8.4 `Extractors`

| Class | Description |
|---|---|
| `HtmlExtractor` | Extract HTML subtrees via CSS selectors |
| `TextExtractor` | Strip HTML and return plain text |
| `JsonLdExtractor` | Parse structured data (`application/ld+json`) |
| `OpenGraphExtractor` | Extract Open Graph meta tags |
| `TableExtractor` | Parse `<table>` elements into row arrays |
| `BrowserExtractor` | Full headless-browser execution + DOM snapshot |
| `XmlExtractor` | Parse XML/RSS/Atom feeds |

---

### 8.5 `Transformers`

| Helper | Description |
|---|---|
| `mapFields(schema)` | Rename / pick fields using a mapping object |
| `filterNulls()` | Remove null and undefined values from records |
| `deduplicateBy(key)` | De-duplicate records by a given field |
| `normalizeUrls()` | Resolve relative URLs against the base URL |
| `truncate(maxLength)` | Truncate string fields to a maximum length |

---

### 8.6 `Storage Adapters`

| Class | Description |
|---|---|
| `JsonStorage` | Write newline-delimited JSON (NDJSON) |
| `CsvStorage` | Write RFC 4180-compliant CSV |
| `SqliteStorage` | Persist records to a local SQLite database |
| `PostgresStorage` | Stream records into a PostgreSQL table |
| `S3Storage` | Upload batched records to Amazon S3 |
| `ConsoleStorage` | Print records to stdout (development) |

---

## 9. Stripe Billing Integration

`blackroad-web-scraper` ships a first-party Stripe integration that gates feature usage,
enforces plan limits, and reports metered consumption automatically.
No external billing service is required.

### 9.1 Plans & Usage Tiers

| Plan | Request Units / mo | Concurrency | Headless Browser | Support |
|---|---|---|---|---|
| **Starter** | 100,000 | 5 | ✗ | Community |
| **Growth** | 1,000,000 | 25 | ✓ | Email (48 h SLA) |
| **Enterprise** | Unlimited | Custom | ✓ | Dedicated (4 h SLA) |

> **Request Units** — One request unit is consumed per HTTP request. Headless-browser requests
> consume five (5) units to account for browser overhead.

---

### 9.2 Setup

#### 1. Install the Stripe peer dependency

```bash
npm install stripe
```

#### 2. Add environment variables

```dotenv
BLACKROAD_API_KEY=bro_live_xxxxxxxxxxxxxxxx
STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxxxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxxxxxxxxx
STRIPE_PRICE_ID=price_xxxxxxxxxxxxxxxx
```

#### 3. Initialise the billing client

```ts
import { Scraper, StripeBilling } from 'blackroad-web-scraper';

const billing = new StripeBilling({
  secretKey: process.env.STRIPE_SECRET_KEY!,
  webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
});

const scraper = new Scraper({
  apiKey: process.env.BLACKROAD_API_KEY!,
  billing,
});
```

---

### 9.3 Webhook Handling

`blackroad-web-scraper` provides a ready-made Express/Fastify middleware for handling Stripe
webhook events — subscription updates, cancellations, and payment failures.

#### Express

```ts
import express from 'express';
import { stripeWebhookMiddleware } from 'blackroad-web-scraper';

const app = express();

app.post(
  '/webhooks/stripe',
  express.raw({ type: 'application/json' }),
  stripeWebhookMiddleware({
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
    onSubscriptionUpdated: (subscription) => {
      console.log('Subscription updated:', subscription.id);
    },
    onSubscriptionCancelled: (subscription) => {
      console.log('Subscription cancelled:', subscription.id);
    },
    onPaymentFailed: (invoice) => {
      console.error('Payment failed for invoice:', invoice.id);
    },
  }),
);
```

#### Fastify

```ts
import Fastify from 'fastify';
import { stripeWebhookPlugin } from 'blackroad-web-scraper';

const fastify = Fastify();
await fastify.register(stripeWebhookPlugin, {
  webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
  path: '/webhooks/stripe',
});
```

---

### 9.4 Metered Billing

Usage is reported automatically after each successful request batch. You can also
report usage manually:

```ts
import { StripeBilling } from 'blackroad-web-scraper';

const billing = new StripeBilling({ secretKey: process.env.STRIPE_SECRET_KEY! });

// Report 250 request units for a subscription item
await billing.reportUsage({
  subscriptionItemId: 'si_xxxxxxxxxxxxxxxx',
  quantity: 250,
  action: 'increment',
});
```

---

## 10. Authentication & Security

### API Key Authentication

All requests to the BlackRoad OS API require a valid API key passed either via the
`apiKey` constructor option or the `BLACKROAD_API_KEY` environment variable.

```ts
const scraper = new Scraper({ apiKey: process.env.BLACKROAD_API_KEY });
```

### Secret Rotation

API keys can be rotated at any time from the [BlackRoad OS dashboard](https://dashboard.blackroad.os).
The SDK will automatically pick up the new key on its next initialisation.

### Data Security

- All HTTP client connections enforce **TLS 1.2+**.
- Scraped data is never transmitted to BlackRoad OS servers; it remains in your environment.
- Browser sandbox isolation is enforced via Playwright's built-in process isolation.

---

## 11. Proxy & Rate Limiting

### Proxy Configuration

```ts
const scraper = new Scraper({
  apiKey: process.env.BLACKROAD_API_KEY,
  proxy: {
    enabled: true,
    pool: [
      'http://user:pass@proxy1.example.com:8080',
      'http://user:pass@proxy2.example.com:8080',
    ],
    rotation: 'round-robin',
  },
});
```

### `robots.txt` Compliance

By default, `respectRobotsTxt: true` is set in the rate-limit configuration.
The SDK fetches and caches each domain's `robots.txt` on first visit and will
skip disallowed paths, logging a warning instead.

To disable (use responsibly):

```ts
const scraper = new Scraper({
  apiKey: process.env.BLACKROAD_API_KEY,
  rateLimit: { respectRobotsTxt: false },
});
```

---

## 12. Error Handling

All SDK errors extend `BlackroadError` and carry a machine-readable `code` property:

```ts
import { BlackroadError, ErrorCode } from 'blackroad-web-scraper';

try {
  const result = await scraper.fetch('https://example.com');
} catch (err) {
  if (err instanceof BlackroadError) {
    switch (err.code) {
      case ErrorCode.RateLimitExceeded:
        console.error('Rate limit hit — back off and retry.');
        break;
      case ErrorCode.PlanLimitReached:
        console.error('Monthly request quota exhausted — upgrade your plan.');
        break;
      case ErrorCode.NavigationTimeout:
        console.error('Page did not load within the configured timeout.');
        break;
      default:
        console.error('Unexpected error:', err.message);
    }
  }
}
```

---

## 13. Logging & Observability

### Structured Logging

The SDK emits structured JSON logs via a built-in logger. Set the `LOG_LEVEL`
environment variable or configure via `defineConfig`:

```bash
LOG_LEVEL=debug node scraper.js
```

### OpenTelemetry Tracing

```ts
const scraper = new Scraper({
  apiKey: process.env.BLACKROAD_API_KEY,
  observability: {
    tracing: {
      enabled: true,
      exporterUrl: 'http://localhost:4318/v1/traces',
    },
  },
});
```

### Prometheus Metrics

```ts
const scraper = new Scraper({
  apiKey: process.env.BLACKROAD_API_KEY,
  observability: {
    metrics: { enabled: true, port: 9090 },
  },
});
// Metrics available at http://localhost:9090/metrics
```

---

## 14. End-to-End Testing

`blackroad-web-scraper` ships a test utilities module to support integration and E2E test suites.

### Mock Server

```ts
import { createMockServer } from 'blackroad-web-scraper/testing';

const server = await createMockServer([
  {
    path: '/products',
    html: '<html><body><h1>Products</h1></body></html>',
  },
]);

const scraper = new Scraper({
  apiKey: 'test-key',
  baseUrl: server.url,
});

const result = await scraper.fetch('/products');
expect(result.title).toBe('Products');

await server.close();
```

### Fixtures

```ts
import { loadFixture } from 'blackroad-web-scraper/testing';

// Loads HTML from __fixtures__/product-page.html
const html = loadFixture('product-page');
```

### Jest / Vitest Integration

```ts
import { ScraperMock } from 'blackroad-web-scraper/testing';

jest.mock('blackroad-web-scraper', () => ({
  Scraper: ScraperMock,
}));
```

---

## 15. CLI Reference

The `brscrape` CLI is installed automatically alongside the package.

```
Usage: brscrape [command] [options]

Commands:
  fetch <url>           Fetch and print a single URL
  crawl <url>           Crawl a site from a seed URL
  pipeline <config>     Execute a pipeline from a config file
  billing status        Display current billing plan and usage
  billing upgrade       Open the billing upgrade portal
  help [command]        Display help for a command

Global Options:
  --api-key <key>       BlackRoad OS API key (overrides BLACKROAD_API_KEY)
  --config <path>       Path to blackroad.config.ts|js (default: ./blackroad.config)
  --output <path>       Output directory (default: ./output)
  --format <fmt>        Output format: json | csv | ndjson (default: json)
  --log-level <level>   debug | info | warn | error (default: info)
  --version             Print the installed package version
  --help                Print this help message
```

#### Examples

```bash
# Fetch a single page and print the result
brscrape fetch https://example.com

# Crawl a site with a depth limit of 3
brscrape crawl https://example.com --depth 3 --output ./data

# Run a pipeline defined in a config file
brscrape pipeline ./pipelines/products.ts

# Check your billing plan and remaining request units
brscrape billing status
```

---

## 16. TypeScript Support

`blackroad-web-scraper` is written in TypeScript and ships with bundled type declarations.
No `@types/*` package is required.

```bash
npm install blackroad-web-scraper   # types included
```

The package exposes the following type-entry points:

```ts
import type {
  ScraperOptions,
  FetchOptions,
  ScrapeResult,
  PipelineRunOptions,
  PipelineReport,
  StorageAdapter,
  Extractor,
  BillingOptions,
  StripePlan,
  BlackroadError,
  ErrorCode,
} from 'blackroad-web-scraper';
```

TypeScript version ≥ 5.0 with `"moduleResolution": "bundler"` or `"node16"` is recommended.

---

## 17. Changelog

See the full release history in [CHANGELOG.md](./CHANGELOG.md).

---

## 18. Support

| Channel | Details |
|---|---|
| **Documentation** | [https://docs.blackroad.os/web-scraper](https://docs.blackroad.os/web-scraper) |
| **npm Package** | [https://www.npmjs.com/package/blackroad-web-scraper](https://www.npmjs.com/package/blackroad-web-scraper) |
| **Issue Tracker** | [GitHub Issues](https://github.com/BlackRoad-OS/blackroad-web-scraper/issues) |
| **Email** | [support@blackroad.os](mailto:support@blackroad.os) |
| **Enterprise Sales** | [enterprise@blackroad.os](mailto:enterprise@blackroad.os) |

---

## 19. License

This software is the exclusive proprietary property of **BlackRoad OS, Inc.**  
All rights reserved. Public visibility of this repository does **not** constitute
an open-source release.

See [LICENSE](./LICENSE) for the full terms of the BlackRoad OS, Inc. Proprietary
Software License.

---

<div align="center">

Copyright © 2024–2026 BlackRoad OS, Inc. — All Rights Reserved.  
Founder, CEO & Sole Stockholder: Alexa Louise Amundson

</div>
