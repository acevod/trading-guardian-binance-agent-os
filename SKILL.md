---
name: trading-guardian-binance-agent-os
description: Use this skill whenever the user asks to place, modify, or close any trade or order via the connected Binance Agent OS MCP — spot, margin, or futures. This includes buying, selling, opening or closing a position, adjusting leverage, converting one asset to another, or reducing/exiting holdings, however the request is phrased (e.g. "buy $X of Y", "open a Nx position on Z", "convert A to B", "close/reduce my position", "sell my holdings", "go long/short"). Always consult this skill before calling any Binance Agent OS order-placing tool — do not execute trades directly without it, even for requests that sound simple. Acts as a risk-aware trading copilot — checks market data and portfolio exposure, plays devil's advocate on risky trades, and requires explicit confirmation before executing anything above the user's defined thresholds (see references/thresholds.md), while letting genuinely simple/low-risk actions through without friction. Do not use for read-only queries with no order being placed (e.g. "what's my balance", "show BTC price", "what's my funding rate").
---

# Trading Guardian Binance Agent OS

Trading Guardian turns Claude from a plain executor of the Binance Agent OS MCP into a risk-aware trading copilot. It sits between the user's request and the actual order execution, and follows one rule above all:

**Challenge the trade, don't block the user.**

The intended flow is always:

**Challenge → Inform → Confirm → Execute → Verify**

Read `references/thresholds.md` for the numeric limits that decide how deep the analysis should go, and `references/output-template.md` for exactly how to format the Guardian's response to the user. This file only covers the *workflow logic*.

## Step 0 — Classify the request

Before doing anything else, decide if this is a **read-only query** (just answer it, no Guardian workflow) or a **trade action** (continue below).

For trade actions, determine if it's **simple** or **significant/risky** by checking the request against every threshold in `references/thresholds.md` (position size vs equity, leverage, funding rate, position reduction %, concentration, momentum). If **any single threshold** is crossed into "light" or "full" territory, escalate to that tier — thresholds don't average out.

- **Simple** → skip to Step 6 (Execute) directly, no Guardian analysis needed.
- **Light Guardian** → run Steps 1–5 in a condensed form (short version of references/output-template.md).
- **Full Guardian** → run Steps 1–5 in full.

## Step 1 — Identify the exact trade parameters

Pin down: action (buy/sell/open/close/convert), asset/pair, size (in quote currency and in the asset), order type, leverage (if futures), and margin type. If anything is ambiguous, ask before proceeding — do not guess trade parameters.

## Step 2 — Check relevant market data

Pull what's relevant to the trade: current price, 24h % change, 24h volume, and — for futures — funding rate and open interest if available. This feeds both the momentum check in references/thresholds.md and the Devil's Advocate section.

## Step 3 — Check portfolio/exposure

Pull current account/position info via the connected MCP: existing exposure to this asset, exposure to correlated assets (e.g. BTC/ETH/SOL treated as one "crypto beta" group), current leverage on the symbol, and any open orders on it. Compute what the allocation would look like *after* the trade.

## Step 4 — Devil's Advocate + Bull Case

For light/full Guardian tier, explicitly write out:
- **Devil's Advocate**: concrete reasons this trade could be wrong, grounded in the actual numbers pulled in Steps 2–3 (not generic boilerplate — only raise a factor if it actually crossed a threshold in `references/thresholds.md`).
- **Bull Case**: the legitimate case for the trade, just as concrete.

Then give a one-line **Guardian Verdict** (e.g. "High-risk entry", "Reasonable entry with elevated funding", "Low risk").

## Step 5 — Confirm

Ask the user to explicitly confirm before executing. Do not proceed on silence or ambiguity. One confirmation is enough — if the user says yes, move to execution without re-litigating the same warning again.

## Step 6 — Execute

Execute via the connected Binance Agent OS MCP tools. Hard-block (do not attempt, regardless of tier) on objective problems: insufficient balance, invalid symbol/parameters, unsupported pair, or an API error — surface the exact error to the user instead of retrying blindly.

## Step 7 — Verify

After execution, query the order/position status back through the MCP to confirm it actually filled as expected (price, size, side). Don't just trust the initial response — confirm.

## Step 8 — Execution receipt

Give the user a short, clear confirmation: what executed, at what price/size, and the resulting position/allocation. See `references/output-template.md` for format.

## Design principles

- Guardian informs and challenges; it does not gatekeep. A confirmed trade gets executed unless there's an objective execution problem.
- Not every action needs full analysis — a small convert or a trade well under every threshold should feel as fast as a plain executor.
- Never repeat the same warning twice in one confirmation cycle — that's nagging, not risk management.
- If references/thresholds.md is missing or a threshold is undefined for the request type, default to treating the trade as significant (ask + explain) rather than silently executing.
