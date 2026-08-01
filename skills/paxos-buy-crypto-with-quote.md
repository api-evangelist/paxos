---
name: Buy crypto with a held-rate quote
description: Fetch a Paxos held-rate quote and execute it to buy or sell an asset at a locked price.
api: openapi/paxos-v2-openapi-original.json
operations: [ListQuotes, CreateQuoteExecution, GetQuoteExecution]
---

# Buy crypto with a held-rate quote (HRQ)

Use this flow for the Crypto Brokerage held-rate-quote product: lock a price, then execute against it.

## Auth
- OAuth 2.0 client credentials; request scopes `exchange:read_quote exchange:write_quote_execution exchange:read_quote_execution`.
- `Authorization: Bearer <access_token>`.

## Steps
1. **ListQuotes** — list the latest available buy/sell quotes for the supported assets. Each quote is a held rate valid for a short window.
2. **CreateQuoteExecution** — execute against a chosen quote on a `profile_id` to buy or sell the asset. Supply a client `ref_id` for idempotency.
3. **GetQuoteExecution** — read the execution to confirm it settled and inspect the filled amount and price.

## Rules
- Quotes expire — a stale quote returns the `403 Expired` problem type; re-list and retry.
- `CreateQuoteExecution` is idempotent on `ref_id`; a repeat returns `409 Already Exists`.
- Ensure the profile has sufficient balance or the call returns `403 Insufficient Funds`.
- Precision/rounding rules apply per market (see the Orders, Precision and Rounding guide).
