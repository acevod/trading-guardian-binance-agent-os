# Trading Guardian — Execution Safety

This reference defines the state and freshness controls that apply between risk assessment, confirmation, and execution.

## 1. Risk snapshot

Treat the following as one risk snapshot:

- account equity / balance;
- available balance when relevant;
- current price and required market metrics;
- funding/open interest for futures when available and relevant;
- current position/exposure;
- correlated-group exposure;
- current leverage;
- relevant open orders;
- post-trade allocation calculation;
- bound trade intent parameters.

A tier decision is valid only against a complete snapshot. Missing fields that are required to compute an applicable rule are a hard stop.

## 2. Freshness

A data point is not considered fresh merely because the MCP returned it successfully. Prefer source timestamps and validate them against current time.

Use these default maximum ages unless the connected MCP provides a stricter source-specific requirement:

| Data | Maximum age |
|---|---:|
| Current price / ticker | 15 seconds |
| 24h market metrics | 60 seconds |
| Futures funding / open interest | 60 seconds |
| Account equity / available balance | 30 seconds |
| Position / exposure / leverage | 30 seconds |
| Relevant open orders | 30 seconds |
| Correlated-group market-cap classification | 24 hours |

If a source does not expose a timestamp and freshness cannot reasonably be established, treat the affected **execution-critical** data as unavailable and hard-block rather than assuming it is current.

These are safety defaults, not exchange guarantees. They should be tightened where the MCP/source supports better freshness information.

## 3. Confirmation freshness

A Light/Full confirmation is not a standing authorization.

- If the user confirms promptly and the final revalidation shows no material state change, the confirmation remains valid.
- If there is a noticeable delay, revalidate the complete risk snapshot, not only equity/price/funding.
- If the final snapshot changes a tier-producing condition, concentration status, execution-critical parameter, or required market state, the old confirmation is invalid.
- If a formerly Simple trade becomes Light/Full, do not execute under the old Simple classification. Present the new Guardian state and obtain confirmation.

## 4. Final gate

Immediately before execution, all of the following must be true:

1. Required risk data is present and fresh.
2. Bound trade parameters are unchanged.
3. For Light/Full, confirmation matches the bound parameters.
4. Recomputed tier is unchanged from the required authorization path.
5. No new hard-stop condition exists (insufficient balance, invalid/unsupported parameters, authorization error, etc.).

If any condition fails, do not call the order execution tool.

## 5. Ambiguous execution results

If an execution request times out, returns an ambiguous response, or the client cannot establish whether the order was accepted:

1. Do not blindly retry.
2. Query the relevant order/position state.
3. Determine whether the original request was accepted, filled, partially filled, pending, or rejected.
4. Only issue another order if it is objectively established that the original request was not accepted and the new order is still authorized by the current workflow.

If the MCP exposes an idempotency key or client order ID, prefer using it for the bound intent — keyed to the specific confirmation cycle (not trade parameters alone), so a retry of the same attempt reuses the key but a separate, later-confirmed trade with identical parameters gets a new one. Do not assume idempotency exists if the MCP does not expose it.
