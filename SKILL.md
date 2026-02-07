---
name: api-best-practice-skill-creator
description: "Create API best practice documentation tailored to a specific API. Use when: building integration best practices for a specific API, documenting error handling and retry patterns for an API, creating context-dependent field requirements for specific use cases/regions/segments, writing rate limiting and idempotency guidance for an API. This generates best practice content — not general advice."
---

# API Best Practice Skill Creator

Create best practice documentation tailored to your specific API. This guide helps PMs and developer relations teams build comprehensive best practice skills that ensure developers integrate correctly from the start.

## What This Skill Creates

A best practice skill for your API covering:

**Authentication patterns** — How to securely store and use credentials for your API, environment-specific configuration, and credential testing.

**Error handling** — Your API's error format, status codes, and appropriate handling for each. Which errors are retryable, which require user intervention.

**Retry and backoff** — Specific retry strategies for your API's transient errors, respecting your rate limits and Retry-After headers.

**Rate limiting** — Your API's rate limits, how to detect them, and strategies for staying within limits.

**Idempotency** — How your API handles idempotency keys, which operations need them, and your idempotency window.

**Timeouts** — Recommended timeout values for different operation types on your API.

**Webhook consumption** — Your webhook signature format, verification process, and delivery guarantees.

**Pagination** — Your API's pagination format and how to iterate through all results.

**Context-dependent required fields** — Fields that are optional in your API but required for specific use cases, customer segments, or regions.

## Implementation Process

### Gather API-Specific Details

For each section, collect the specifics from your API documentation and support team:

**Authentication:**
- What authentication method does your API use? (Bearer token, API key header, OAuth)
- What environment variables should developers set?
- What endpoint can developers call to verify credentials work?
- What does an auth failure response look like?

**Errors:**
- What is your API's error response format?
- List each status code your API returns and what it means
- Which errors are retryable? Which indicate a permanent failure?

**Rate limits:**
- What are your per-minute and burst limits?
- What headers indicate rate limit status?
- What does a 429 response look like?

**Idempotency:**
- Does your API support idempotency keys?
- Which operations require them?
- What is the idempotency window?
- What header or parameter carries the key?

**Webhooks:**
- What signature algorithm do you use?
- What header contains the signature?
- What is the expected response time?
- What is your retry policy?

**Pagination:**
- What pagination style do you use? (cursor, offset, page number)
- What fields indicate more results exist?
- What is the maximum page size?

### Document Context-Dependent Fields

Identify optional fields that become required in specific contexts. Interview your support team and review common integration issues.

**By use case:** What operations does your API support? For each, what optional fields are strongly recommended?

**By customer segment:** Do enterprise customers need specific fields for compliance or reconciliation? Do multi-tenant platforms need attribution fields?

**By region:** What regional regulations affect your API? GDPR in EMEA? RBI in India? LGPD in Brazil? What fields do these require?

**By payment/transaction method:** If your API supports multiple methods, what fields does each require?

### Write the Best Practice Skill

Structure the SKILL.md for your API's best practice skill:

```
---
name: [your-api]-best-practices
description: "Best practices for integrating with [Your API]. Use when: implementing [Your API] integration, handling [Your API] errors, setting up [Your API] webhooks, troubleshooting [Your API] issues."
---

# [Your API] Integration Best Practices

## Authentication
[Your API's specific auth pattern with curl example]

## Error Handling
[Your API's error format and handling for each status code]

## Retry Logic
[Which of your errors are retryable, your specific backoff recommendations]

## Rate Limiting
[Your specific limits and how to detect/handle them]

## Idempotency
[Your idempotency key format, required operations, window]

## Timeouts
[Recommended timeouts for your API's operations]

## Webhooks
[Your signature format, verification example, delivery guarantees]

## Pagination
[Your pagination format with example]

## Required Fields by Context
[Your API's optional-but-required fields organized by use case, segment, region]

## Common Mistakes
[Top integration mistakes specific to your API]
```

### Provide Curl Examples

For each section, include a curl example showing the correct pattern for your API:

```
# Authentication test
curl -X GET "https://api.yourcompany.com/v1/health" \
  -H "Authorization: Bearer $YOUR_API_KEY"

# Expected success response
{"status": "ok", "environment": "sandbox"}

# Auth failure response
{"error": {"code": "invalid_api_key", "message": "..."}}
```

Show both success and failure cases so developers know what to expect.

### Identify Common Mistakes

Ask your support team:
- What questions do they answer repeatedly?
- What mistakes do new integrators make?
- What's hard to find in current documentation?

Document each as: mistake, consequence, correct approach.

## Testing

Before distributing the skill:

1. Have someone unfamiliar with your API read the best practices
2. Ask them to implement a basic integration using only the skill
3. Note where they get stuck or confused
4. Update the skill to address those gaps

## Example: Minimal Best Practice Skill

```
---
name: acme-api-best-practices
description: "Best practices for Acme API integration. Use when: implementing Acme API, handling Acme errors, setting up Acme webhooks."
---

# Acme API Best Practices

## Authentication

Store credentials in environment variables:

export ACME_API_KEY='sk_live_...'
export ACME_BASE_URL='https://api.acme.com'

Test authentication:

curl -X GET "$ACME_BASE_URL/v1/me" \
  -H "Authorization: Bearer $ACME_API_KEY"

## Error Handling

Acme returns errors as:

{"error": {"code": "...", "message": "...", "param": "..."}}

- 400: Bad request. Check the `param` field for which parameter failed.
- 401: Invalid API key. Verify credentials.
- 429: Rate limited. Wait for Retry-After seconds.
- 5xx: Retry with exponential backoff.

## Rate Limiting

Limits: 100 requests/minute, 10 requests/second burst.

When rate limited, the response includes:

Retry-After: 30

Wait that many seconds before retrying.

## Idempotency

For POST requests, include:

curl -X POST "$ACME_BASE_URL/v1/orders" \
  -H "Authorization: Bearer $ACME_API_KEY" \
  -H "Idempotency-Key: order_12345" \
  -d '{"amount": 1000}'

Idempotency window: 24 hours.

## Required Fields by Context

**EMEA customers:** Include `gdpr_consent: true` on customer creation.

**Enterprise:** Include `external_id` for reconciliation.
```
