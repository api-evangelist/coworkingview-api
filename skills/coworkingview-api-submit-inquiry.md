---
name: coworkingview-submit-inquiry
description: Forward a workspace enquiry for one space or a shortlist to the operator via CoworkingView, safely retryable.
api: CoworkingView REST API
base_url: https://api.coworkingview.com
operations:
  - inquiry
  - contact
---

# Submit a workspace enquiry

CoworkingView forwards your enquiry to the operator; it does not broker the contract, hold availability, or take
payment. There is nothing to cancel through the API — get the details right before sending.

## Steps

1. **Pick the target.** Use the search skill to settle on one or more property slugs.
2. **Submit the lead.** `POST /v1/leads/inquiry` (operationId `inquiry`) with JSON body — required: `name`, `email`,
   `locale`, `properties` (array of slugs). Optional: `phone`, `company`, `message`, `desks`, `moveInDate`,
   `attribution`. For a general (not space-specific) enquiry use `POST /v1/leads/contact` (`contact`) instead.
3. **Pass the Turnstile token.** In production a Cloudflare Turnstile `challengeToken` is required; a missing/invalid
   token returns `403 CHALLENGE_FAILED`.
4. **Make it idempotent.** Send an `Idempotency-Key` header (a UUID per logical submission). A retry with the SAME key
   and body replays the original response instead of creating a second lead; the same key with a DIFFERENT body returns
   `422 IDEMPOTENCY_KEY_CONFLICT`.

## Rules

- Validation failures return `422 VALIDATION_FAILED` with field-level `errors[]` ({path, message}) — fix and retry.
- Do not promise availability or a price; the operator closes the contract directly with the searcher.
- To withdraw or amend a sent enquiry, contact the operator out of band — the API exposes no reversal operation.
