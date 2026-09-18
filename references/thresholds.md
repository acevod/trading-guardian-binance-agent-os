# Trading Guardian — Thresholds

Numeric limits Trading Guardian uses to decide whether a trade is **simple**, **light Guardian**, or **full Guardian**. Edit the numbers here freely — this file doesn't affect the workflow logic in `SKILL.md`.

If any one threshold below crosses into "light" or "full," the whole trade escalates to that tier — tiers don't average across categories.

## 1. Position size vs account equity
| Range | Tier |
|---|---|
| < 5% of equity | Simple |
| 5–15% of equity | Light Guardian |
| > 15% of equity | Full Guardian |

## 2. Leverage (futures)
| Range | Tier |
|---|---|
| ≤ 3x | Simple |
| 4x – 10x | Light Guardian |
| > 10x | Full Guardian (always flag as high-risk regardless of size) |

## 3. Funding rate (futures, per 8h)
| Range | Treatment |
|---|---|
| -0.03% to 0.03% | Normal — don't mention |
| > 0.03% or < -0.03% (up to ±0.1%) | Elevated — raise in Devil's Advocate |
| > 0.1% or < -0.1% | Extreme — explicit warning, expensive to hold |

## 4. Position reduction / close size
| Range | Tier |
|---|---|
| < 30% of holding in that symbol | Simple |
| 30% – 70% | Light Guardian |
| > 70% (or closing to zero) | Full Guardian |

## 5. Concentration

**Single-asset concentration**
- Single asset > 40% of total equity → flag concentration risk (escalate at least to Light Guardian; Full if also size/leverage thresholds are crossed).

**Correlated-group concentration ("crypto beta")**
- Treat the following as one correlated group by default: BTC, ETH, SOL, and any other non-stablecoin asset that is currently ranked in the top 20 by market capitalization (or that the model reasonably judges to have high historical correlation with BTC).
- Stablecoins (USDT, USDC, BUSD, FDUSD, DAI, etc.) are **never** part of this group.
- If the group would exceed 60% of total equity after the trade → flag concentration risk.
- If it is genuinely unclear whether an asset belongs in the group, **default to flagging** rather than skipping the check.

## 6. Momentum (Devil's Advocate input)
- 24h price move > ±7% in the *same direction* as the requested trade → flag as "chasing momentum"

## Defaults / fallback
- Default account base currency: USDT
- Default market: USDⓈ-M Futures (not COIN-M) for futures trades
- Default margin type: Cross
- **Critical data unavailable (equity, market data, or portfolio/position data):** hard-block. Do not execute. Inform the user and ask them to retry. Do not fall back to Light Guardian or any silent path.

## Notes
These are starting-point defaults, not fixed rules — revisit as account size, trading style, or risk tolerance changes.
