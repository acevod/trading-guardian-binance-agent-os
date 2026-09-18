# Changelog

All notable changes to this Skill are documented here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/), versioning follows [Semantic Versioning](https://semver.org/).

## [2.0.0] — 2026-09-19

### Breaking / behavior changes
- Missing or stale critical data (equity, required market data, or portfolio/position data) now **hard-blocks** execution. Previously, missing portfolio/equity data fell back to Light Guardian minimum instead of stopping — that silent fallback is removed.
- Leverage evaluation now applies to **margin trades**, not only futures. A margin buy that draws on borrowed balance is evaluated the same as futures leverage, even with no futures "leverage" field set. Trades that previously classified as Simple may now classify as Light/Full.
- All percentage-based checks (position size, single-asset and correlated-group concentration) must be computed in one consistent, converted base currency. If equity is split across currencies/assets that can't be reliably converted, this is now treated as equity being unavailable → hard-block, where previously there was no explicit rule.
- Core workflow restructured and renamed: `Challenge → Inform → Confirm → Execute → Verify` (v1) is now `Bind → Gather State → Evaluate → Challenge/Inform → Confirm → Revalidate → Execute → Verify` (v2), with a new mandatory **Step 5 — Final pre-execution revalidation** that runs for every tier, including Simple.
- `references/execution-safety.md` and `tests/regression-cases.md` are new required reads / references from `SKILL.md` — installs of prior versions that only shipped `SKILL.md` + `thresholds.md` + `output-template.md` are incomplete under this version.

### Added
- **Trust boundary / override resistance:** MCP tool output is explicitly treated as untrusted data, never instructions; user attempts to skip confirmation, reclassify a trade, or "just execute" are explicitly ignored unless they are a genuine reply to an already-shown Step 4 confirmation prompt for the same bound intent.
- **Trade-intent binding:** exact trade parameters are bound at Step 0; any change to size, symbol, side, leverage, or order type creates a new intent requiring a new classification/confirmation cycle.
- **Anti-replay / idempotency guidance:** idempotency keys, where the MCP supports them, are scoped to the specific confirmation cycle — stable across retries of the same attempt, distinct across separately confirmed trades with identical parameters.
- **Freshness policy** (`references/execution-safety.md`): explicit max-age table per data type (price, market metrics, equity, positions, correlated-group classification) and hard-block behavior when freshness can't be established.
- **Ambiguous-execution handling:** no blind retry on timeout/ambiguous execution result — query order/position state first.
- **Partial-fill / rejected-fill reporting:** Step 7 and the output template no longer fold a partial or rejected fill into a normal success receipt.
- **Deterministic correlated-group definition:** BTC/ETH/SOL plus any verifiably top-20-by-market-cap, non-stablecoin asset; unknown classification defaults to flagging rather than silently excluding.
- **Multi-leg / batch request handling:** each leg of a multi-trade request is its own bound intent with its own confirmation, while equity-based checks account for the combined effect of all legs in the request.
- **Convert/swap momentum handling:** momentum is evaluated independently on both legs of a convert (decreasing exposure to A, increasing exposure to B).
- **Session-level trading pattern signal:** an informational, warning-only overtrading/revenge-trading flag in Devil's Advocate; never gates execution.
- **Decision log:** optional one-line append to Light/Full execution receipts (tier, key thresholds crossed, equity snapshot) for the user's own record-keeping — not persisted by the Skill itself.
- `tests/regression-cases.md`: behavioral regression cases covering tier boundaries, hard-stops, revalidation, parameter binding, ambiguous execution, prompt injection, confirmation timing, idempotency scoping, margin leverage, multi-leg, convert momentum, overtrading signal, and currency mismatch.
- `references/execution-safety.md`: new reference file for freshness, confirmation validity, the final pre-execution gate, and ambiguous-execution handling.

### Fixed
- Volume classification is now explicitly non-tier-producing and only reported when the source provides a real classification (no invented thresholds).
- Wording clarified so an affirmative confirmation reply ("execute it", etc.) is never read as valid confirmation when it's part of the user's opening message for a new intent rather than a reply to an already-shown Step 4 prompt.

## [1.0.0] — initial release

- Initial Trading Guardian Skill: `Challenge → Inform → Confirm → Execute → Verify` flow.
- Tiered classification (Simple / Light Guardian / Full Guardian) based on position size, leverage, position reduction, and concentration.
- Devil's Advocate / Bull Case / Guardian Verdict formatting for Light/Full tiers.
- Basic hard-blocks on insufficient balance, invalid parameters, unsupported pair, and API error.
