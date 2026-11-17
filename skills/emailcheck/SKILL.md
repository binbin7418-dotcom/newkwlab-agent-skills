---
name: emailcheck
description: Validate email addresses and clean email lists with the EmailCheck API on RapidAPI - one address per call, or a batch of up to 100 addresses in a single call that returns per-address verdicts plus a summary report. Returns valid, score (0-100), verdict (valid, risky, or invalid), checks for syntax, domain existence, MX records, disposable or temporary mail domains, role accounts like admin@ support@ noreply@, free providers like Gmail or Outlook, typo detection with a suggested correction (user@gamil.com becomes user@gmail.com), domain age in days from RDAP registration records (domainAgeDays), and a catch-all risk signal. Use when the user asks to validate an email, verify an email address, check if an email is real or deliverable, clean a mailing list, scrub a newsletter or CRM list before sending, reduce bounce rate, block disposable or throwaway signups, detect fake signups, check emails at signup or import, validate a CSV of addresses, or asks what score, verdict, suggestion, typoLikely, domainAgeDays, catchAllRisk, MISSING_EMAILS, EMPTY_BATCH, or BATCH_TOO_LARGE mean. One batch call with 100 addresses counts as one API call. For auth, hosts, and billing details use the newkwlab-router skill.
---

# EmailCheck — Email Validation and Batch List Cleaning

## Overview

EmailCheck answers one question per address — *should you trust this email?* — with the evidence attached, not a black-box yes/no. Send one address and get `valid`, a 0–100 `score`, and a `verdict` of `valid`, `risky`, or `invalid`, backed by syntax, domain and MX checks, disposable / role-account / free-provider signals, a typo suggestion when the domain looks misspelled, the domain's age from RDAP registration records, and a catch-all risk signal.

The batch endpoint is the workhorse: up to 100 addresses in **one call** (it bills as one request), returning every address validated individually **plus a summary report** — how many are valid, risky, invalid, disposable and likely typos, with counts and percentages. Read the summary first, then act on the per-address results.

Full plumbing reference (auth, errors, MCP setup): the `newkwlab-router` skill.

## Access

- REST host: `emailcheck-email-validation-api.p.rapidapi.com`
- MCP (preferred in agent clients): server at `https://mcp.rapidapi.com` with headers `x-api-host: emailcheck-email-validation-api.p.rapidapi.com` and `x-api-key: YOUR_RAPIDAPI_KEY`. Tools: `Validate_Email_via_GET`, `Validate_Email`, `Batch_validate`.
- Subscribe first (free BASIC tier: 100 calls/month): https://rapidapi.com/binbin7418/api/emailcheck-email-validation-api
- Auth on REST: headers `x-rapidapi-key: YOUR_RAPIDAPI_KEY` and `x-rapidapi-host: emailcheck-email-validation-api.p.rapidapi.com`. Never print or log the key.

## Endpoints

| Job | Endpoint | Input |
|---|---|---|
| Validate one address | `GET /validate` | `?email=user@example.com` |
| Validate one address | `POST /validate` | `{"email": "user@example.com"}` |
| Clean a list | `POST /validate/batch` | `{"emails": ["a@x.com", "b@y.com"]}` — up to 100 |

## Workflow

1. **One address?** Call `/validate` (GET is fine). Report the verdict, the score, and the two or three checks that decided it — do not dump the raw JSON without a conclusion.
2. **A list?** Split it into chunks of at most 100 and call `/validate/batch` once per chunk. State the cost first: *N chunks = N billed calls*, not one call per address.
3. **Read the summary before the details.** Lead with the shares: "of 100 addresses: 62 valid, 21 risky, 17 invalid, 9 disposable, 6 likely typos."
4. **Act per verdict.** `invalid` → drop. `typoLikely` with a `suggestion` → offer the corrected address, never silently rewrite it. `risky` → keep but flag; combine `score`, `disposable`, `domainAgeDays` and `catchAllRisk` for signup-defence decisions.
5. A bad entry inside a batch marks only itself `invalid`; it never fails the batch. Named batch errors: `400 MISSING_EMAILS` (no `emails` array), `400 EMPTY_BATCH` (empty array), `400 BATCH_TOO_LARGE` (more than 100 — split and retry).

## Response fields

| Field | Meaning |
|---|---|
| `email` | The address checked (trimmed, otherwise as submitted) |
| `valid` | `true` when the domain exists and publishes MX records |
| `score` | 0–100. Starts at 100; deductions for missing MX (−45), unknown domain (−40), disposable domain (−50), role account (−10), likely typo (−15, score capped at 60) |
| `verdict` | `valid` (score ≥ 80 with MX present), `risky` (50–79, or a likely typo), `invalid` (no MX / unknown domain / bad syntax) |
| `checks.syntax` | Address matches a valid format and length limits |
| `checks.domainExists` | DNS resolves for the domain |
| `checks.mxFound` | The domain publishes MX records |
| `checks.disposable` | Known disposable / temporary-mail provider |
| `checks.roleAccount` | Local part is a role address (admin@, support@, noreply@, …) |
| `checks.freeProvider` | Major free consumer provider (Gmail, Outlook, QQ, …) |
| `mxRecords` | The domain's MX hostnames |
| `suggestion` | Corrected full address when the domain looks like a typo of a popular provider, else `null` |
| `typoLikely` | `true` when `suggestion` is present; a typo'd address is never a clean valid |
| `domainAgeDays` | Days since the domain was registered, from RDAP registry data; `null` when RDAP has no record — never estimated |
| `domainAgeSource` | `"RDAP"` when `domainAgeDays` comes from a registry record, else `null` |
| `catchAllRisk` | `possible` / `low` / `unknown` — a risk signal inferred from MX and provider patterns |

## Real response (verified live)

`GET /validate?email=support@allbirds.com`:

```json
{
  "email": "support@allbirds.com",
  "valid": true,
  "score": 90,
  "verdict": "valid",
  "checks": {
    "syntax": true,
    "domainExists": true,
    "mxFound": true,
    "disposable": false,
    "roleAccount": true,
    "freeProvider": false
  },
  "mxRecords": ["allbirds-com.mail.protection.outlook.com"],
  "suggestion": null,
  "typoLikely": false,
  "domainAgeDays": 9068,
  "domainAgeSource": "RDAP",
  "catchAllRisk": "possible"
}
```

Note how the verdict is read: score 90 and `valid` despite `roleAccount: true` (a role address costs 10 points, it does not fail the address), and `catchAllRisk: "possible"` is a signal to weigh, not a verdict.

## Runnable curl

```bash
: "${RAPIDAPI_KEY:?set RAPIDAPI_KEY to your own RapidAPI key}"

# One address
curl -s "https://emailcheck-email-validation-api.p.rapidapi.com/validate?email=support@allbirds.com" \
  -H "x-rapidapi-key: $RAPIDAPI_KEY" \
  -H "x-rapidapi-host: emailcheck-email-validation-api.p.rapidapi.com"

# A whole list in one call (max 100 addresses)
curl -s -X POST "https://emailcheck-email-validation-api.p.rapidapi.com/validate/batch" \
  -H "x-rapidapi-key: $RAPIDAPI_KEY" \
  -H "x-rapidapi-host: emailcheck-email-validation-api.p.rapidapi.com" \
  -H "Content-Type: application/json" \
  -d '{"emails": ["support@allbirds.com", "user@gamil.com", "test@mailinator.com"]}'
```

## Billing

BASIC $0 — 100 calls/month · PRO $9/month — 10,000 calls · ULTRA $29/month — 50,000 calls. A batch of 100 addresses is one call, so a 10,000-address list costs 100 calls, not 10,000.
