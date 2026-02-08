# API Best Practice Skill — Intake Template

Fill out this template before using the skill creator. Paste the completed version into Claude along with your OpenAPI/Swagger spec.

---

## API Overview

**API Name:** _e.g., Acme Payments API_

**Base URLs:**
- Production: _e.g., https://api.acme.com_
- Sandbox: _e.g., https://sandbox.api.acme.com_

**OpenAPI Spec:** _Attach your OpenAPI 3.x or Swagger 2.0 file (JSON or YAML)_

---

## Authentication

**Method:** _e.g., OAuth2 Bearer Token / API Key in header / Basic Auth_

**Header or parameter name:** _e.g., Authorization: Bearer {token}_

**Environment variables developers should set:**
- _e.g., ACME_API_KEY_
- _e.g., ACME_API_BASE_URL_

**Credential verification endpoint:** _e.g., GET /v1/me — returns 200 if credentials are valid_

---

## Error Format

**Standard error response structure:**
```json

```
_Paste your actual error response JSON here_

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

**Pagination style:** _e.g., cursor-based / offset-based / page number_

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

Fill in one row per endpoint. Leave blank if the global setting applies.

| Endpoint | Idempotency Required? | Idempotency Header | Idempotency Window | Specific Rate Limit | Specific Timeout | Notes |
|----------|----------------------|-------------------|-------------------|--------------------|-----------------| ------|
| _POST /v1/..._ | _Yes / No_ | _e.g., Idempotency-Key_ | _e.g., 24 hours_ | _e.g., 50/min_ | _e.g., 60s_ | |
| _POST /v1/..._ | _Yes / No_ | | | | | |
| _GET /v1/..._ | _No_ | | | | | |
| _PUT /v1/..._ | _Yes / No_ | | | | | |
| _DELETE /v1/..._ | _Yes / No_ | | | | | |

_Add more rows as needed._

---

## Context-Dependent Required Fields

These are fields that are optional in general but required for specific regions, customer segments, or use cases.

| Field Name | Which Endpoints? | When Required? | Why? |
|-----------|-----------------|---------------|------|
| _e.g., gdpr_consent_ | _POST /v1/orders_ | _EMEA customers_ | _GDPR compliance_ |
| _e.g., tax_id_ | _POST /v1/invoices_ | _B2B transactions_ | _Tax reporting_ |
| _e.g., external_id_ | _All POST endpoints_ | _Enterprise customers_ | _Reconciliation_ |
| | | | |

_Add more rows as needed._

---

## Anything Else?

_Note any additional best practices, known gotchas, deprecated fields, or migration notes that should be included in the skill._

---

**Once complete:** Paste this filled-out template into Claude along with your OpenAPI spec and say:

> "Create a best practice skill for my API using this intake template."
