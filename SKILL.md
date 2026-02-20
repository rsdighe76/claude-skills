---
name: api-best-practice-skill-creator
description: "Create or update API best practice documentation from an OpenAPI spec. Generates per-endpoint validation skills for authentication, errors, retries, rate limits, and idempotency."
---

# API Best Practice Skill Creator

Create or update best practice documentation tailored to your specific API. This guide helps PMs, Solution Engineers, Architects, Developer Evangelist and Developer Relations teams or other such folks build comprehensive best practice skills that ensure developers integrate correctly from the start or use it as a validator upon integration. 

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
- ✗ The generated skill will NOT search the web or external docs at validation time
- ✗ The skill will NOT make assumptions about your API
- ✗ The skill will NOT hallucinate requirements

**Note:** During intake template generation (Step 1b), this creator skill may search public developer docs to pre-fill suggestions — but these are always marked **[web]** for your review and the final generated skill never searches the web.

**Why this matters:** Best practices are often proprietary, undocumented publicly, or vary by company. Your input is the authoritative source, and the generated skill must only reference what you've explicitly provided.

### Step 1: Provide Your API Details

Before proceeding, you need at minimum an OpenAPI or Swagger specification for your API. There are three ways to get started:

**Option A: Generate a customized intake template from your spec (recommended)**

Provide your OpenAPI spec and ask: _"Generate an intake template for my API."_ The skill will parse your spec and produce a **customized template** with your actual API name, base URLs, auth method, error format, and every endpoint pre-listed as blocks ready to fill in. You then only need to add what the spec doesn't know (rate limits, idempotency rules, context-dependent fields, etc.), paste the completed template back, and the skill generates your best practice files.

This is the easiest path — you see your real endpoints listed out and just fill in the blanks.

**Option B: Fill out the generic intake template manually**

If you'd prefer to work offline, fill out the [Intake Template](INTAKE-TEMPLATE.md) with your API details and paste it along with your spec. See the [Intake Example](INTAKE-EXAMPLE.md) for a completed version.

**Option C: Just provide the spec — I'll ask questions as we go**

Provide your OpenAPI spec and I'll walk you through the remaining details step by step, asking about rate limits, idempotency, context-dependent fields, etc. as needed. No prep required.

**Accepted spec formats:**
- OpenAPI 3.x JSON or YAML
- Swagger 2.0 JSON or YAML
- URL to a hosted spec (e.g., `https://api.example.com/openapi.json`)
- Pasted spec content

If you don't have a spec, create one first or use a tool like Swagger Editor to generate it from your API.

**If the user provides a completed intake template** (Option A or B), skip to Step 5 (Create the Skill Files) — all the information from Steps 2-4 is already in the template.

**If the user asks to generate a customized intake template** (Option A), proceed to Step 1b below.

**If the user provides just the spec with no template** (Option C), proceed to Step 2 below.

---

### Step 1b: Generate a Customized Intake Template

**When the user provides a spec and asks for a customized intake template:**

**Phase 1: Extract from the OpenAPI spec**

Parse the spec to extract:
- API name from `info.title`
- Base URLs from `servers`
- Auth method from `components.securitySchemes`
- Error format from 4xx/5xx response schemas
- Pagination style from response schemas
- All endpoints from `paths`

**Phase 2: Enrich from public documentation (if available)**

Search the web for the API's public developer documentation to pre-fill additional details. Look for:
- Rate limit documentation (requests per minute, burst limits, rate limit headers)
- Retry and backoff guidance
- Idempotency requirements and key formats
- Timeout recommendations
- Webhook setup and signature verification docs
- Known deprecated fields or migration guides
- Error code reference pages

**IMPORTANT:** Any information gathered from the web is a **suggestion only**. Mark web-sourced fields with **[web]** so the user knows to review and confirm or correct them. The user is the authoritative source — web info is a starting point to reduce manual effort.

**Phase 3: Generate the customized template**

Produce a template with:
- **[auto]** fields pre-filled from the spec
- **[web]** fields pre-filled from public docs (clearly marked for user review)
- **Every endpoint** listed as a separate block with the actual path pre-filled
- **Blank fields** for anything not found in the spec or public docs

**Phase 4: Present for review**

Present the customized template and say:
```
Here's your customized intake template based on your OpenAPI spec
and public documentation.

Fields marked [auto] were extracted from your spec.
Fields marked [web] were found in your public docs — please review
and correct these, as your internal requirements may differ.

Please fill in any remaining blank fields:
- Rate limits (if not found in docs)
- Idempotency details per endpoint
- Context-dependent required fields (only you know these)
- Any internal gotchas or undocumented best practices

Once complete, paste it back and I'll generate your best practice skill.
```

**When the user returns the completed template**, skip to Step 5 (Create the Skill Files).

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

### Step 3: Determine Global vs. Per-Endpoint Patterns

**IMPORTANT: First, ask which aspects vary per endpoint vs. are global across all endpoints.**

After parsing the OpenAPI spec, ask the user:

```
I've found [N] endpoints in your OpenAPI spec. Before creating the endpoint files,
I need to understand which aspects are consistent across all endpoints vs. which vary per endpoint.

Please tell me which of these aspects vary by endpoint:

1. **Authentication:** Same for all endpoints, or varies per endpoint?
2. **Error format:** Same for all endpoints, or varies per endpoint?
3. **Rate limits:** Same for all endpoints, or varies per endpoint?
4. **Retry strategy:** Same for all endpoints, or varies per endpoint?

For aspects that are "same for all", I'll create a shared reference.
For aspects that "vary", I'll ask endpoint-specific questions.
```

**[WAIT FOR USER RESPONSE]**

---

### Step 4: Gather API-Specific Details

Based on the user's response, collect details:

**For GLOBAL aspects (same across all endpoints), ask once:**

**Authentication (if not fully defined in spec):**
- ✓ _Authentication method_ — Auto-extracted from `securitySchemes` if present
- ✓ _Header/parameter names_ — Auto-extracted from `securitySchemes` if present
- ✗ What environment variables should developers set? (e.g., `YOUR_API_KEY`, `API_BASE_URL`)
- ✗ What endpoint can developers call to verify credentials work? (e.g., `/v1/me`, `/v1/health`)

**Errors (if global):**
- ✓ _Error response format_ — Auto-extracted from 4xx/5xx response schemas if defined
- ✗ Which errors are retryable? Which indicate a permanent failure?
- ✗ Retry strategy: exponential backoff details (initial delay, max retries)

**Rate limits (if global):**
- ✗ What are your per-minute and burst limits?
- ✗ What headers indicate rate limit status? (e.g., `X-RateLimit-Remaining`)
- ✗ Does your API send `Retry-After` headers?

**Timeouts (if global):**
- ✗ What are recommended timeout values? (e.g., 30s for all operations)

**Pagination (if applicable):**
- ✓ _Pagination style_ — Infer from response schemas if fields like `cursor`, `next`, `offset`, or `page` are present
- ✗ What fields indicate more results exist? (e.g., `has_more`, `next_cursor`)

**For PER-ENDPOINT aspects (vary by endpoint), ask for each endpoint:**

For each endpoint that varies, ask:
- "For [METHOD /path], what is the specific [error format/rate limit/retry strategy]?"

**For ALL endpoints, always ask:**
- Does this endpoint require idempotency? If yes, what header and window?
- What are the required fields (always required)?
- What are context-dependent fields (required for EMEA, Enterprise, specific use cases)?

---

**IMPORTANT FLOW:**
1. Ask which aspects vary (Step 3)
2. **WAIT for user response**
3. Ask detailed questions (Step 4)
4. **WAIT for user to provide all answers**
5. **ONLY after receiving all answers**, acknowledge with "Thank you for providing those details"
6. **THEN** proceed to create the skill files

Do NOT say "Perfect!" or "Great!" or start creating files before the user has answered your questions.

### Step 5: Create the Skill Files

Create the output directory in the user's current working directory (or ask the user for a preferred path). Structure:

```
[your-api]-best-practices/
  SKILL.md                    (main orchestrator)
  endpoints/
    POST-v1-payment-intents.md
    POST-v1-customers.md
    GET-v1-customers-{id}.md
    ... (one file per endpoint)
```

**Before finalizing:** Search all generated files for `example.com` and other placeholder URLs — replace every occurrence with the user's actual API domain. Placeholder leakage is a common mistake.

**Optional — Language-specific examples:** Ask the user if they want code examples in specific languages (Python, Node.js, Go, etc.) in addition to curl. If yes, include language-specific snippets in the endpoint files alongside the curl examples.

---

**File 1: SKILL.md (Main Orchestrator)**

```markdown
---
name: [your-api]-best-practices
description: "Best practices for integrating with [Your API]. Use when: implementing [Your API] integration, handling [Your API] errors, setting up [Your API] webhooks, troubleshooting [Your API] issues."
---

# [Your API] Integration Best Practices

## How to Use This Skill

**This skill validates your API integration against [Your API]'s best practices.**

**To use, provide one of the following:**
1. Copy/paste your API request (curl, JSON, HTTP format)
2. Upload a file with your integration code
3. Copy/paste your code snippet

**Authoritative Source:**
- This skill uses ONLY the best practices defined in this skill
- All validation rules come from the official API specification
- No web searches or external docs - ensuring accuracy

**What happens next:**
1. I'll identify which endpoint(s) you're calling
2. Load the best practices for that endpoint
3. Validate your implementation
4. Report any issues with specific recommendations

**Example:**
```bash
curl -X POST "https://api.example.com/v1/payment_intents" \
  -H "Authorization: Bearer sk_test_123" \
  -d '{"amount": 1000, "currency": "usd"}'
```

---

## Analysis Process

**When you provide your code, I will:**

1. **Identify which endpoint(s)** you're calling
2. **Determine input type:**
   - Single API request (curl/JSON) → Validate request structure only
   - Full code implementation → Validate everything (request + error handling + retries + timeouts)

3. **Load endpoint-specific best practices** from the `endpoints/` directory

4. **Run validation checks** based on input type:
   - **Request-level:** Authentication, required fields, idempotency headers
   - **Code-level:** Error handling, retry logic, rate limiting, timeouts (for full code only)

5. **Report findings** using this format:
   ```
   ⚠️  Issue: [What's wrong]
   Why: [Consequence]
   Recommendation: [How to fix - code snippet]
   ```

6. **Summary:** Count of issues by category

**Philosophy:**
- These are recommendations, not requirements
- You may have valid reasons to deviate
- If you acknowledge a finding and choose to proceed, that's acceptable

**If user says they want to ignore a recommendation:**
- Acknowledge the decision
- Move on to other aspects

---

## Endpoints

This API has the following endpoints (each has detailed best practices in `endpoints/` directory):

[LIST ALL ENDPOINTS HERE - auto-generated from OpenAPI spec]
- POST /v1/payment_intents → See endpoints/POST-v1-payment-intents.md
- POST /v1/customers → See endpoints/POST-v1-customers.md
- GET /v1/customers/{id} → See endpoints/GET-v1-customers-{id}.md
...

When you provide your code, I'll automatically load the relevant endpoint file(s).
```

---

**File 2: endpoints/POST-v1-payment-intents.md (Example Endpoint File in Conversational Format)**

Use this conversational format for each endpoint file:

```markdown
# POST /v1/payment_intents

## What You Need to Know

**This endpoint creates a payment intent. It requires idempotency because duplicate payments are costly.**

### Quick checklist

When you call this endpoint, make sure you:
1. Include an `Authorization: Bearer` header with your API key
2. **Include an `Idempotency-Key` header** (see below - this is critical)
3. Include `amount` and `currency` in the request body
4. Include `external_id` if you're an Enterprise customer
5. Include `gdpr_consent: true` if serving EMEA customers

### About Idempotency

This endpoint **requires** an idempotency key because payment failures could cause duplicate charges.

Add a header like this:
```bash
-H "Idempotency-Key: pi_abc123"
```

Generate unique keys using UUID or similar:
```python
import uuid
key = f"pi_{uuid.uuid4()}"
```

The idempotency window is **24 hours**. Reusing the same key within 24 hours returns the cached result.

### Example: Good Request

Here's a complete, correct request:
```bash
curl -X POST "https://api.example.com/v1/payment_intents" \
  -H "Authorization: Bearer sk_live_abc123" \
  -H "Idempotency-Key: pi_550e8400" \
  -d '{
    "amount": 1000,
    "currency": "usd",
    "external_id": "order_789"
  }'
```

### Example: Common Mistakes ❌

```bash
# Missing idempotency key - risky!
# Missing external_id for enterprise
curl -X POST "https://api.example.com/v1/payment_intents" \
  -H "Authorization: Bearer sk_test_123" \
  -d '{"amount": 1000, "currency": "usd"}'
```

### Required Fields

**Always required:**
- `amount` (integer) - Amount in cents
- `currency` (string) - Three-letter ISO code (e.g., "usd", "eur")

**Context-dependent (may be required):**
- `gdpr_consent` (boolean) - Required for EMEA customers
- `external_id` (string) - Required for Enterprise customers
- `payment_method_data.card.cvc` - Required when payment_method=card

### For Full Code Implementations

If you're writing integration code (not just testing with curl), also make sure to:

**Error handling:**
- Check the response status code before parsing the response
- Handle errors: 400 (bad request), 429 (rate limit), 5xx (server error)
- Parse error format: `{"error": {"code": "...", "message": "...", "param": "..."}}`

**Retry logic:**
- Retry transient errors (429, 5xx) with exponential backoff
- Do NOT retry client errors (4xx except 429)
- Respect `Retry-After` header when present

**Timeouts:**
- Set a 30-second timeout for this endpoint

**Rate limits:**
- This endpoint uses the global rate limit: 100 requests/minute
- Check `X-RateLimit-Remaining` header to track usage
- When you hit 429, wait for `Retry-After` seconds

### What This Skill Validates

**Request-level checks (applies to curl and code):**
- ✓ Authentication header present
- ✓ Idempotency-Key header present
- ✓ Required fields (amount, currency) included
- ✓ Context fields included when needed (gdpr_consent, external_id)

**Code-level checks (full implementations only):**
- ✓ Response status checked before parsing
- ✓ Error handling for 400, 429, 5xx
- ✓ Retry logic with exponential backoff
- ✓ Appropriate timeout set (30s)

---

## Validation Execution

**When I analyze your code, I will:**

1. Check if you provided a single request or full code
2. Run request-level checks for both
3. Run code-level checks only for full code
4. Report ALL issues found using this format:

```
⚠️  Issue: Missing Idempotency-Key header
Why: Duplicate requests may create multiple charges if network fails
Recommendation: Add this header:

Before:
curl -X POST "https://api.example.com/v1/payment_intents" \
  -H "Authorization: Bearer sk_live_123" \
  -d '{"amount": 1000, "currency": "usd"}'

After:
curl -X POST "https://api.example.com/v1/payment_intents" \
  -H "Authorization: Bearer sk_live_123" \
  -H "Idempotency-Key: pi_$(uuidgen)" \
  -d '{"amount": 1000, "currency": "usd"}'

Note: If you're handling idempotency at a different layer (load balancer, API gateway),
you can acknowledge and proceed.
```

**Philosophy:**
- These are recommendations to help you integrate successfully
- You may have valid architectural reasons for different implementations
- If you acknowledge a finding and choose to proceed, that's acceptable

```

---

**For each endpoint in your API, create a similar file using the conversational format above.**

Key elements of the conversational format:
1. **What You Need to Know** - Plain language explanation
2. **Quick checklist** - Actionable items
3. **Critical sections first** - e.g., idempotency if required
4. **Good example** - Shows correct implementation
5. **Common mistakes** - Shows what NOT to do
6. **Required fields** - Clear table or list
7. **For Full Code** - Additional requirements for implementations
8. **What This Skill Validates** - Clear separation of request vs code checks

---

## Step 6: Deliver ALL Files

**If file-writing tools are available (e.g., running in Claude Code):**

Write all files directly to disk:
1. Create `[your-api]-best-practices/` and `endpoints/` in the user's current working directory (or ask for a preferred path)
2. Write the main `SKILL.md`
3. Write every endpoint file into `endpoints/`
4. Confirm with a file listing and total count

---

**If no file-writing tools are available (e.g., running in Claude.ai):**

Present every file in the chat in this exact order and format:

```
I've created [N] files for your [API Name] best practices skill.
Copy each file's content into the correct path to set up the skill.

Directory structure:
[your-api]-best-practices/
  SKILL.md
  endpoints/
    POST-v1-example.md
    ... (one per endpoint)
```

Then output each file as a clearly labelled block:

---
**File 1 of [N]: `[your-api]-best-practices/SKILL.md`**
```markdown
[COMPLETE FILE CONTENT]
```

---
**File 2 of [N]: `[your-api]-best-practices/endpoints/POST-v1-example.md`**
```markdown
[COMPLETE FILE CONTENT]
```

[Continue for every endpoint file — do NOT skip any]

---

**CRITICAL rules for the chat presentation path:**
- Show ALL files — never summarize or truncate
- Always show the main `SKILL.md` first, then endpoint files in order
- Include the full file path as the label so the user knows exactly where to save it
- End with: "All [N] files above. Create the directory structure and copy each file to its path."

---

## Step 7: Testing the Generated Skill

Before distributing the skill:

1. Test with real requests - provide actual curl commands to verify validation works
2. Test with code samples - provide full implementation code to verify code-level checks work
3. Test edge cases - verify context-dependent fields are properly detected
4. Have a developer unfamiliar with the API try using it and note confusions

## Summary

By following this skill creator, you will generate (or update):

**1. Main SKILL.md file** - Orchestrator that:
- Explains how to use the skill
- Identifies which endpoint the user is calling
- Loads the appropriate endpoint file(s)
- Runs validation based on input type

**2. Per-endpoint files** - One for each endpoint, using conversational format:
- Plain language explanation of the endpoint
- Quick checklist of requirements
- Good and bad examples
- Required and context-dependent fields
- Request-level and code-level validation rules

**Key benefits of this structure:**
- ✓ Easy to find - one file per endpoint
- ✓ Easy to read - conversational, example-first format
- ✓ Easy to maintain - update one endpoint without affecting others
- ✓ Realistic - captures per-endpoint variations (errors, rate limits, etc.)
- ✓ Scalable - add new endpoints by adding new files

**Remember:**
- Always ask which aspects vary per endpoint vs. are global
- Use the conversational format for readability
- Include both good and bad examples
- Separate request-level from code-level checks
- Make recommendations advisory, not blocking

---

## Updating an Existing Skill

**When the user asks to update an existing API best practice skill, follow this process:**

### Identify and Read

1. Ask the user: which skill, what changes, which files are affected
2. Read the main SKILL.md and all affected endpoint files to understand the current structure

### Make Consistent Updates

For every update, apply changes across **all** affected files — not just some:

- **Maintain the conversational format** used in existing files
- **Keep the same structure**: Quick Checklist → Examples → Required Fields → Validation Checks
- **Update examples** in every affected file to reflect the change
- **Add validation checks** to "What This Skill Validates" in every affected file
- **Update the main SKILL.md** if adding global best practices or new endpoints

**Common scenarios and what to touch:**

| Scenario | Main SKILL.md | Endpoint files | What to update |
|----------|:---:|:---:|----------------|
| Deprecated field warnings | ✓ | All with that field | Examples (old vs new), validation checks, Quick Checklist |
| New required fields | — | Affected endpoints | Quick Checklist, examples, Required Fields, validation checks |
| New best practices | If global | If endpoint-specific | Examples, validation checks |
| New endpoint | Add to list | Create new file | Follow conversational format from existing files |

### Write and Summarize

1. Write all updated files back to disk (do not just display diffs in chat)
2. Summarize: what changed, how many files updated, any breaking changes
