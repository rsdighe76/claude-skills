---
name: api-best-practice
description: "Guide for integrating with APIs correctly and reliably. Use when: implementing retry logic and error handling for API calls, handling rate limiting and backoff strategies, implementing idempotency for safe retries, managing API authentication and credential storage, consuming webhooks securely, debugging API integration issues, reviewing API integration code for best practices."
---

# API Integration Best Practices

Guidance for consuming and integrating with APIs correctly. This covers the patterns and practices that make API integrations reliable, secure, and maintainable.

## Authentication and Credentials

Never hardcode API keys in source code. Store credentials in environment variables or a secrets manager. Use different keys for development, staging, and production environments.

```
# Good: environment variable
export API_KEY='sk_live_...'
curl -H "Authorization: Bearer $API_KEY" https://api.example.com/v1/resource

# Bad: hardcoded in script
curl -H "Authorization: Bearer sk_live_abc123..." https://api.example.com/v1/resource
```

Rotate keys periodically and immediately if compromised. Design integrations so key rotation requires only a config change, not a code deploy.

Test authentication separately before making business logic calls. A simple health check or account info endpoint confirms credentials work before attempting complex operations.

## Error Handling

Handle errors by status code category, not just success/failure.

**4xx errors** are client errors — something is wrong with the request. These should not be retried without fixing the underlying issue.
- 400 Bad Request: Invalid request format or parameters. Log the error details for debugging.
- 401 Unauthorized: Authentication failed. Check credentials, don't retry.
- 403 Forbidden: Valid credentials but insufficient permissions. Check API key scope.
- 404 Not Found: Resource doesn't exist. May be expected (check before create) or a bug.
- 422 Unprocessable Entity: Validation failed. Read the error details for which fields failed.
- 429 Too Many Requests: Rate limited. Implement backoff and retry.

**5xx errors** are server errors — something is wrong on the API side. These are typically transient and should be retried with backoff.
- 500 Internal Server Error: Retry with backoff.
- 502 Bad Gateway: Retry with backoff.
- 503 Service Unavailable: Retry with backoff, check status page.
- 504 Gateway Timeout: Retry with backoff.

Always log the full error response, including any request ID or correlation ID the API returns. This makes debugging with API support much faster.

## Retry Logic and Backoff

Retry only on transient failures: 429, 5xx, network timeouts, and connection errors. Never retry 4xx errors (except 429) — they will fail the same way every time.

Use exponential backoff with jitter to avoid thundering herd problems:

```
Base delay: 1 second
Attempt 1: wait 1s + random(0-500ms)
Attempt 2: wait 2s + random(0-500ms)
Attempt 3: wait 4s + random(0-500ms)
Attempt 4: wait 8s + random(0-500ms)
```

Respect the `Retry-After` header when present. If the API tells you to wait 30 seconds, wait 30 seconds — don't use your calculated backoff.

Set a maximum retry count (typically 3-5) and a maximum total wait time. An integration that retries forever is worse than one that fails fast.

## Rate Limiting

Know your rate limits before building the integration. Most APIs document limits as requests per minute or per second, sometimes with burst allowances.

Detect rate limiting from 429 responses and the `Retry-After` header. Some APIs also return rate limit status in headers on every response:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 23
X-RateLimit-Reset: 1620000000
```

When rate limited, back off and respect the reset time. If you're consistently hitting limits, consider:
- Batching multiple operations into single requests (if the API supports it)
- Caching responses to reduce redundant calls
- Requesting a higher rate limit from the API provider
- Spreading requests over time rather than bursting

## Idempotency

For operations that create resources or have side effects, use idempotency keys to make retries safe. An idempotency key is a unique identifier you generate for each logical operation.

```
curl -X POST https://api.example.com/v1/charges \
  -H "Authorization: Bearer $API_KEY" \
  -H "Idempotency-Key: order_12345_charge_attempt_1" \
  -d '{"amount": 1000, "currency": "usd"}'
```

If the request times out or you get a 5xx error, retry with the same idempotency key. The API will either:
- Return the original response if the first request succeeded
- Process the request if the first request never arrived

Generate idempotency keys deterministically from your business logic (order ID, transaction reference) so retries naturally use the same key.

Most APIs enforce an idempotency window (24-48 hours). After that, the same key may be treated as a new request.

## Timeouts

Set explicit timeouts on all API calls. A missing timeout means a hung connection can block your application indefinitely.

```
# Connect timeout: how long to wait for connection establishment
# Read timeout: how long to wait for data after connected

curl --connect-timeout 5 --max-time 30 https://api.example.com/v1/resource
```

Choose timeouts based on the operation:
- Simple reads: 5-10 seconds
- Writes and creates: 10-30 seconds
- Bulk operations: 30-60 seconds or more

When a timeout occurs, you often don't know if the operation succeeded. Use idempotency keys so you can safely retry.

## Webhook Consumption

Verify webhook signatures before processing. Every legitimate webhook from a well-designed API includes a signature you can validate:

```
# Typical signature verification flow:
1. Extract signature from header (e.g., X-Signature)
2. Extract timestamp from header or payload
3. Compute expected signature: HMAC-SHA256(webhook_secret, timestamp + "." + raw_body)
4. Compare computed signature to provided signature (constant-time comparison)
5. Verify timestamp is within acceptable window (e.g., 5 minutes) to prevent replay attacks
```

Respond quickly — acknowledge receipt within 5 seconds, then process asynchronously. Slow responses trigger retries, creating duplicate deliveries.

Implement idempotency for webhook processing using the event ID. Store processed event IDs and skip duplicates. Webhooks are delivered at-least-once, so duplicates are expected.

Don't trust webhook payloads for critical operations. Use the webhook as a notification, then fetch the current resource state from the API to ensure you have the latest data.

## Pagination

Never assume you've received all results. Check for pagination indicators in every list response:

```json
{
  "data": [...],
  "has_more": true,
  "next_cursor": "abc123"
}
```

Fetch all pages when you need complete data. Implement this as a loop that continues while `has_more` is true or `next_cursor` is present.

For large datasets, consider whether you actually need all records or if filtering/limiting is more appropriate.

## Debugging Integration Issues

When something isn't working:

1. **Verify authentication first** — Make a simple authenticated request to confirm credentials work.

2. **Check the exact request** — Log or print the full request: method, URL, headers, body. Compare against the API documentation.

3. **Read the full error response** — APIs return error details in the response body. Read them.

4. **Check for request IDs** — Most APIs return a request ID in the response or headers. Include this when contacting support.

5. **Check the API status page** — 5xx errors and timeouts may be a service incident, not your code.

6. **Compare with a working example** — Use curl or Postman to make the same request. If curl works but your code doesn't, the issue is in your request construction.

## Context-Dependent Required Fields

Many APIs have fields that are technically optional but effectively required for specific use cases, customer segments, or regions. Treat these as required in your integration even if the API accepts requests without them.

**By use case:**
- Payment APIs: `statement_descriptor` is optional but critical for reducing chargebacks — customers don't recognize generic descriptors on their statements
- Shipping APIs: `signature_required` is optional but should be true for high-value orders to prove delivery
- Messaging APIs: `callback_url` is optional but required if you need delivery receipts or replies

**By customer segment:**
- Enterprise integrations often require `metadata` or `external_id` fields for reconciliation with internal systems
- Multi-tenant platforms should always populate `customer_id` or `account_id` even when optional, for proper attribution and billing
- Partner integrations may require `partner_id` or `referral_code` fields to track attribution

**By region or compliance:**
- EMEA/GDPR: `data_processing_consent` or `legal_basis` fields become required for processing personal data
- PSD2/SCA: `return_url` and `payment_method_options.card.request_three_d_secure` are required for European card payments
- India: `billing_address` fields are required for card transactions per RBI guidelines
- Brazil: `tax_id` (CPF/CNPJ) is required for most payment and invoicing operations

**By payment method:**
- Bank transfers: `account_holder_name` and `bank_code` are required even if marked optional
- SEPA: `mandate_reference` and `creditor_id` are required for direct debits
- Local payment methods (iDEAL, Bancontact, etc.): country-specific fields become mandatory

When integrating, ask:
1. What use case am I implementing? Check if there are "optional but recommended" fields for that flow.
2. Who are my customers? Enterprise, SMB, and consumer integrations have different field requirements.
3. Where are my customers located? Regional regulations often mandate specific fields.
4. What payment methods am I supporting? Each method has its own required fields beyond the base API requirements.

Document these context-dependent requirements in your integration. A field that's optional in the API docs but required for your use case should be validated as required in your code.

## Common Mistakes

**Not handling partial failures in batch operations.** When creating multiple resources in one request, some may succeed and some may fail. Check the response for individual item statuses.

**Logging sensitive data.** Don't log API keys, passwords, or full request/response bodies that contain PII. Redact or mask sensitive fields.

**Ignoring deprecation warnings.** APIs return deprecation notices in headers or response bodies. Monitor for these and plan migrations.

**Assuming consistent response times.** API latency varies. Design for the slow case, not the average case.

**Not testing error paths.** Test what happens when the API returns 401, 429, 500. Simulate timeouts. Your error handling code needs to work when errors actually happen.
