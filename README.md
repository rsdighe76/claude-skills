# API Best Practice Skill Creator

A Claude skill that generates (or updates) API best practice documentation tailored to a specific API. Built for PMs, Solution Engineers, Architects, Developer Evangelists, and Developer Relations teams.

## What It Does

Give this skill your OpenAPI/Swagger spec and answer a few questions about your API's behavior. It produces a complete best practice skill with:

- **Main SKILL.md** — Orchestrator that identifies endpoints, loads per-endpoint rules, and validates user code
- **Per-endpoint files** — One file per endpoint covering required fields, idempotency, error handling, retries, rate limits, and context-dependent requirements

The generated skill can then validate API integrations (curl commands, code snippets, or full implementations) against your documented best practices.

## What It Covers

| Topic | Description |
|-------|-------------|
| Authentication | Credential storage, environment config, testing |
| Required fields | Context-dependent fields by use case, region, or segment |
| Error handling | Your error format, status codes, retryable vs permanent |
| Retry & backoff | Exponential backoff, Retry-After headers |
| Rate limiting | Limits, detection headers, staying within bounds |
| Idempotency | Key format, which operations need them, window duration |
| Timeouts | Recommended values per operation type |
| Webhooks | Signature verification, delivery guarantees |
| Pagination | Cursor/offset patterns, iterating through results |

## Two Ways to Provide Your API Details

You can provide your API details in **three ways** — pick whichever works best for you:

### Option A: Generate a customized intake template from your spec (recommended)

Provide your OpenAPI spec and ask the skill to generate a customized intake template. It will parse your spec and search your public developer docs to produce a template with your actual API name, endpoints, auth method, error format, rate limits, and more pre-filled. Fields from your spec are marked **[auto]** and fields from public docs are marked **[web]** so you know what to review. You just confirm, correct, fill in any gaps, and paste it back.

### Option B: Fill out the generic intake template manually

Use the **[Intake Template](INTAKE-TEMPLATE.md)** to gather everything upfront offline. Fill in the sections, then paste the completed template into Claude along with your OpenAPI spec. This is ideal when multiple people are contributing details.

See **[Intake Example](INTAKE-EXAMPLE.md)** for a fully completed version using a fictional Payments API.

### Option C: Answer questions conversationally (great for simpler APIs)

Just provide your OpenAPI spec and the skill will walk you through it step by step — asking about rate limits, idempotency, context-dependent fields, etc. as it goes. No prep needed beyond having your spec ready.

**Either way, you'll need the same information:**

1. **Your OpenAPI/Swagger spec** — JSON or YAML, v2.0 or 3.x
2. **Rate limit details** — Per-minute limits, burst limits, rate limit headers, Retry-After behavior
3. **Error format** — Your error response structure and which status codes are retryable vs permanent
4. **Idempotency rules** — Which endpoints need idempotency keys, header name, window duration
5. **Context-dependent requirements** — Fields required for specific regions, segments, or use cases
6. **Retry strategy** — Backoff approach, initial delay, max retries
7. **Timeout recommendations** — Per endpoint or operation type
8. **Webhook details** (if applicable) — Signature format, verification, delivery guarantees

## How It Works

### Creating a New Skill

1. **Provide your OpenAPI/Swagger spec** and either a filled-out intake template or just the spec alone
2. **Answer questions** (if not using the template) about which aspects vary per endpoint vs. are global
3. **Provide API-specific details** (rate limits, retry strategies, context-dependent fields, etc.)
4. **Receive complete skill files** ready to use

### Updating an Existing Skill

1. **Point to the existing skill** and describe what needs changing
2. **The skill reads the current structure** and makes consistent updates across all affected files
3. **All modified files are presented** with a summary of changes

Supports common update scenarios: deprecated fields, new required fields, new best practices, and new endpoints.

## Sample Prompts

### Generating a customized intake template (Option A)

> "Here's my OpenAPI spec: https://api.example.com/openapi.json — generate an intake template for my API."

> "Parse this spec and create a customized intake template so I know exactly what to fill in."

### Creating a skill (with completed template — Option A or B)

> "Here's my completed intake template and OpenAPI spec — generate the best practice skill." _(paste filled-out template below)_

### Creating a skill (conversational — Option C)

> "Create a best practice skill for our Payments API. Here's the OpenAPI spec: https://api.example.com/openapi.json"

> "I have a REST API for order management. Here's the Swagger spec. Build me a best practice skill that covers authentication, error handling, and idempotency."

> "Generate an API best practice skill from this OpenAPI 3.0 spec. Our API has different rate limits per endpoint and requires GDPR consent fields for European customers."

> "Create a best practice skill for our webhook-heavy API. We need signature verification guidance and retry handling for each endpoint."

### Updating a skill

> "Update the payments skill to add deprecation warnings for the `legacy_config` field across all endpoints."

> "Add a new POST /v2/refunds endpoint to my existing payments best practice skill."

> "Our rate limits changed from 100/min to 250/min. Update the skill to reflect the new limits across all endpoint files."

> "Add a new required field `shipping_address` to the create order endpoint. It's required for all physical goods orders."

### Validating with a generated skill

> "Here's my curl command for creating a payment. Check it against best practices."

> "Review this Python integration code against the best practices skill. [paste code]"

> "I'm getting 429 errors. What does the best practice skill say about rate limiting for this endpoint?"

## Output Structure

```
your-api-best-practices/
  SKILL.md                          # Main orchestrator
  endpoints/
    POST-v1-payment-intents.md      # Per-endpoint best practices
    POST-v1-customers.md
    GET-v1-customers-{id}.md
    ...
```

## Design Principles

- **Authoritative** — Uses only information you provide; no web searches or assumptions
- **Deterministic** — Same input always produces the same validation results
- **Advisory** — Recommendations, not blockers; users can acknowledge and proceed
- **Endpoint-first** — Each endpoint has its own file for easy maintenance and scalability

## Author

**Rahul Dighe**
- Email: rsdighe@hotmail.com
- LinkedIn: [linkedin.com/in/rahuldighe](https://www.linkedin.com/in/rahuldighe)
