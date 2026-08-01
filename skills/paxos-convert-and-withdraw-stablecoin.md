---
name: Convert to a stablecoin and withdraw on-chain
description: Convert on-platform fiat to a Paxos-issued stablecoin 1:1 and withdraw it to an external blockchain address.
api: openapi/paxos-v2-openapi-original.json
operations: [CreateStablecoinConversion, GetStablecoinConversion, CreateDepositAddress, CreateCryptoWithdrawal]
---

# Convert to a stablecoin and withdraw on-chain

Use this flow to turn on-platform USD into a Paxos-issued stablecoin (USDG / PYUSD / USDP / PAXG) and move it to an external wallet.

## Auth
- OAuth 2.0 client credentials; request scopes `conversion:write_conversion_stablecoin conversion:read_conversion_stablecoin transfer:write_deposit_address transfer:write_crypto_withdrawal`.
- `Authorization: Bearer <access_token>`.

## Steps
1. **CreateStablecoinConversion** — convert assets already on-platform 1:1 between fiat and a stablecoin on a `profile_id`. Supply a client `ref_id`.
2. **GetStablecoinConversion** — poll the conversion `id` until it completes (or subscribe to `orchestration.*` / `transfer.*` webhooks).
3. **CreateDepositAddress** *(optional, if receiving)* — create a blockchain deposit address on the target network for a profile.
4. **CreateCryptoWithdrawal** — withdraw the stablecoin to a specified destination address on the chosen network. Travel Rule originator/beneficiary data may be required or the call returns `403 Travel Rule Information Required`.

## Rules
- All creates are idempotent on `ref_id`; repeats return `409 Already Exists`.
- Unsupported network/asset combos return `400 Unsupported Address`.
- Watch `transfer.crypto_withdrawal.pending|completed|failed` webhooks for settlement.
