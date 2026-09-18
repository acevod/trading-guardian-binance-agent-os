# Trading Guardian — Thresholds

Numeric limits Trading Guardian uses to decide whether a trade is **simple**, **light Guardian**, or **full Guardian**. Edit the numbers here freely — this file defines the policy inputs; `SKILL.md` defines the workflow.

If any one **tier-producing** threshold below crosses into Light or Full, the whole trade escalates to the highest applicable tier. Warning-only signals do not by themselves create a tier, but they must still be checked and surfaced when applicable.

## 1. Position size vs account equity — tier-producing
| Range | Tier |
|---|---|
| < 5% of equity | Simple |
| 5–15% of equity | Light Guardian |
| > 15% of equity | Full Guardian |

Calculate using the intended order notional in the account base currency divided by the current account equity. If the required notional or equity cannot be determined reliably, hard-block rather than guessing.

## 2. Leverage (futures) — tier-producing
| Range | Tier |
|---|---|
| ≤ 3x | Simple |
| > 3x and ≤ 10x | Light Guardian |
| > 10x | Full Guardian (always flag as high-risk regardless of size) |

Use the leverage that will actually be submitted. If leverage is being changed, evaluate the resulting leverage as part of the same bound intent.

## 3. Funding rate (futures, per 8h) — warning-only
| Range | Treatment |
|---|---|
| -0.03% to 0.03% inclusive | Normal — don't mention unless useful |
| > 0.03% to 0.1% inclusive, or < -0.03% to -0.1% inclusive | Elevated — raise in Devil's Advocate |
| > 0.1% or < -0.1% | Extreme — explicit warning; expensive to hold |

Funding is a risk signal but does **not** independently change Simple/Light/Full under the current policy. This avoids pretending that a funding observation has a tier when no tier mapping is defined.

## 4. Position reduction / close size — tier-producing
| Range | Tier |
|---|---|
| < 30% of holding in that symbol | Simple |
| ≥ 30% and ≤ 70% | Light Guardian |
| > 70% (or closing to zero) | Full Guardian |

For a reduction, calculate the requested reduction divided by the current holding in the same symbol. If the current holding cannot be established, hard-block rather than guessing.

## 5. Concentration — tier-producing

### Single-asset concentration
- If the **post-trade** allocation of one asset is > 40% of total equity → at least Light Guardian.
- It becomes Full only if another Full tier-producing threshold is also crossed.

### Correlated-group concentration (deterministic crypto beta group)

Treat the following as one correlated group by default:

- BTC
- ETH
- SOL
- any other **non-stablecoin** asset that is verifiably ranked in the top 20 by market capitalization by the available market-data source at the time of the risk snapshot.

Stablecoins (USDT, USDC, BUSD, FDUSD, DAI, etc.) are never part of this group.

Do **not** add an asset solely because the model believes it has high historical correlation with BTC. If the top-20 classification cannot be verified from available data, mark the classification as **unknown** and default to flagging the concentration check rather than silently treating the asset as uncorrelated.

If the post-trade correlated-group allocation would exceed 60% of total equity → at least Light Guardian.

## 6. Momentum — warning-only

- 24h price move > +7% and requested action is bullish/increasing exposure → flag as **chasing momentum**.
- 24h price move < -7% and requested action is bearish/decreasing exposure → flag as **chasing momentum**.

This is a warning-only signal under the current policy. It does not independently change the tier unless another tier-producing rule is crossed.

## 7. Volume — descriptive signal only

Volume must be reported only when a reliable baseline or classification is available from the market-data source. This policy does **not** define a numeric volume threshold, so do not invent one.

If the source provides a native `normal/elevated` classification, report it as supplied and identify it as source-provided. Otherwise omit the volume classification rather than manufacturing a threshold.

## Defaults / fallback
- Default account base currency: USDT
- Default market: USDⓈ-M Futures (not COIN-M) for futures trades
- Default margin type: Cross
- Critical data unavailable (equity, required market data, or portfolio/position data): hard-block. Do not execute.
- Undefined or ambiguous tier-producing rule: Full Guardian / confirmation required rather than silent execution.
