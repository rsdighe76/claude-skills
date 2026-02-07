---
name: api-best-practice-skill-creator
description: "Create API best practice documentation tailored to a specific API. Use when: building integration best practices for a specific API, documenting error handling and retry patterns for an API, creating context-dependent field requirements for specific use cases/regions/segments, writing rate limiting and idempotency guidance for an API. This generates best practice content — not general advice."
---

# API Best Practice Skill Creator

Create best practice documentation tailored to your specific API. This guide helps PMs, Solution Engineers, Architects, Developer Evangelist and Developer Relations teams or other such folks build comprehensive best practice skills that ensure developers integrate correctly from the start or use it as a validator upon integration. 

## What This Skill Creates

A best practice skill for your API covering:

**Authentication patterns** — How to securely store and use credentials for your API, environment-specific configuration, and credential testing.

**Context-dependent required fields** — Fields that are optional in your API but required for specific use cases, customer segments, or regions.

**Error handling** — Your API's error format, status codes, and appropriate handling for each. Which errors are retryable, which require user intervention.

**Retry and backoff** — Specific retry strategies for your API's transient errors, respecting your rate limits and Retry-After headers.

**Rate limiting** — Your API's rate limits, how to detect them, and strategies for staying within limits.

**Idempotency** — How your API handles idempotency keys, which operations need them, and your idempotency window.

**Timeouts** — Recommended timeout values for different operation types on your API.

**Webhook consumption** — Your webhook signature format, verification process, and delivery guarantees.

**Pagination** — Your API's pagination format and how to iterate through all results.

## Implementation Process

### Core Principle: Authoritative, Deterministic Best Practices

**The skill you create will be the single source of truth for your API's best practices.**

This means:
- ✓ The generated skill uses ONLY information you provide (OpenAPI spec + your input)
- ✓ All validation rules are explicitly defined by you
- ✓ The skill is deterministic - same input always produces same validation results
- ✗ The skill will NOT search the web or external docs
- ✗ The skill will NOT make assumptions about your API
- ✗ The skill will NOT hallucinate requirements

**Why this matters:** Best practices are often proprietary, undocumented publicly, or vary by company. Your input is the authoritative source, and the generated skill must only reference what you've explicitly provided.

### Step 1: Provide OpenAPI/Swagger Specification

Before proceeding, you must provide the OpenAPI or Swagger specification for your API. This is required — do not continue without it.

Accepted formats:
- OpenAPI 3.x JSON or YAML file
- Swagger 2.0 JSON or YAML file
- URL to a hosted spec (e.g., `https://api.example.com/openapi.json`)

If you don't have a spec, create one first or use a tool like Swagger Editor to generate it from your API.

The spec provides the foundation for understanding your API's endpoints, request/response schemas, and authentication requirements.

### Step 2: Extract Information from OpenAPI Spec

Parse the OpenAPI specification to automatically extract:

**From `components.securitySchemes`:**
- Authentication methods (HTTP Bearer, API Key, OAuth2, OpenID Connect)
- Header/query parameter names for API keys
- OAuth2 flows and endpoints

**From `paths` and operations:**
- Available endpoints and HTTP methods
- Request/response schemas
- Status codes returned by each endpoint
- Required vs optional parameters

**From response schemas:**
- Error response format (if defined in 4xx/5xx responses)
- Pagination patterns (cursor, offset, page number fields in responses)

**From `servers`:**
- Base URLs for different environments (production, sandbox)

### Step 3: Gather Missing API-Specific Details

With the OpenAPI spec parsed, collect additional details that are typically not in specifications:

**Authentication (if not fully defined in spec):**
- ✓ _Authentication method_ — Auto-extracted from `securitySchemes` if present
- ✓ _Header/parameter names_ — Auto-extracted from `securitySchemes` if present
- ✗ What environment variables should developers set? (e.g., `YOUR_API_KEY`, `API_BASE_URL`)
- ✗ What endpoint can developers call to verify credentials work? (e.g., `/v1/me`, `/v1/health`)
- ✗ What does an auth failure response look like? (specific example with error codes)

**Errors:**
- ✓ _Error response format_ — Auto-extracted from 4xx/5xx response schemas if defined
- ✓ _Status codes_ — Auto-extracted from operation responses if defined
- ✗ Which errors are retryable? Which indicate a permanent failure?
- ✗ What does each error code mean in practice? (business context, not just HTTP status)

**Rate limits:**
- ✗ What are your per-minute and burst limits?
- ✗ What headers indicate rate limit status? (e.g., `X-RateLimit-Remaining`)
- ✗ What does a 429 response look like? (specific example)
- ✗ Does your API send `Retry-After` headers?

**Idempotency:**
- ✗ Does your API support idempotency keys?
- ✗ Which operations require them?
- ✗ What is the idempotency window? (e.g., 24 hours)
- ✗ What header or parameter carries the key? (e.g., `Idempotency-Key` header)

**Webhooks:**
- ✗ What signature algorithm do you use? (e.g., HMAC-SHA256)
- ✗ What header contains the signature? (e.g., `X-Webhook-Signature`)
- ✗ What is the expected response time? (e.g., respond within 5 seconds)
- ✗ What is your retry policy? (e.g., 3 retries with exponential backoff)

**Pagination:**
- ✓ _Pagination style_ — Infer from response schemas if fields like `cursor`, `next`, `offset`, or `page` are present
- ✗ What fields indicate more results exist? (e.g., `has_more`, `next_cursor`)
- ✗ What is the maximum page size?

**Timeouts:**
- ✗ What are recommended timeout values for different operation types? (e.g., 30s for reads, 60s for writes)

### Step 3: Document Context-Dependent Fields

Identify optional fields that become required in specific contexts. Interview your support team and review common integration issues.

**By use case:** What operations does your API support? For each, what optional fields are strongly recommended?

**By customer segment:** Do enterprise customers need specific fields for compliance or reconciliation? Do multi-tenant platforms need attribution fields?

**By region:** What regional regulations affect your API? GDPR in EMEA? RBI in India? LGPD in Brazil? What fields do these require?

**By operation variant:** If your API operations support multiple variants or types (e.g., payment methods, shipping types, communication channels, content formats), what fields does each variant require?

### Step 4: Write the Best Practice Skill

Structure the SKILL.md for your API's best practice skill.

**Critical Requirements:**
1. The skill must require users to provide their implementation for comparison
2. **The skill must ONLY use information provided by you (the skill creator)** - it should NOT:
   - Search the web for best practices
   - Pull from external documentation
   - Make assumptions about your API
   - Hallucinate requirements
3. The skill must be **deterministic and authoritative** - based solely on the OpenAPI spec and details you provide
4. If the skill needs additional information not provided during creation, it should ask you (the skill creator) first, NOT search externally

```
---
name: [your-api]-best-practices
description: "Best practices for integrating with [Your API]. Use when: implementing [Your API] integration, handling [Your API] errors, setting up [Your API] webhooks, troubleshooting [Your API] issues."
---

# [Your API] Integration Best Practices

## How to Use This Skill

**Before proceeding, you MUST provide one of the following:**

1. **Copy/paste your API request** in the format your code uses (curl, HTTP, language-specific client)
2. **Upload a file** containing your API integration code
3. **Copy/paste the code** that generates your API request

This allows the skill to compare your implementation against best practices and identify specific issues.

**Important Constraints:**
- This skill uses **ONLY the authoritative best practices defined below** - it will NOT search the web or external documentation
- All validation rules come from the official API specification and internal best practices documented in this skill
- If you need information not covered in this skill, the skill will ask for your permission before looking elsewhere
- This ensures accuracy and prevents incorrect or outdated external information from being used

**Example formats accepted:**

```bash
# Curl format
curl -X POST "https://api.example.com/v1/orders" \
  -H "Authorization: Bearer sk_test_123" \
  -d '{"amount": 1000}'
```

```python
# Python code
import requests

response = requests.post(
    "https://api.example.com/v1/orders",
    headers={"Authorization": f"Bearer {api_key}"},
    json={"amount": 1000}
)
```

```javascript
// JavaScript/Node.js code
fetch('https://api.example.com/v1/orders', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ amount: 1000 })
});
```

Without your actual implementation, this skill can only provide general guidance. With it, the skill will analyze your code and provide specific feedback:

**What the skill will check:**
- ✗ **Missing authentication headers** → Show exactly what header/value to add
- ✗ **Missing required or context-dependent fields** → Identify which fields are missing for your use case
- ✗ **Incorrect error handling** → Point out which status codes aren't handled and how to handle them
- ✗ **Missing retry logic** → Show which errors should be retried and the backoff strategy to use
- ✗ **Missing idempotency keys** → Identify operations that need idempotency keys
- ✗ **Rate limiting issues** → Flag missing rate limit detection/handling
- ✗ **Incorrect timeout values** → Recommend appropriate timeout values for your operations

**Output format:**
For each issue found, the skill will provide:
1. **What's wrong** - Specific issue in your implementation
2. **Why it matters** - Consequence of not fixing it
3. **How to fix** - Exact change needed (code snippet or specific value)

**Important:** These are recommendations, not requirements. You may have valid reasons to deviate (e.g., handling at a different layer, testing environment, architectural constraints). The skill provides information so you can make informed decisions. If you choose to ignore a recommendation, that's acceptable - just ensure you understand the implications.

---

## Analysis Process

**When user provides their code, follow this process:**

1. **Parse the user's code** - Identify endpoints called, HTTP methods, headers, error handling, etc.
2. **Run validation checklist** - Go through EVERY ☐ check below systematically
3. **For each check:**
   - Determine if the code violates the rule
   - If YES → Document the violation using ❌ Issue/Why/Fix format
   - If NO → Continue to next check
4. **Report ALL violations** - List every issue found, don't skip or summarize
5. **Provide summary** - At the end, summarize total violations by category

**Do NOT skip validation checks even if you find many violations. Users need to see ALL issues.**

**If user says they want to ignore a recommendation:**
- Acknowledge their decision
- Optionally ask if they want you to document the reasoning (for team context)
- Do NOT repeatedly warn about the same issue
- Move on to analyzing other aspects of their code

## Validation Rules

**EXECUTION REQUIREMENTS:**
1. **You MUST run ALL validation checks listed below** - do not skip any checks
2. **For EACH check, analyze the user's code** and determine if it violates the rule
3. **Report ALL violations found** - even if there are many issues
4. **Use the exact "Issue/Why/Fix" format** shown below for each violation

**IMPORTANT CONSTRAINTS:**
- Only validate against the rules explicitly defined below
- If user code involves endpoints/fields not documented here, inform the user that validation is limited to documented best practices
- DO NOT search the web, make assumptions, or provide generic advice

**IMPORTANT PHILOSOPHY:**
- These are **recommendations**, not hard requirements
- Report all findings, but respect user decisions to deviate
- Users may have valid architectural reasons to ignore recommendations
- If user acknowledges a finding and chooses to proceed, respect that decision
- Focus on providing information so users can make informed choices

---

### MANDATORY VALIDATION CHECKLIST

When analyzing user code, you MUST check ALL of the following. For each violation found, report it using the format: ❌ Issue / Why / Fix

### ☐ Authentication (REQUIRED CHECK)

**MUST check for ALL of the following:**
- ✗ Missing Authorization header → **Issue found**: Add `Authorization: Bearer {api_key}`
- ✗ Hardcoded credentials in code → **Issue found**: Move to environment variables: `$YOUR_API_KEY`
- ✗ Using test keys in production → **Issue found**: Use production keys from environment
- ✗ No auth error handling (401/403) → **Issue found**: Add error handling for auth failures

**Example finding report:**
```
⚠️  Issue: Missing Authorization header
Why: API requests will fail with 401 Unauthorized
Recommendation: Add this header to your request:
  headers: {"Authorization": f"Bearer {YOUR_API_KEY}"}

(If you're handling auth elsewhere or have a specific reason for this, you can acknowledge and proceed)
```

### ☐ Error Handling (REQUIRED CHECK)

**MUST check for ALL of the following:**
- ✗ Not checking response status codes → **Issue found**: Should check status before parsing
- ✗ Missing retryable error handling (5xx, 429) → **Issue found**: Should handle retryable errors
- ✗ Missing non-retryable error handling (4xx) → **Issue found**: Should handle client errors
- ✗ Not parsing error response format → **Issue found**: Should parse API error format

For each issue, specify the exact status code, whether it's retryable, and the API's error response format.

### ☐ Retry Logic (REQUIRED CHECK)

**MUST check for ALL of the following:**
- ✗ No retry logic for transient errors (5xx, 429, timeouts) → **Issue found**: Recommend implementing retry
- ✗ Missing exponential backoff → **Issue found**: Recommend exponential backoff (not fixed delay)
- ✗ Not respecting Retry-After headers → **Issue found**: Should check and respect Retry-After
- ✗ Retrying non-retryable errors (4xx) → **Issue found**: Should not retry client errors

For each issue, provide the specific retry strategy for this API.

### ☐ Rate Limiting (REQUIRED CHECK)

**MUST check for ALL of the following:**
- ✗ No rate limit detection (checking for 429) → **Issue found**: Should detect 429 status
- ✗ Not reading rate limit headers → **Issue found**: Should read X-RateLimit-Remaining (or API-specific headers)
- ✗ Missing backoff when rate limited → **Issue found**: Should implement backoff strategy
- ✗ Not respecting Retry-After header → **Issue found**: Should wait for Retry-After duration

For each issue, specify this API's rate limits and header names.

### ☐ Idempotency (REQUIRED CHECK)

**If user code makes POST/PATCH requests to endpoints that require idempotency, you MUST check:**
- ✗ Missing idempotency key on required operations → **Issue found**: Should include idempotency key header
- ✗ Reusing idempotency keys across different requests → **Issue found**: Should generate unique keys per operation
- ✗ Not generating unique keys properly → **Issue found**: Should use proper unique ID generation (UUID, etc.)

**First, identify which endpoints require idempotency (list them explicitly in the skill).** Then check if user code calls those endpoints. If yes, validate idempotency key usage.

For each issue, specify which operations require keys, the header name, and the idempotency window.

### ☐ Timeouts (REQUIRED CHECK)

**MUST check for ALL of the following:**
- ✗ No timeout set → **Issue found**: Should set timeout (code may hang indefinitely otherwise)
- ✗ Timeout too short for operation → **Issue found**: Should use appropriate timeout (specify recommended values)
- ✗ Same timeout for all operations → **Issue found**: Consider different timeouts by operation type

For each issue, provide recommended timeout values by operation type for this API.

### ☐ Webhooks (REQUIRED CHECK - if user code receives webhooks)

**If user code implements webhook handlers, MUST check:**
- ✗ Not verifying webhook signatures → **Issue found**: Recommend verifying signatures for security
- ✗ Not responding within expected time → **Issue found**: Should respond within X seconds
- ✗ Not handling retries properly → **Issue found**: Should implement proper retry handling

### ☐ Pagination (REQUIRED CHECK - if user code fetches list endpoints)

**If user code calls paginated endpoints, MUST check:**
- ✗ Only fetching first page → **Issue found**: Should iterate through all pages
- ✗ Not checking for more results → **Issue found**: Should check has_more/next_cursor field
- ✗ Incorrect pagination parameter usage → **Issue found**: Should use correct pagination fields

### ☐ Required Fields by Context (REQUIRED CHECK)

**MUST check if user code is missing optional-but-required fields based on:**

**By use case:**
- Identify what operation user is implementing
- Check if optional fields are required for that use case
- **Issue found**: If required field missing, report it with context

**By customer segment (enterprise, SMB):**
- Look for indicators of customer segment in code/context
- Check if segment-specific fields are included
- **Issue found**: If required field missing, report it with reasoning

**By region (EMEA, India, Brazil):**
- Look for region indicators (customer data, addresses, etc.)
- Check if region-specific fields are included
- **Issue found**: If required field missing, report it (e.g., GDPR consent for EMEA)

**By operation variant (payment method, shipping type, etc.):**
- Identify which variant is being used
- Check if variant-specific fields are included
- **Issue found**: If required field missing, report it with variant context

For each missing field, explain what context makes it required, potential consequences, and how to include it. Users may have valid reasons for different implementations.

### Common Mistakes
[List your API's most common integration mistakes with specific detection rules]
```

### Step 5: Write Analysis Logic

For each best practice section, document how to analyze user code:

**Pattern matching approach:**
- Look for specific patterns in the code (header names, error handling blocks, retry loops)
- Check for presence/absence of required elements
- Validate values against your API's requirements

**Feedback format:**
For every issue found, provide:
```
⚠️  Issue: [Specific problem in their code]
Why: [Consequence - what might break or go wrong]
Recommendation: [Suggested code change]

Before:
[Their current code snippet]

After:
[Recommended code snippet]

Note: [Acknowledge user may have valid reasons for different approach]
```

### Step 6: Identify Common Mistakes and Detection Patterns

Ask your support team:
- What questions do they answer repeatedly?
- What mistakes do new integrators make?
- What's hard to find in current documentation?

For each common mistake, document:
1. **How to detect it** - What pattern to look for in user code
2. **What's wrong** - Why it's a problem
3. **How to fix** - Specific code change needed

**Example:**
```
Common Mistake: Using sandbox credentials in production

Detection: Look for test/sandbox key patterns (e.g., sk_test_, sandbox.api.example.com)
Issue: Test credentials don't work in production
Fix: Replace with production credentials from environment variables
```

## Testing

Before distributing the skill:

1. Have someone unfamiliar with your API read the best practices
2. Ask them to implement a basic integration using only the skill
3. Note where they get stuck or confused
4. Update the skill to address those gaps

## Example: Validation-Focused Best Practice Skill

```
---
name: acme-api-best-practices
description: "Validate Acme API integration against best practices. Use when: reviewing Acme API code, troubleshooting integration issues, validating implementation."
---

# Acme API Integration Validator

Provide your API request or code implementation, and this skill will check it against Acme API best practices.

**Authoritative Source:** This skill uses ONLY the official Acme API best practices documented below. It will NOT search external sources or make assumptions. All validation rules come from the Acme API specification and internal engineering best practices.

## Analysis Process

**When you provide your code, this skill will:**
1. Parse your code to identify endpoints, methods, headers, error handling
2. Run through EVERY validation check below (☐ checkboxes)
3. For each check, determine if your code violates the rule
4. Report ALL violations found using ❌ Issue/Why/Fix format
5. Provide a summary of total violations by category

**This skill will report ALL issues found, not just a subset.**

## Validation Rules

**EXECUTION REQUIREMENTS:**
1. **Run ALL validation checks below** - do not skip any
2. **Report ALL violations found** - use ❌ Issue/Why/Fix format
3. **Only validate against rules documented here** - no web searches or assumptions

---

### ☐ Authentication (REQUIRED CHECK)

**MUST check for ALL of the following:**
- ✗ Missing `Authorization: Bearer` header → **Issue found**
  - Recommendation: Add `Authorization: Bearer $ACME_API_KEY` header
- ✗ Hardcoded API keys in code → **Issue found**
  - Recommendation: Move to environment variable: `ACME_API_KEY`
- ✗ Test keys (sk_test_) in production code → **Issue found**
  - Recommendation: Use production key (sk_live_)
- ✗ No 401/403 error handling → **Issue found**
  - Recommendation: Catch auth errors and refresh/check credentials

### ☐ Error Handling (REQUIRED CHECK)

**Acme error format:** `{"error": {"code": "...", "message": "...", "param": "..."}}`

**MUST check for ALL of the following:**
- ✗ Not checking response status before parsing → **Issue found**
  - Recommendation: Check status code first: `if response.status_code != 200:`
- ✗ Not handling 400 (bad request) → **Issue found**
  - Recommendation: Parse `error.param` to identify bad field
- ✗ Not handling 429 (rate limit) → **Issue found**
  - Recommendation: Check for 429, read `Retry-After` header, wait and retry
- ✗ Not retrying 5xx errors → **Issue found**
  - Recommendation: Retry with exponential backoff (1s, 2s, 4s, 8s)

### ☐ Rate Limiting (REQUIRED CHECK)

**Limits:** 100 requests/minute, 10 requests/second burst

**MUST check for ALL of the following:**
- ✗ No rate limit detection → **Issue found**
  - Recommendation: Check for `response.status_code == 429`
- ✗ Not reading `Retry-After` header → **Issue found**
  - Recommendation: `wait_time = int(response.headers.get('Retry-After', 60))`
- ✗ Immediate retry after 429 → **Issue found**
  - Recommendation: Wait `Retry-After` seconds before retrying

### ☐ Idempotency (REQUIRED CHECK)

**Idempotency is recommended for:** POST requests to `/v1/orders`, `/v1/payments`
**Header:** `Idempotency-Key`
**Window:** 24 hours

**If user code makes POST requests to `/v1/orders` or `/v1/payments`, you MUST check:**
- ✗ Missing `Idempotency-Key` header → **Issue found**
  - Recommendation: Add header with unique key: `Idempotency-Key: order_{uuid}`
  - Why: Prevents duplicate orders if network fails and request is retried
- ✗ Reusing same key for different requests → **Issue found**
  - Recommendation: Generate unique key per logical operation
- ✗ Not generating unique keys properly → **Issue found**
  - Recommendation: Use UUID or similar: `import uuid; key = str(uuid.uuid4())`

### Required Fields by Context

**EMEA customers:**
- ✗ Missing `gdpr_consent` field on customer creation
  - Fix: Add `"gdpr_consent": true` to request body
  - Why: Required for GDPR compliance

**Enterprise customers:**
- ✗ Missing `external_id` field
  - Fix: Add `"external_id": "{your_system_id}"` to request body
  - Why: Required for reconciliation and support

## Analysis Output Format

For each issue found:
```
⚠️  Issue: Missing Idempotency-Key header on POST /v1/orders
Why: Duplicate requests may create multiple orders if network fails and the request is retried
Recommendation: Add this header:

Before:
requests.post(f"{base_url}/v1/orders", headers={"Authorization": f"Bearer {key}"})

After:
requests.post(
    f"{base_url}/v1/orders",
    headers={
        "Authorization": f"Bearer {key}",
        "Idempotency-Key": f"order_{order_id}"
    }
)

Note: If you're handling idempotency at a different layer (load balancer, API gateway, etc.) or have a specific reason for this implementation, you can acknowledge and proceed.
```
```
