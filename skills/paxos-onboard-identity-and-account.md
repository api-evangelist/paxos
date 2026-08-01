---
name: Onboard an identity and open an account
description: KYC an end user or institution on Paxos, open an account and profile, and confirm it can transact.
api: openapi/paxos-v2-openapi-original.json
operations: [CreateIdentity, SandboxSetIdentityStatus, CreateAccount, CreateProfile, ListProfileBalances]
---

# Onboard an identity and open an account

Use this flow to bring a new person or institution onto the Paxos platform so they can hold balances and transact.

## Auth
- OAuth 2.0 client credentials. Get a token from `https://oauth.paxos.com/oauth2/token` (Sandbox: `https://oauth.sandbox.paxos.com/oauth2/token`), request the scopes `identity:write_identity identity:write_account funding:write_profile funding:read_profile`.
- Send `Authorization: Bearer <access_token>`.

## Steps
1. **CreateIdentity** — create a `PERSON` or `INSTITUTION` identity with the required KYC details. Supply a client `ref_id` for idempotency; retry with the same `ref_id` to avoid duplicates (a duplicate returns 409).
2. *(Sandbox only)* **SandboxSetIdentityStatus** — set `id_verification_status: APPROVED` and `sanctions_verification_status: APPROVED` (institutions need only the sanctions status) so the test identity can transact. Enum: `PENDING | ERROR | APPROVED | DENIED | DISABLED`.
3. **CreateAccount** — create an account for the identity via `identity_id`; this identity is the primary owner for tax purposes.
4. **CreateProfile** — create a `NORMAL` profile under the account to hold asset balances.
5. **ListProfileBalances** — confirm the profile exists and read its available/trading balances.

## Rules
- Every create is idempotent on `ref_id` — reuse it on retries.
- In Production, identity verification is asynchronous; watch `identity.approved` / `identity.denied` / `identity.documents_required` webhooks rather than forcing status.
- Errors are RFC 9457 `application/problem+json` (see errors/paxos-problem-types.yml).
