Я не могу создать репозиторий или загрузить файл, так как исчерпал лимит вызовов инструментов в этом сообщении. Давай я сначала подготовлю полный spec-документ, а в следующем сообщении создам репозиторий и загружу его.

Вот полный Protocol Specification который нужно опубликовать:

***

# CiteFlow Protocol Specification v0.1

**Status:** Draft  
**Date:** 2026-03-30  
**License:** Apache 2.0

***

## Abstract

CiteFlow is an open protocol that enables website owners to declare machine-readable policies governing how AI systems, search engines, and autonomous agents may access, cite, quote, and monetize their content. The protocol defines a manifest format, HTTP semantics, tool endpoint conventions, handoff session schema, and settlement event taxonomy.

***

## 1. Motivation

AI-powered search and agent runtimes increasingly consume web content without a structured contract between content producers and consumers. Existing mechanisms (`robots.txt`, `noindex`, schema.org) address crawl permissions and structured data but provide no framework for:

- granular citation licensing;
- usage-based pricing for content access;
- tool/action endpoint discovery;
- context-preserving handoff to the origin site;
- settlement and attribution accounting.

CiteFlow fills this gap with a minimal, composable protocol designed to be framework-agnostic and incrementally adoptable.

***

## 2. Definitions

| Term | Definition |
|---|---|
| **Publisher** | The operator of a website implementing CiteFlow |
| **Consumer** | An AI system, search engine, or agent that reads CiteFlow policies |
| **Answer Unit** | A discrete piece of citable content (paragraph, table, price, FAQ, product card) |
| **Tool** | A callable endpoint exposed by the Publisher for Consumer use |
| **Handoff Session** | A signed, time-limited context object passed from Consumer to Publisher |
| **Policy** | A rule set governing how a Consumer may interact with a resource |
| **Settlement Event** | A billable event logged for accounting purposes |

***

## 3. Manifest

### 3.1 Location

Publishers MUST serve the manifest at:

```
GET /.well-known/citeflow.json
Content-Type: application/json
```

### 3.2 Schema

```json
{
  "$schema": "https://citeflow.cloud/schemas/v0.1/manifest.json",
  "version": "0.1",
  "site": {
    "id": "string — canonical hostname",
    "name": "string",
    "canonical": "string — https URL",
    "locale": "string — BCP 47",
    "contact": "string — email or URL"
  },
  "policies": {
    "default": "PolicyMode",
    "citation": {
      "freeChars": "integer",
      "requireCanonicalLink": "boolean",
      "requireBrandMention": "boolean",
      "requireSourceBadge": "boolean",
      "paidAboveChars": "integer",
      "pricePerKB": "number",
      "currency": "string — ISO 4217"
    },
    "crawl": {
      "mode": "allow | metered | paid | block",
      "pricePerMB": "number",
      "currency": "string"
    },
    "training": {
      "mode": "allow | deny | licensed",
      "contactForLicense": "string"
    }
  },
  "zones": [
    {
      "match": "string — glob pattern",
      "mode": "PolicyMode",
      "citation": { "...CitationPolicy override..." },
      "crawl": { "...CrawlPolicy override..." },
      "expose": ["string — structured data types"]
    }
  ],
  "bots": {
    "botId": {
      "mode": "PolicyMode",
      "revenueShare": "number — fraction 0–1"
    }
  },
  "tools": [
    {
      "name": "string",
      "description": "string",
      "endpoint": "string — relative path",
      "method": "GET | POST",
      "billing": "free | per-call | per-qualified-lead | per-sale | revenue-share",
      "price": "number",
      "currency": "string",
      "inputSchema": "object — JSON Schema",
      "auth": "none | bearer | signed-request"
    }
  ],
  "handoff": {
    "enabled": "boolean",
    "endpoint": "string — relative path",
    "ttlSeconds": "integer",
    "signed": "boolean",
    "signingAlgorithm": "HS256 | RS256 | ES256"
  },
  "attribution": {
    "preferredLabel": "string",
    "preferredUrl": "string",
    "logoUrl": "string",
    "badgeUrl": "string"
  },
  "freshness": {
    "defaultTTLSeconds": "integer",
    "signatureAlgorithm": "sha256 | sha512",
    "lastVerified": "string — ISO 8601"
  }
}
```

### 3.3 PolicyMode Values

| Value | Meaning |
|---|---|
| `allow` | Consumer may access freely |
| `allow-with-attribution` | Consumer may access; attribution is required |
| `allow-with-limit` | Consumer may access up to `citation.freeChars` characters |
| `paid` | Consumer must negotiate payment before access |
| `revenue-share` | Access is allowed; revenue is split per `revenueShare` |
| `action-only` | Content is not freely accessible; only tool calls are permitted |
| `block` | Consumer must not access this resource |

***

## 4. HTTP Semantics

### 4.1 Request Headers (Consumer → Publisher)

| Header | Required | Description |
|---|---|---|
| `X-CiteFlow-Bot-Id` | Recommended | Identifies the consuming agent |
| `X-CiteFlow-Bot-Token` | Conditional | Signed token for verified bots |
| `X-CiteFlow-Intent` | Optional | `crawl`, `citation`, `tool`, `handoff` |
| `X-CiteFlow-Access-Token` | Conditional | Settlement access token |

### 4.2 Response Headers (Publisher → Consumer)

| Header | Meaning |
|---|---|
| `X-CiteFlow-Policy` | Applied PolicyMode for this resource |
| `X-CiteFlow-Citation-Limit` | Maximum free characters |
| `X-CiteFlow-Price` | Price for paid access (decimal, ISO currency) |
| `X-CiteFlow-Currency` | ISO 4217 currency code |
| `X-CiteFlow-Freshness-TTL` | Seconds until this response should be re-validated |
| `X-CiteFlow-Provenance` | SHA-256 of canonical content at time of response |
| `X-CiteFlow-Attribution` | Publisher's preferred attribution string |

### 4.3 Status Codes

| Code | Meaning in CiteFlow context |
|---|---|
| `200 OK` | Policy allows access; resource delivered |
| `402 Payment Required` | Resource requires payment; `X-CiteFlow-Price` header present |
| `403 Forbidden` | Policy is `block` for this Consumer |
| `451 Unavailable For Legal Reasons` | Resource cannot be delivered due to licensing constraints |

When returning `402`, the response body SHOULD include:

```json
{
  "citeflow": "0.1",
  "policy": "paid",
  "price": 0.002,
  "currency": "USD",
  "unit": "per-mb",
  "paymentEndpoint": "/.well-known/citeflow/pay",
  "supportedMethods": ["stripe", "lightning", "x402"]
}
```

***

## 5. Answer Unit Markup

### 5.1 HTML Attributes

Publishers MAY annotate HTML elements with CiteFlow attributes:

```html
<article
  data-cf-unit="article.summary"
  data-cf-mode="allow-with-attribution"
  data-cf-freshness-ttl="86400"
  data-cf-chars="680"
>
  ...article content...
</article>

<section
  data-cf-unit="pricing.snapshot"
  data-cf-mode="paid"
  data-cf-price="0.01"
  data-cf-currency="USD"
>
  ...pricing table...
</section>

<div
  data-cf-unit="faq.block"
  data-cf-mode="allow-with-attribution"
  data-cf-type="faq"
>
  ...FAQ content...
</div>
```

### 5.2 Answer Unit Types

| Type | Description |
|---|---|
| `article.summary` | Introductory or summary text |
| `article.full` | Complete article body |
| `faq.block` | FAQ section |
| `pricing.snapshot` | Current pricing information |
| `product.card` | Product name, description, key attributes |
| `product.availability` | In-stock / out-of-stock status |
| `dataset.table` | Structured tabular data |
| `review.aggregate` | Aggregate rating and review count |
| `how-to.steps` | Step-by-step instructions |
| `event.listing` | Date, location, registration info |

### 5.3 JSON-LD Extension

Publishers SHOULD include a `CiteFlowPolicy` node in their JSON-LD:

```json
{
  "@context": [
    "https://schema.org",
    "https://citeflow.cloud/context/v0.1"
  ],
  "@type": "WebPage",
  "citeflow:policy": {
    "@type": "citeflow:ContentPolicy",
    "citeflow:mode": "allow-with-attribution",
    "citeflow:citationLimit": 280,
    "citeflow:requiresLink": true,
    "citeflow:freshnessSeconds": 3600
  }
}
```

***

## 6. Tool Endpoint Convention

### 6.1 Discovery

Tools are listed in the manifest under `tools[]`. Consumers discover available tools by reading `/.well-known/citeflow.json`.

### 6.2 Request Format

```http
POST /api/citeflow/tools/{toolName}
Content-Type: application/json
X-CiteFlow-Bot-Id: perplexity-v2
X-CiteFlow-Bot-Token: <signed-token>

{
  "input": { ...tool-specific params... },
  "sessionId": "string — optional consumer session",
  "locale": "en-US"
}
```

### 6.3 Response Format

```json
{
  "citeflow": "0.1",
  "tool": "quote",
  "status": "success",
  "output": { "...tool-specific result..." },
  "billing": {
    "event": "tool_call",
    "amount": 0.03,
    "currency": "USD",
    "invoiceId": "string"
  },
  "attribution": {
    "label": "Acme Pricing",
    "url": "https://acme.com/pricing"
  }
}
```

### 6.4 Standard Tool Names (Recommended)

| Name | Purpose |
|---|---|
| `quote` | Return a price estimate for given parameters |
| `check_availability` | Return stock / availability status |
| `check_price` | Return current price for a SKU or service |
| `book` | Create a booking or appointment |
| `compare` | Compare two or more products / plans |
| `estimate` | Calculate an estimate (shipping, delivery, ROI) |
| `search_catalog` | Search products or services |
| `get_faq` | Return FAQ answers for a query |

***

## 7. Handoff Session

### 7.1 Purpose

A Handoff Session allows a Consumer to pass structured intent context to a Publisher when a user transitions from an AI interface to the Publisher's website. This enables Publishers to pre-fill forms, skip qualification steps, or present personalized content.

### 7.2 Creation

Consumer creates a session by calling the handoff endpoint:

```http
POST /.well-known/citeflow/handoff
Content-Type: application/json
X-CiteFlow-Bot-Id: perplexity-v2

{
  "intent": "string — human-readable user intent",
  "params": { "...extracted parameters..." },
  "locale": "en-US",
  "device": "desktop | mobile | unknown",
  "consentGiven": true,
  "stage": "awareness | consideration | decision | checkout",
  "ttlSeconds": 900
}
```

### 7.3 Response

```json
{
  "sessionId": "string — UUID",
  "token": "string — signed JWT or opaque token",
  "redirectUrl": "string — destination URL with token embedded",
  "expiresAt": "string — ISO 8601",
  "billing": {
    "event": "handoff_created",
    "amount": 0,
    "currency": "USD"
  }
}
```

### 7.4 Token Payload (if JWT)

```json
{
  "iss": "citeflow:consumer:perplexity-v2",
  "sub": "handoff",
  "aud": "acme.com",
  "exp": 1743330000,
  "iat": 1743329100,
  "jti": "uuid",
  "cf": {
    "intent": "string",
    "params": {},
    "locale": "en-US",
    "stage": "decision",
    "consentGiven": true
  }
}
```

### 7.5 Publisher Verification

Publishers MUST verify the token signature using the Consumer's published public key (discoverable via `/.well-known/citeflow/bots/{botId}.json`) before trusting the payload.

***

## 8. Settlement Events

### 8.1 Event Taxonomy

All billable interactions MUST emit a settlement event. Events are logged by the Publisher and optionally forwarded to a settlement provider.

| Event | Trigger |
|---|---|
| `crawl_requested` | Consumer fetched a page |
| `crawl_paid` | Consumer paid for a metered crawl |
| `citation_rendered` | Consumer cited content in an AI response |
| `citation_paid` | Consumer paid for above-limit citation |
| `tool_called` | Consumer called a Publisher tool |
| `tool_paid` | Tool call billed successfully |
| `handoff_created` | Consumer initiated a handoff session |
| `handoff_converted` | User completed a target action after handoff |
| `lead_qualified` | Handoff resulted in a qualified lead |
| `sale_completed` | Handoff resulted in a completed transaction |

### 8.2 Event Schema

```json
{
  "eventId": "string — UUID",
  "event": "EventType",
  "timestamp": "string — ISO 8601",
  "botId": "string",
  "siteId": "string",
  "resourceUrl": "string",
  "unitId": "string — Answer Unit id if applicable",
  "toolName": "string — if tool event",
  "sessionId": "string — if handoff event",
  "billing": {
    "amount": "number",
    "currency": "string — ISO 4217",
    "model": "per-call | per-mb | per-char | revenue-share | cpl | cpa",
    "status": "pending | settled | disputed"
  },
  "attribution": {
    "label": "string",
    "url": "string"
  }
}
```

***

## 9. Bot Identity & Verification

### 9.1 Known Bot Registry Format

Verified bots SHOULD publish their identity at a well-known URL. Publishers MAY use this to validate `X-CiteFlow-Bot-Token` headers.

Consumers publish their public key at:

```
https://{consumer-domain}/.well-known/citeflow/bot.json
```

Schema:

```json
{
  "botId": "string",
  "name": "string",
  "operator": "string",
  "publicKey": "string — PEM or JWK",
  "keyAlgorithm": "RS256 | ES256",
  "policyUrl": "string",
  "contactUrl": "string"
}
```

### 9.2 Request Signing

For signed requests, Consumer MUST include:

```
X-CiteFlow-Bot-Token: <JWT signed with bot's private key>
```

The JWT payload MUST include `iss` (botId), `aud` (Publisher hostname), `iat`, `exp`, and a `nonce`.

***

## 10. Freshness & Provenance

### 10.1 Freshness Declaration

Publishers SHOULD declare content freshness to enable Consumers to avoid serving stale answers:

```http
X-CiteFlow-Freshness-TTL: 3600
X-CiteFlow-Last-Verified: 2026-03-30T09:00:00Z
X-CiteFlow-Provenance: sha256:a3f9d1...
```

### 10.2 Provenance Hash

The `X-CiteFlow-Provenance` header MUST be a SHA-256 hash of the canonical text content of the page at the time of the response. Consumers MAY use this to detect content drift and invalidate cached answer units.

***

## 11. Conformance Levels

| Level | Requirements |
|---|---|
| **CiteFlow Basic** | Manifest at `/.well-known/citeflow.json`, default policy declared, citation policy present |
| **CiteFlow Standard** | Basic + zone policies + HTTP headers + JSON-LD extension |
| **CiteFlow Full** | Standard + tools + handoff + settlement events + bot verification |

***

## 12. Security Considerations

- All handoff tokens MUST be short-lived (recommended max `ttlSeconds: 900`).
- Publishers MUST validate bot token signatures before applying permissive policies.
- `402` responses MUST NOT include sensitive content.
- Settlement event payloads MUST be transmitted over TLS.
- Publishers MUST sanitize and validate all tool input payloads before processing.
- Consumer `botId` values SHOULD be treated as untrusted until verified.

***

## 13. Privacy Considerations

- Handoff payloads MUST NOT include raw personal data without explicit `consentGiven: true`.
- Consumers MUST NOT re-use handoff session tokens across different users.
- Publishers MUST honor user opt-out signals within handoff context.
- Settlement events SHOULD be aggregated and anonymized before sharing with third-party analytics.

***

## 14. Extensibility

Third-party extensions to this protocol MUST use a namespaced prefix, e.g. `x-myplatform-*` for custom HTTP headers and `myplatform:` for custom JSON-LD properties. Extensions MUST NOT conflict with reserved CiteFlow field names.

***

## 15. Reference Implementation

A reference implementation of the CiteFlow server adapter and manifest generator is maintained at:

`https://github.com/CiteFlow/sdk`

***

## 16. Changelog

| Version | Date | Notes |
|---|---|---|
| 0.1 | 2026-03-30 | Initial draft |

***

## Authors

CiteFlow Working Group · `spec@citeflow.cloud`

3. **Положить spec** как `spec/citeflow-protocol-v0.1.md`.

Напиши мне в следующем сообщении — и я сразу сделаю всё это через GitHub API: создам репозиторий и залью файл одним действием.
