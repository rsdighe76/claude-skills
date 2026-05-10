---
name: api-best-practice-skill-creator
description: "TRIGGER when: user wants to create or update an API best practice skill, generate integration docs from an OpenAPI spec, build a validator for a specific API, document error handling or retry patterns, or create context-dependent field requirements for a specific API. DO NOT TRIGGER when: user is asking about general REST concepts, authentication theory, or wants to validate code against an already-generated API skill."
---

# API Best Practice Skill Creator

Create or update best practice documentation tailored to your specific API. This guide helps PMs, Solution Engineers, Architects, Developer Evangelist and Developer Relations teams or other such folks build comprehensive best practice skills that ensure developers integrate correctly from the start or use it as a validator upon integration. 

## On First Invocation

**If the user has not yet provided an API spec or intake template, immediately present this welcome message — do not wait for them to ask:**

---

Welcome! I create best practice documentation tailored to your specific API — covering authentication, error handling, retries, rate limits, idempotency, and per-endpoint requirements.

To get started I need your OpenAPI or Swagger spec. Choose one of three paths:

**Option A — Generate a customized intake template (recommended)**
Paste or link your spec and say "generate an intake template." I'll pre-fill a template with your actual endpoints, auth method, and error format — you just fill in the gaps.

**Option B — Fill out the template manually**
Download the [Intake Template](https://github.com/rsdighe76/api-skill-creator/blob/master/INTAKE-TEMPLATE.md) and paste it back with your spec.

**Option C — Just give me the spec**
Share your spec and I'll ask questions as we go — no prep needed.

**Accepted formats:** OpenAPI 3.x or Swagger 2.0 (JSON/YAML), a URL to a hosted spec, or pasted content.

Please share your API spec to begin.

---

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

### Web Search Rule

Before fetching any URL or generating any content, you must:

1. List every URL you plan to fetch except the one the user provided to you, numbered
2. Stop completely — do not fetch, do not generate, do not proceed
3. Wait for the user to reply with "all", "none", or specific numbers
4. Only then fetch the approved URLs and continue

This applies at every stage of the workflow, not just the beginning. If you are about to fetch a URL at any point, pause and ask first. Treat every web fetch as requiring explicit approval.

---

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

If you'd prefer to work offline, fill out the [Intake Template](https://github.com/rsdighe76/api-skill-creator/blob/master/INTAKE-TEMPLATE.md) with your API details and paste it along with your spec. See the [Intake Example](https://github.com/rsdighe76/api-skill-creator/blob/master/INTAKE-EXAMPLE.md) for a completed version.

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

**API version check:** When parsing the spec, extract the version from `info.version`. If the spec doesn't declare a version, ask the user before generating: "What API version should this skill target? This will appear as line 1 of the generated SKILL.md." Never omit or bury the version.

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

**Phase 2: Show planned URLs and wait for user selection — DO NOT search yet**

Based on the API name and spec, determine which public URLs you would search. Present the numbered list to the user and STOP. Do not fetch any URL until the user replies.

```
Before I search the web, here are the URLs I plan to check:

1. [URL] — rate limits and throttling docs
2. [URL] — error code reference
3. [URL] — authentication and token guide
...

Which would you like me to search?
- "all" — search all of them
- "none" — skip web search, I'll fill in the blanks myself
- numbers (e.g. "1, 3") — search only those
```

**STOP HERE. Do not fetch any URL. Do not proceed to Phase 2b. Do not generate the template. Wait for the user to reply with their selection before doing anything else.**

Once the user replies, fetch only the URLs they selected. Leave all other web-sourced fields blank.

**Phase 2b: Enrich from approved URLs**

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

Before presenting the template, list every URL fetched during Phase 2 so the user can verify the sources:

```
Sources consulted (web):
- [URL 1] — [what was found there]
- [URL 2] — [what was found there]
(If no web sources were consulted, say "No web sources consulted — all fields from spec only.")
```

Then present the customized template and say:
```
Here's your customized intake template based on your OpenAPI spec
and public documentation.

Fields marked [auto] were extracted from your spec.
Fields marked [web] were found in the sources listed above — please review
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

**Deployment target — always ask:**

```
Where will this skill be used?
A) Claude Code (CLI / IDE extension — files live on disk, relative paths work)
B) Claude.ai Project (upload all files directly to the Project)

Both options use relative paths in SKILL.md. The difference is delivery:
- Claude Code: files stay on disk in the generated directory
- Claude.ai: you upload every file (SKILL.md + all endpoints/ + shared/) to the Project knowledge base
```

Store the answer as `DEPLOYMENT_TARGET` (code or claude_ai). Both targets use identical relative-path routing in SKILL.md — no GitHub raw URLs needed.

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
  shared/
    authentication.md         (credentials, token setup, env vars, scopes)
    error-codes.md            (global error format, universal status codes, retry rules)
    workflows.md              (multi-step patterns: creation, pagination, lifecycle, teardown, recovery)
  endpoints/
    POST-v1-payment-intents.md
    POST-v1-customers.md
    GET-v1-customers-id.md    (no curly braces — use plain param name, e.g. customer_id not {customer_id})
    ... (one file per endpoint)
```

**Filename convention for path parameters:** strip curly braces from parameter names in filenames. `GET /v1/customers/{customer_id}` → `GET-v1-customers-customer_id.md`. Curly braces are invalid characters in zip/skill bundles on some platforms.

**Before delivering any file, verify SKILL.md starts with `---` on line 1.** Read back the first line after writing. If it is not exactly `---` — if it starts with a blank line, introductory text, or a ` ``` ` fence — rewrite the file with `---` as the first characters. Claude.ai rejects the upload with "SKILL.md must start with YAML frontmatter" otherwise.

**Before delivering any file, scan every generated file for placeholder leakage.** Search for all of the following strings and replace with the user's actual values from their spec or intake:

- `example.com` / `api.example.com`
- `YOUR_API` / `[YOUR_API_NAME]` / `[API Name]` / `[Your API]`
- `sk_test_123` / `<your-key>` / `your_token_here`
- Any remaining template brackets: `[` or `]` in content (not in markdown tables or code comments)
- In SKILL.md frontmatter: flag if `name` still contains `[your-api]` or `<api-name>`, or if `description` still contains `[Your API]`, `[API Name]`, or `<one or two sentences>` — these must be replaced with the actual API name and a real description before delivery.

After replacing, report to the user: **"0 placeholder leaks found"** or **"Fixed N placeholder leaks."** Never deliver files with placeholder leakage.

**Optional — Language-specific examples:** Ask the user if they want code examples in specific languages (Python, Node.js, Go, etc.) in addition to curl. If yes, include language-specific snippets in the endpoint files alongside the curl examples.

---

**File 1: SKILL.md (Main Orchestrator)**

It is a routing hub — not a manual. If you find yourself writing a paragraph here, it belongs in a reference file instead.

**CRITICAL — frontmatter rule:**
- The frontmatter block must be the **very first thing** in SKILL.md — before any headings, text, or content.
- The file content begins with `---` on line 1. Nothing before it — no blank lines, no introductory sentence, no ` ```markdown ` fence, no BOM.
- The template below is shown inside a code block for readability only. When writing the file, do NOT include the opening ` ```markdown ` or closing ` ``` ` markers — write only the content inside them.
- After writing SKILL.md, read back line 1 and confirm it is exactly `---`. If it is not, rewrite the file.
- `name` must be kebab-case, e.g. `paypal-button-best-practices`, `stripe-payments`, `twilio-sms`.
- `description` must be specific enough for Claude to know when to activate the skill — include the API name, key actions (e.g. "order creation, capture, webhook verification"), and the kinds of developer questions it answers.

**Bad — will fail on upload:**
```markdown
# My API Best Practices Skill       ← ❌ no frontmatter
**API Version:** v2
```

**Correct pattern:**
```markdown
---
name: my-api-best-practices
description: "Use when a developer asks about integrating My API — covers authentication, order creation, error handling, and webhook verification."
---

# My API Best Practices Skill       ← ✅ frontmatter first, then content
```

```markdown
---
name: [your-api]-best-practices
description: "Use when a developer asks about integrating [Your API] — covers [key actions e.g. authentication, order creation, error handling, webhook verification]. Always consult this skill before writing any [API Name] integration code — do not guess at endpoints, fields, or auth."
---

**API version:** [X.Y.Z from spec `info.version`]. Always use this version unless the user specifies otherwise.

Validates and guides [Your API] integrations. Paste a request, code snippet, or describe what you're building — I'll read the relevant file and validate or guide.

**IMPORTANT:** Before validating any request or code, read the relevant file from the table below. Do not validate without first reading the file content.

## What Are You Building?

| Building... | Read this file |
|---|---|
| [Developer goal 1] | `endpoints/[file].md` |
| [Developer goal 2] | `endpoints/[file].md` |
| Setting up credentials, tokens, or auth | `shared/authentication.md` |
| Handling errors, retries, or rate limits | `shared/error-codes.md` |
| Multi-step workflows or lifecycle patterns | `shared/workflows.md` |

## When to Load Which File

- **Auth setup** → read `shared/authentication.md`
- **Error codes, retry logic** → read `shared/error-codes.md`
- **Multi-step workflows** → read `shared/workflows.md`
- **Any endpoint** → read the matching file from the table above

Validate a **single request (curl/JSON)** → check request structure only.
Validate **full code** → check request + error handling + retries + timeouts.

## Validation Format

All findings use this format:

```
⚠️ Issue: [what is wrong]
Why: [consequence if ignored]
Recommendation: [how to fix — before/after code]
Note: If you have a valid reason to deviate, acknowledge and proceed.
```
```

---

**File 2: endpoints/POST-v1-[example].md (Endpoint File Format)**

Use this format for every endpoint file. Open with one sentence — what the endpoint does and when to read this file. Show the working example first. Gotchas second. Field reference last. Never open with a table.

```markdown
# POST /v1/[endpoint]

Read this file when: you are calling POST /v1/[endpoint] and want to know what's required, what can go wrong, or how to retry safely.

## The Working Request

Here's a correct, complete request:

```bash
curl -X POST "https://[actual-api-domain]/v1/[endpoint]" \
  -H "Authorization: Bearer $[YOUR_API]_TOKEN" \
  -H "Idempotency-Key: $(uuidgen)" \
  -H "Content-Type: application/json" \
  -d '{
    "[required_field_1]": [value],
    "[required_field_2]": [value]
  }'
```

## Gotchas

The non-obvious things that catch developers by surprise:

- **[Gotcha 1]** — [why it matters and what goes wrong if you miss it]
- **[Gotcha 2]** — [e.g. "Idempotency key must be stored before the call — if your process crashes, you need that key to retry"]
- **[Gotcha 3]** — [context-dependent fields: "If billing_country is in EU, vat_number is required or the request will be rejected"]

## What Fails ❌

```bash
# [Describe what's wrong in comment]
curl -X POST "https://[actual-api-domain]/v1/[endpoint]" \
  -H "Authorization: Bearer $[YOUR_API]_TOKEN" \
  -d '{"[missing or wrong field]": [value]}'
```

## Required Fields

**Always required:**
- `[field]` ([type]) — [what it is]

**Context-dependent:**
- `[field]` ([type]) — required when [condition]

**Do NOT send:**
- `[field]` — [why, e.g. "computed server-side"]

## For Full Code Implementations

**Error handling:**
- For global error codes, retry rules, and error format: see `shared/error-codes.md`
- Endpoint-specific errors:
  - `[status] [code]` — [when this happens and what to do]

**Timeout:** [N] seconds.

**Rate limit:** [N] requests/minute ([bucket name]).

## What This Skill Validates

**Request-level checks (applies to curl and code):**
- ✓ [check 1 — e.g. Authorization header present with correct scope]
- ✓ [check 2 — e.g. Idempotency-Key header present and 8–128 chars]
- ✓ [check 3 — e.g. required fields included]
- ✓ [check 4 — e.g. context-dependent fields included when condition applies]

**Code-level checks (full implementations only):**
- ✓ [check 1 — e.g. timeout set to N seconds]
- ✓ [check 2 — e.g. 4xx vs 5xx handled separately]
- ✓ [check 3 — e.g. Retry-After header respected on 429]
- ✓ [check 4 — e.g. idempotency key stored before call, not generated inline]
```

**For each endpoint in your API, create a file following this format.** The key discipline: one sentence open → working example → gotchas → mistakes → fields → validation. Never start with a table or a checklist.

---

**File 3: shared/error-codes.md**

Generate this file once, covering only what is truly global across all endpoints:

```markdown
# [Your API] Error Reference

## Error Response Format

All errors from [Your API] follow this format:

```json
[PASTE ACTUAL ERROR RESPONSE FORMAT FROM INTAKE — e.g.:]
{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "The field 'amount' must be a positive integer.",
    "param": "amount"
  }
}
```

## Status Codes

| Code | Meaning | Retryable | Action |
|------|---------|:---------:|--------|
| 400  | Bad request — invalid parameters | No | Fix the request |
| 401  | Authentication failed | No | Check credentials |
| 403  | Forbidden — insufficient permissions | No | Check API key scopes |
| 404  | Resource not found | No | Check the ID/path |
| 409  | Conflict — duplicate or state mismatch | No | Check idempotency key |
| 422  | Unprocessable — business rule violation | No | Read the error message |
| 429  | Rate limited | Yes | Wait for `Retry-After` header |
| 500  | Server error | Yes | Retry with backoff |
| 502  | Bad gateway | Yes | Retry with backoff |
| 503  | Service unavailable | Yes | Retry with backoff |

[Adjust the table above to match only the status codes your API actually returns]

## Retry Strategy

**Retryable errors:** [LIST STATUS CODES — e.g., 429, 500, 502, 503]

**Non-retryable errors:** [LIST STATUS CODES — e.g., 400, 401, 403, 404, 409, 422]

**Backoff approach:**
- Initial delay: [e.g., 1 second]
- Multiplier: [e.g., exponential — delay * 2^attempt]
- Jitter: [e.g., add random 0–500ms to avoid thundering herd]
- Max delay: [e.g., 30 seconds]
- Max retries: [e.g., 3]
- Always respect `Retry-After` header when present on 429 responses

## Common Mistakes

| Mistake | Result | Fix |
|---------|--------|-----|
| Retrying a 400 | Wasted requests, same error | Fix the request first |
| Not checking `Retry-After` on 429 | Getting blocked longer | Read the header value |
| Retrying without backoff | Getting rate limited again | Add exponential backoff |
| Catching all errors the same way | Missing recoverable vs permanent | Branch on status code |
```

**Note:** Do NOT put endpoint-specific business errors (e.g., `insufficient_funds`, `card_declined`) in this file — those belong in each endpoint's own file under **For Full Code Implementations**.

---

**File 3: shared/authentication.md**

Generate this file once. It covers everything a developer needs to set up credentials and make their first authenticated call — nothing more.

```markdown
# [Your API] — Authentication

Read this file when: you are setting up credentials for the first time, getting 401 errors, or need to know which scopes to request.

## Credentials Setup

Set these environment variables before running any code:

```bash
export [YOUR_API]_CLIENT_ID=your_client_id
export [YOUR_API]_CLIENT_SECRET=your_client_secret
export [YOUR_API]_BASE_URL=[base_url_from_spec]
```

## Getting a Token

[Your API] uses [auth method from spec — e.g. OAuth2 Client Credentials]:

```bash
curl -X POST "[token_url]" \
  -d "grant_type=client_credentials" \
  -d "client_id=$[YOUR_API]_CLIENT_ID" \
  -d "client_secret=$[YOUR_API]_CLIENT_SECRET" \
  -d "scope=[list required scopes]"
```

Extract `access_token` from the response and pass it as a Bearer header on every request:

```bash
-H "Authorization: Bearer $ACCESS_TOKEN"
```

## Scopes

| Scope | What it grants |
|-------|---------------|
| `[scope_1]` | [what it allows] |
| `[scope_2]` | [what it allows] |

Request only the scopes your integration needs.

## Verifying Credentials

[If there is a health/verify endpoint: describe it. If not: suggest the lightest read endpoint as a credential check.]

## Common Auth Errors

- **401** — token missing, expired, or malformed; re-fetch a token and retry
- **403** — token valid but missing required scope; re-request with correct scopes
```

---

**File 4: shared/workflows.md**

Generate this file by identifying the multi-step patterns a developer actually needs to accomplish real goals with this API. Don't just list endpoints — show how they chain together.

**Identify workflows by asking these questions about the spec:**
- What is the canonical "happy path" from zero to first value? (create entity → attach resource → confirm state)
- What lookups are commonly done by attribute instead of ID? (find by email before creating)
- What multi-step state transitions does the lifecycle require, and in what order?
- What cleanup or teardown sequences exist, and what must happen first? (cancel children before deleting parent)
- What retry or recovery patterns should a developer know when an operation fails mid-workflow?

**Minimum workflows to generate for any CRUD API:**
1. Entity creation with duplicate prevention (check-then-create pattern)
2. Full pagination across a list endpoint
3. Lifecycle advancement (moving a resource through its status states)
4. Teardown/deletion with prerequisite cleanup
5. Error recovery / retry for a failed write

**Add API-specific workflows** whenever the spec reveals: webhooks, batch operations, multi-resource linking, async job polling, or any operation that is obviously unsafe to call naively (e.g. a delete that cascades).

**For each workflow, include:**
- A short title and one-line "when to use this" note
- Numbered steps with exact method + path for each API call
- What data to extract from each response and pass to the next call
- Any precondition checks (e.g. "check if resource exists before creating")
- Any ordering constraints enforced server-side
- What can go wrong mid-workflow and how to recover
- Code or pseudocode for workflows involving loops or conditional branching (pagination, polling, retry-with-backoff); numbered prose for linear workflows

**Tone:** write as a senior engineer explaining to a teammate who knows REST but doesn't know this API. Assume competence, skip basics, emphasize the non-obvious.

```markdown
# [Your API] — Common Workflows

## 1. Create [Entity] Without Duplicates

**When to use:** First time onboarding a [entity], or when you can't guarantee the create hasn't already been called.

1. `GET /[entities]?[unique_attribute]=<value>` — check if it already exists
   - If response `data` array is non-empty → use the existing ID, skip to step 3
   - If empty → proceed to step 2
2. `POST /[entities]` — create the entity
   - Extract `id` from the response
3. Continue with the returned ID

**Recovery:** If step 2 fails mid-flight, re-run step 1 before retrying — the entity may have been created.

---

## 2. Paginate Through All [Entities]

**When to use:** Any time you need the full list, not just the first page.

```python
cursor = None
results = []
while True:
    params = {"limit": [MAX_PAGE_SIZE]}
    if cursor:
        params["cursor"] = cursor
    response = GET /[entities] with params
    results.extend(response["data"])
    cursor = response.get("next_cursor")
    if not cursor:
        break
```

**Note:** Always use `limit=[MAX_PAGE_SIZE]` to minimise the number of round trips.

---

## 3. Advance [Entity] Through Lifecycle

**When to use:** Moving a [entity] from [initial_state] → [next_state] → [final_state].

Valid transitions:
- `[state_a]` → `[state_b]`: [meaning]
- `[state_b]` → `[state_c]`: [meaning]

Steps:
1. `GET /[entities]/{id}` — confirm current status before transitioning
2. `PATCH /[entities]/{id}` with `{"status": "[next_state]"}` and `Idempotency-Key`
   - Extract updated `status` from response to confirm transition succeeded

**Constraint:** [Any server-enforced ordering, e.g. "must reach paid before fulfilled"]

**Recovery:** If the PATCH fails, reuse the same `Idempotency-Key` on retry — the server will return the original result if the transition already happened.

---

## 4. Delete [Entity] Safely

**When to use:** Removing a [entity] and ensuring prerequisites are met first.

1. `GET /[entities]/{id}` — confirm it exists and check current state
2. [Any prerequisite cleanup — e.g. cancel child resources first]
3. `DELETE /[entities]/{id}`
   - Expect `204 No Content` — do not parse a response body

**Warning:** [Note any irreversibility — e.g. "This is permanent. There is no undo."]

---

## 5. Recover a Failed Write

**When to use:** A POST or PATCH returned a network error or 5xx and you don't know if it succeeded.

For endpoints that **support idempotency** (`Idempotency-Key`):
1. Reuse the same key you sent originally
2. Retry the exact same request — the server returns the cached result if it already succeeded

For endpoints that **do not support idempotency** (e.g. POST /customers):
1. `GET /[entities]?[unique_attribute]=<value>` — check if the resource was created
2. If found → use the existing resource, do not retry the POST
3. If not found → retry the POST with a new request

[Add more API-specific workflows here if the spec reveals webhooks, batch ops, multi-resource linking, etc.]
```

---

## Step 6: Deliver ALL Files

**If file-writing tools are available (e.g., running in Claude Code):**

Write all files directly to disk:
1. Create `[your-api]-best-practices/`, `shared/`, and `endpoints/` in the user's current working directory (or ask for a preferred path)
2. Write the main `SKILL.md`
3. Write `shared/authentication.md`
4. Write `shared/error-codes.md`
5. Write `shared/workflows.md`
6. Write every endpoint file into `endpoints/`
7. Confirm with a file listing and total count

**If DEPLOYMENT_TARGET = claude_ai**, after writing the files, also create a zip bundle:

```bash
cd [your-api]-best-practices && zip -r ../[your-api]-best-practices.skill . && cd ..
```

Then tell the user:

```
Upload [your-api]-best-practices.skill to your Claude.ai Project.
The .skill file is a zip bundle — it must contain SKILL.md inside the folder.
All [N] files are included.
```

---

**If no file-writing tools are available (e.g., running in Claude.ai):**

Present every file in the chat in this exact order and format:

```
I've created [N] files for your [API Name] best practices skill.
Copy each file's content and upload all [N] files to your Claude.ai Project —
not just SKILL.md. Claude.ai resolves the endpoint and shared files from
the Project's knowledge base using the relative paths in SKILL.md.

Directory structure to recreate:
[your-api]-best-practices/
  SKILL.md
  shared/
    authentication.md
    error-codes.md
    workflows.md
  endpoints/
    POST-v1-example.md
    ... (one per endpoint)
```

Then output each file as a clearly labelled block, in this order: SKILL.md first, then shared files, then endpoint files:

---
**File 1 of [N]: `[your-api]-best-practices/SKILL.md`**
```markdown
[COMPLETE FILE CONTENT]
```

---
**File 2 of [N]: `[your-api]-best-practices/shared/authentication.md`**
```markdown
[COMPLETE FILE CONTENT]
```

---
**File 3 of [N]: `[your-api]-best-practices/shared/error-codes.md`**
```markdown
[COMPLETE FILE CONTENT]
```

---
**File 4 of [N]: `[your-api]-best-practices/shared/workflows.md`**
```markdown
[COMPLETE FILE CONTENT]
```

---
**File 5 of [N]: `[your-api]-best-practices/endpoints/POST-v1-example.md`**
```markdown
[COMPLETE FILE CONTENT]
```

[Continue for every endpoint file — do NOT skip any]

---

**CRITICAL rules for the chat presentation path:**
- Show ALL files — never summarize or truncate
- Always show the main `SKILL.md` first, then shared files, then endpoint files in order
- Include the full file path as the label so the user knows exactly where to save it
- Report placeholder leakage count before listing files: "0 placeholder leaks found" or "Fixed N placeholder leaks"
- End with: "All [N] files above. Save each file to the directory structure shown, then run `zip -r [your-api]-best-practices.skill [your-api]-best-practices/` and upload the .skill file to your Claude.ai Project."

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
