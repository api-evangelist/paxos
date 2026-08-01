---
name: Place and manage an exchange order
description: Read market data and place, check, and cancel a limit or market order on the Paxos order book.
api: openapi/paxos-v2-openapi-original.json
operations: [ListMarkets, GetOrderBook, CreateOrder, GetOrder, CancelOrder]
---

# Place and manage an exchange order

Use this flow for order-book trading on Paxos (itBit exchange lineage): inspect the book, submit an order, track and cancel it.

## Auth
- OAuth 2.0 client credentials; request scopes `exchange:read_order exchange:write_order`.
- `Authorization: Bearer <access_token>`.

## Steps
1. **ListMarkets** — retrieve the set of currently available markets and their trading details (tick size, precision).
2. **GetOrderBook** — read current bids/asks with resting quantities per price level for the target market.
3. **CreateOrder** — submit a market, limit, or post-only order for a `profile_id`. Supply a client `ref_id` for idempotency.
4. **GetOrder** — read the current state of the order (historical data prior to 2022-05-16 is unavailable).
5. **CancelOrder** — submit a cancellation request. The response acknowledges the request but does not guarantee the order was cancelled — confirm via `GetOrder`.

## Rules
- `CreateOrder` is idempotent on `ref_id`; a repeat returns `409 Order already created`.
- Respect per-market tick size and rounding (Orders, Precision and Rounding guide).
- Insufficient balance returns `403 Insufficient Funds`.
- For low-latency trading, use the FIX 4.2 interface or the WebSocket market/execution feeds instead of REST polling.
