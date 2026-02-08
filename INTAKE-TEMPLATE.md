# API Best Practice Skill — Intake Template

This template is **optional** — you can also just provide your OpenAPI spec and answer questions conversationally as the skill guides you. But if you prefer to gather everything upfront (especially useful for complex APIs with many endpoints), fill this out and paste it into Claude along with your spec.

Fields marked with **[auto]** are extracted automatically from your OpenAPI spec — you can skip them unless your spec is missing that info or you want to override.

---

## API Overview

**OpenAPI Spec:** _Provide URL (e.g., https://api.example.com/openapi.json) or paste the spec content below_

**[auto] API Name:** _Extracted from `info.title` in your spec_

**[auto] Base URLs:** _Extracted from `servers` in your spec_

**[auto] Endpoints:** _Extracted from `paths` in your spec_

---

## Authentication

**[auto] Method:** _Extracted from `components.securitySchemes` — e.g., OAuth2, API Key, Bearer Token_

**[auto] Header or parameter name:** _Extracted from `securitySchemes` — e.g., Authorization: Bearer {token}_

**Environment variables developers should set:**
- _e.g., ACME_API_KEY_
- _e.g., ACME_API_BASE_URL_

**Credential verification endpoint:** _e.g., GET /v1/me — returns 200 if credentials are valid_

---

## Error Format

**[auto] Error response structure:** _Extracted from 4xx/5xx response schemas if defined in your spec_

If not in your spec, paste your actual error response JSON here:
```json

```

**Retryable errors (status codes):** _e.g., 429, 500, 502, 503_

**Non-retryable errors (status codes):** _e.g., 400, 401, 403, 404, 422_

---

## Rate Limits

**Does this vary per endpoint?** _Yes / No_

**If global (same for all endpoints):**
- Requests per minute: ___
- Burst limit: ___
- Rate limit headers: _e.g., X-RateLimit-Remaining, X-RateLimit-Limit_
- Sends Retry-After header? _Yes / No_

**If per-endpoint, fill in the endpoint details table below.**

---

## Retry Strategy

**Does this vary per endpoint?** _Yes / No_

**If global:**
- Initial delay: _e.g., 1 second_
- Max retries: _e.g., 3_
- Backoff type: _e.g., exponential with jitter_
- Max delay: _e.g., 30 seconds_

**If per-endpoint, fill in the endpoint details table below.**

---

## Timeouts

**Does this vary per endpoint?** _Yes / No_

**If global:**
- Recommended timeout: _e.g., 30 seconds_

**If per-endpoint, fill in the endpoint details table below.**

---

## Pagination (if applicable)

**[auto] Pagination style:** _May be inferred from response schemas if fields like `cursor`, `next`, `offset`, or `page` are present_

**Fields:**
- Next page indicator: _e.g., next_cursor, has_more_
- Page size parameter: _e.g., limit, per_page_
- Default page size: ___
- Max page size: ___

---

## Webhooks (if applicable)

**Signature header:** _e.g., X-Webhook-Signature_

**Signature algorithm:** _e.g., HMAC-SHA256_

**Verification process:** _e.g., Compare HMAC of raw body using webhook secret_

**Retry policy:** _e.g., 3 retries over 24 hours_

**Delivery guarantee:** _e.g., at-least-once_

---

## Endpoint Details

Only fill in what the spec doesn't cover. The endpoint list and required fields are auto-extracted. Copy the block below for each endpoint you want to add details for.

### Endpoint: _METHOD /v1/your-endpoint_

- **Idempotency required?** _Yes / No_
- **Idempotency header:** _e.g., Idempotency-Key_
- **Idempotency window:** _e.g., 24 hours_
- **Specific rate limit:** _e.g., 50/min (leave blank if global applies)_
- **Specific timeout:** _e.g., 60s (leave blank if global applies)_
- **Notes:** ___

### Endpoint: _METHOD /v1/your-endpoint_

- **Idempotency required?** _Yes / No_
- **Idempotency header:** ___
- **Idempotency window:** ___
- **Specific rate limit:** ___
- **Specific timeout:** ___
- **Notes:** ___

_Copy and paste the block above for each additional endpoint._

---

## Context-Dependent Required Fields

These are fields that are optional in general but required for specific regions, customer segments, or use cases. This is **not** in your spec — only you know these rules. Copy the block below for each field.

### Field: _field_name_

- **Which endpoints?** _e.g., POST /v1/orders, POST /v1/payments_
- **When required?** _e.g., EMEA customers, B2B transactions, Enterprise plan_
- **Why?** _e.g., GDPR compliance, tax reporting_

### Field: _field_name_

- **Which endpoints?** ___
- **When required?** ___
- **Why?** ___

_Copy and paste the block above for each additional field._

---

## Anything Else?

_Note any additional best practices, known gotchas, deprecated fields, or migration notes that should be included in the skill._

---

**Once complete:** Paste this filled-out template into Claude along with your OpenAPI spec and say:

> "Create a best practice skill for my API using this intake template."
