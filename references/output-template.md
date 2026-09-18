# Trading Guardian — Output Templates

## Full Guardian

```
TRADE GUARDIAN

Requested: [action, e.g. "Buy $500 BTC" / "Open 20x ETH long"]

MARKET CHECK:
- [Symbol] momentum: [24h % move, direction]
- Funding: [normal/elevated/extreme, with number if applicable]
- Volume: [source-provided classification, or omit if unavailable]

PORTFOLIO CHECK:
- Current [asset] allocation: [X]% of equity
- Estimated allocation after trade: ~[Y]%
- [Concentration flag if triggered, e.g. "Crypto-correlated group now ~Z%"]

DEVIL'S ADVOCATE:
- [Concrete reason #1, tied to an actual observed condition]
- [Concrete reason #2]
- [Concrete reason #3 if applicable]

BULL CASE:
- [Concrete reason #1]
- [Concrete reason #2]

GUARDIAN VERDICT: [one-line risk note]

Confirming exact parameters: [re-state size, symbol, side, leverage, order type and other execution-critical parameters]
Do you still want me to execute this exact trade?
```

## Light Guardian

```
Quick check before I execute [action]:
- [One or two concrete factors that triggered Light tier]

Verdict: [short risk note].

Confirming exact parameters: [re-state size, symbol, side, leverage if any, order type]
Go ahead with this exact trade?
```

## Simple execution

No Guardian formatting. Execute only after the complete risk-state evaluation and mandatory final revalidation show that no tier-producing threshold was crossed.

## Execution receipt

```
✅ Executed: [action, size, symbol]
Fill price: [price]
Resulting position/allocation: [brief state after]
```

If execution fails, report the exact safe error instead of the success receipt — don't retry silently.

If the fill is partial or still pending, report the actual filled size and current order status instead of the success receipt.

## Data-unavailable hard stop

```
I can't safely evaluate this trade right now — [equity / market data / portfolio data] is unavailable or not fresh enough from Binance Agent OS.

Please check your Agent OS connection and try again. I will not execute without this data.
```

## State-change / stale confirmation

```
The trade's risk state changed before execution:
- [specific changed condition and old → new value]

I won't execute under the previous risk assessment. [For Light/Full: ask for fresh confirmation using the exact current parameters.]
```

## Style notes
- Keep each section to bullet points, not paragraphs.
- Never pad Devil's Advocate/Bull Case with generic filler.
- Do not claim a volume classification unless the source provides one or a documented baseline supports it.
- Don't repeat the same warning twice in one confirmation cycle.
- Always re-state the exact bound parameters in the confirmation question.
