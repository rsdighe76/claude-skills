# API Best Practice Skill — Intake Example

This is a completed intake for a fictional "Acme Payments API" to show what a filled-out template looks like. Fields marked **[auto]** are extracted from the spec — shown here for reference but you'd skip them in practice.

---

## API Overview

**OpenAPI Spec:** https://api.acmepay.com/openapi.json

**[auto] API Name:** Acme Payments API _(from `info.title`)_

**[auto] Base URLs:** _(from `servers`)_
- Production: https://api.acmepay.com
- Sandbox: https://sandbox.api.acmepay.com

**[auto] Endpoints:** _(from `paths` — 9 endpoints detected)_

---

## Authentication

**[auto] Method:** OAuth2 Bearer Token _(from `securitySchemes`)_

**[auto] Header or parameter name:** Authorization: Bearer {access_token} _(from `securitySchemes`)_

**Environment variables developers should set:**
- ACME_CLIENT_ID
- ACME_CLIENT_SECRET
- ACME_API_BASE_URL

**Credential verification endpoint:** GET /v1/identity — returns 200 with merchant profile if credentials are valid

---

## Error Format

**[auto] Error response structure:** _(from 4xx/5xx response schemas)_
```json
{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "The field 'amount' must be a positive integer.",
    "param": "amount",
    "doc_url": "https://docs.acmepay.com/errors/INVALID_PARAMETER"
  }
}
```

**Retryable errors (status codes):** 429, 500, 502, 503

**Non-retryable errors (status codes):** 400, 401, 403, 404, 409, 422

---

## Rate Limits

**Does this vary per endpoint?** Yes

**Global default:**
- Requests per minute: 100
- Burst limit: 20 requests/second
- Rate limit headers: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset
- Sends Retry-After header? Yes (on 429 responses)

**Exceptions listed in endpoint details table below.**

---

## Retry Strategy

**Does this vary per endpoint?** No

**Global:**
- Initial delay: 1 second
- Max retries: 3
- Backoff type: Exponential with jitter (delay * 2^attempt + random 0-500ms)
- Max delay: 30 seconds

---

## Timeouts

**Does this vary per endpoint?** Yes

**Global default:**
- Recommended timeout: 30 seconds

**Exceptions listed in endpoint details table below.**

---

## Pagination (if applicable)

**[auto] Pagination style:** Cursor-based _(inferred from `next_cursor` and `has_more` in response schemas)_

**Fields:**
- Next page indicator: `has_more` (boolean) and `next_cursor` (string)
- Page size parameter: `limit`
- Default page size: 20
- Max page size: 100

---

## Webhooks (if applicable)

**Signature header:** X-Acme-Signature-256

**Signature algorithm:** HMAC-SHA256

**Verification process:** Compute HMAC-SHA256 of the raw request body using your webhook secret. Compare with the value in X-Acme-Signature-256 (hex-encoded). Reject if they don't match.

**Retry policy:** 5 retries with exponential backoff over 48 hours (1min, 5min, 30min, 2hr, 24hr)

**Delivery guarantee:** At-least-once. Deduplicate using the `event_id` field.

---

## Endpoint Details

The endpoint list and required fields are auto-extracted from the spec. This table adds what the spec doesn't know: idempotency rules and per-endpoint overrides.

| Endpoint | Idempotency Required? | Idempotency Header | Idempotency Window | Specific Rate Limit | Specific Timeout | Notes |
|----------|----------------------|-------------------|-------------------|--------------------|-----------------| ------|
| POST /v1/payments | Yes | Idempotency-Key | 48 hours | 50/min | 60s | Longer timeout for bank processing |
| POST /v1/refunds | Yes | Idempotency-Key | 48 hours | 30/min | 60s | Tied to original payment_id |
| POST /v1/customers | Yes | Idempotency-Key | 24 hours | (global) | (global) | |
| GET /v1/payments/{id} | No | | | (global) | (global) | |
| GET /v1/payments | No | | | (global) | (global) | Paginated, use cursor |
| POST /v1/payouts | Yes | Idempotency-Key | 48 hours | 20/min | 90s | Slowest endpoint, bank transfers |
| PUT /v1/customers/{id} | No | | | (global) | (global) | |
| DELETE /v1/customers/{id} | No | | | 10/min | (global) | Low limit to prevent accidental bulk delete |
| POST /v1/webhooks | No | | | 5/min | (global) | Webhook endpoint registration |

---

## Context-Dependent Required Fields

| Field Name | Which Endpoints? | When Required? | Why? |
|-----------|-----------------|---------------|------|
| gdpr_consent | POST /v1/payments, POST /v1/customers | EMEA customers (determined by currency EUR, GBP, or billing country in EU/UK) | GDPR compliance — payment will be rejected without it |
| tax_id | POST /v1/payments | B2B transactions (business_type = "company") | Tax reporting requirements in EU |
| external_id | POST /v1/payments, POST /v1/refunds, POST /v1/payouts | Enterprise plan customers | Required for reconciliation with their ERP systems |
| shipping_address | POST /v1/payments | Physical goods (item_type = "physical") | Required by card network rules for physical goods |
| ip_address | POST /v1/payments | Card-not-present transactions | Fraud scoring; strongly recommended, will trigger warning if missing |

---

## Anything Else?

- The `metadata` field accepts up to 20 key-value pairs (keys max 40 chars, values max 500 chars). Useful but not validated — include as a best practice tip.
- The `statement_descriptor` field is truncated to 22 characters by card networks. Warn developers if they pass longer strings.
- Sandbox environment returns fake bank responses with a 2-second artificial delay. Production is typically faster but set timeouts based on worst case.
- The legacy `source` field on POST /v1/payments still works but is deprecated in favor of `payment_method`. Flag this as a deprecation warning.

---

**This example was used with the prompt:**

> "Create a best practice skill for the Acme Payments API using this intake template."
